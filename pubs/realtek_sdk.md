# Software Development KITchen sink

*This article was adapted from, and originally published at http://community.hpe.com/t5/Security-Research/Software-Development-KITchen-sink/ba-p/6745115 (no longer active)*

**CVE-2014-8361 (ZDI-15-155), recently disclosed by the Zero Day Initiative, provides a depressing example of why near-intractable vulnerabilities will continue to plague the tech industry - and will only get worse as we fully embrace the Internet of Things.**

One characteristic of the so-called Internet of Things (IoT) is short development and deployment cycles.  The typical IoT device competes in a market where short time-to-market is as important as the features the device provides.  A key enabler of these rapid processes is utilization of commercial off-the-shelf (COTS) components.  In order to take full advantage of COTS parts, IoT vendors are dependent on the Software Development Kits (SDKs) provided by the parts' suppliers.  The SDKs reduce the complexity and duration of the IoT vendors' own software development effort.  In some cases, the SDK may be minimal, consisting mainly of a few application programming interfaces (APIs).  When hardware is involved, the SDK usually also includes a driver for the hardware.

Commonly, however, these SDKs consist of myriad pieces including debuggers, dedicated integrated development environments (IDEs), miscellaneous tools, documentation, and sample code.  One could say they sometimes include everything but the kitchen sink.  In some cases the SDK ships with production code that the IoT vendor may utilize directly and, knowingly or unknowingly, ship with the product without modification.  The product may even be dependent on this code and unable to function properly without it.  Therefore, a vulnerability in an SDK component can have [far-reaching implications](https://speakerdeck.com/duosec/the-internet-of-things-weve-got-to-chat), especially if the component is widely used and the vulnerability is [publicly reachable](https://www.ftc.gov/news-events/press-releases/2014/02/ftc-approves-final-order-settling-charges-against-trendnet-inc).

Such is the case with Realtek's rtl81xx chipset SDK miniigd vulnerability, which we'll detail in this post.  The vulnerability, CVE-2014-8361, was disclosed without an available patch via HP's Zero Day Initiative as [ZDI-15-155](http://www.zerodayinitiative.com/advisories/ZDI-15-155/) due to a lack of vendor response.  (As this article was in final draft, a software engineer from Realtek made contact requesting details on the vulnerability).  The Realtek rtl81xx series chipsets are highly integrated "system-on-a-chip"solutions used in numerous SOHO (small office / home office) wireless access devices including the TRENDnet TEW-731BR wireless router sold in the United States, on which this vulnerability was discovered by TippingPoint security researcher Ricky Lawshae.  Our investigation indicates the problem does not affect the WAN (public) side of the router when running in its default configuration, though NAT traversal is effective for exploitation.

It's difficult to get a full and authoritative accounting of the exact chipsets used in each of these devices, especially in foreign markets.  In fact, it took quite of bit research just to conclude the vulnerability was in the Realtek chipset versus the core device.  However, according to the [Wireless Chipset Adapter Directory](http://linux-wless.passys.nl/), at least 70 device models use Realtek rtl818x chipsets. These include popular devices from Belkin, D-Link, Netgear, TRENDware (aka TRENDnet), and at least one device from Linksys.  The TRENDnet TEW-731BR and TEW-652BRP V3.xR appear to share the same hardware, specifically the RTL8192CE wireless controller, and therefore the 652BRP is likely vulnerable as well.

It's important to note that this vulnerability is not necessarily limited to the Realtek rtl818x chipset family.  However, without access to Realtek SDKs or a wide array of devices, it's difficult to identify all possibly vulnerable devices.  Clearly a large market share translates into a large vulnerabilty share, especially since nearly all product vendors use a very small set of hardware vendors.  A safe approach is to assume that if a device has UPnP functionality and [uses a Realtek chipset](http://linux-wless.passys.nl/query_chipset.php?chipset=Realtek), it is likely vulnerable.

#### SDK

A software development kit (SDK) is typically a set of software development tools enabling and facilitating application creation and debugging for a certain software framework, hardware platform, computer system, or any similar development platform.

Based on the device's nature and the [open-source implementation](http://rtl819x.sourceforge.net/) (clone) of Realtek's rtl819x SDK on SourceForge, it's likely that Realtek's SDK includes the following:

* A build tool chain for cross-compilation of the operating system
* Source code for the operating system and any open source applications
* Various binary applications fundamental to the device's functionality
* Configuration info for each supported chipset
* Documentation etc.

A vendor will typically complement the SDK with some "value-added" applications and customize the appearance of the web user interface.  The vendor could also tailor the behavior of the system (e.g., add startup commands, or add or remove user accounts) and replace or remove some of the applications distributed with the SDK.  However, since most SDK-shipped applications are hardware-specific and generally provide core functionality, it's unlikely many vendors remove or alter these applications.  This is likely the case with the miniigd daemon in the TEW-731BR.

#### MiniIGD

This article only authoritatively attests to the information discernible from publicly available sources and examination of the [Trendnet TEW-731BR router](http://www.trendnet.com/products/proddetail.asp?prod=230_TEW-731BR&cat=165) shipped with firmware v1.02b05 (released in June 2014) and miniigd version 1.07.  However, research indicates the vulnerability discovered in this device ships with the Realtek SDK, and therefore was not written or licensed (outside any Realtek licensing) by Trendnet.

IGD stands for Internet Gateway Device, which can be used to describe the device, the functionality, or the protocol that implements the functionality.  The full name of the protocol is [Internet Gateway Device Standardized Device Control Protocol](http://datatracker.ietf.org/doc/rfc6887/), which is a Network Address Translation Port Mapping Protocol (NAT-PMP).  Essentially, this protocol and this functionality permit NAT traversal via automatic configuration of port forwarding.  Many routers and firewalls declare as IGDs, allowing any local Universal Plug and Play  (UPnP) control point to establish a port mapping.  As the reader may be aware, UPnP has been [rife with vulnerabilities](http://www.rapid7.com/db/modules/exploit/linux/http/dlink_upnp_exec_noauth) recently, but research of publicly available information did not reveal this specific vulnerability to have been previously known.

Since UPnP and port mapping have been discussed multiple times by multiple sources, they will not be explained in depth here.  As a simple example, a gaming console can use IGD functionality to automatically configure the NAT to always forward a certain port from the NAT's public interface to the IP address of the console, facilitating online gaming.  IGD is implemented with UPnP, which leverages HTTP, SOAP (Simple Object Access Protocol), and XML over IP to provide device and service descriptions, actions, data transfers, and event notifications.

There are alternatives to the miniigd daemon. However, the exact landscape is difficult to understand, since most implementations are privately licensed commercial endeavors and many implementations appear to have common roots.  It is likely that several implementations forked from the same or similar code bases, but have since diverged and been updated separately for vulnerabilities common to the original codebase.  For example, LinuxIGD updates in 2006 eliminated `system()` calls, among other security improvements, that still plague miniigd.  HD Moore's "Security Flaws in Universal Plug and Play" research and whitepaper drove changes to Portable SDK for UPnP Devices (aka libupnp and forked from Intel's Linux SDK for UPnP Devices) and the open source MiniUPnP project.  Prior to his research, the libupnp codebase had not been updated since 2006.  Confusingly, MiniUPnP is a software project that includes the miniupnpd daemon, which implements the IGD functionality, which is implemented with UPnP.

#### The Vulnerability

The TRENDnet TEW-731BR router with firmware v1.02b05, released in June 2014, utilizes the Realtek RTL8196 chipset and thus was likely designed using the Realtek SDK. The router's IGD functionality is provided by the miniigd daemon, which is a SOAP service.  During testing, this service was always listening on TCP port 52869; however, it is possible for the port to change under UPnP rules.

The daemon suffers from command injection vulnerabilities when handling NewInternalClient requests.  Multiple XML values can be abused because they are not sanitized. Moreover they are, inexplicably, written to a temporary file using shell redirection instead of simply writing a file using the C language's standard file I/O functions.  At a minimum, the code should utilize `execve()`instead of `system()`; using `system()` in this case is a well-known bad practice and is specifically highlighted by [CERT's C coding standard](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=2130132).  Writing this file using C file I/O functions would arrest this particular form of command injection, but the fields should still be sanitized in case the temporary file is used dangerously (e.g. it is "sourced" or parsed dangerously) at a later point.  This vulnerability is not novel and has been discovered previously in some of the other IGD implementations, but the hardware SDK developers do not seem to learn from others' mistakes.  The crux of the vulnerability is detailed in the MIPS assembly below:

```
LOAD:0040997C:

addiu   $v0, (aTmpUpnp_info - 0x410000) # (1) "/tmp/upnp_info"
addiu   $a1, (aEchoSSSSNaSS - 0x410000) # (2) "echo \"%s,%s,%s,%s,NA,%s\" >> %s"
move    $a3, $s7                        # (3) Move function arg into a3
addiu   $a0, $sp, 0x178+var_F0          # (4) Move stack var var_F0 into a0 (like lea)
sw      $s6, 0x178+var_168($sp)         # (5) Write key-value pair values to stack.
sw      $s1, 0x178+var_164($sp)         #     these values are attacker controlled.
sw      $s0, 0x178+var_160($sp)         #     Refer to prior instructions (not shown).
jalr    $t9 ; sprintf                   # (6) sprintf(a0_dest,a1_fmt,a2,a3,stack+)
sw      $v0, 0x178+var_15C($sp)         # (7) Store result on stack
lw      $gp, 0x178+var_158_maxsize4($sp)# (8) Restore global pointer (convention)
nop
la      $t9, system                     # (9) Load addr of imported "system" function
nop
jalr    $t9 ; system                    # (10) exec "echo \"$evil\" >> /tmp/upnp_info"
```
At location 1, a pointer to the string `"/tmp/upnp_info"` is stored in register v0.  At location 2, a pointer to the string `"echo \"%s,%s,%s,%s,NA,%s\" >> %s"` is stored in register a1.  The value of register s7, which is copied to register a3 at location 3, was populated by the return value of `GetValueFromNameValueList` (not shown).  `GetValueFromNameValueList` retrieves the value from a key-value pair list (aka a hash, map, or dictionary structure).  The attacker, when submitting the `AddPortMapping` SOAP request, controls these values.

Next, the address of `var_F0` (a 200-byte stack buffer) is loaded into register a0, which will be the destination for the `sprintf` output.  Incidentally, this is a possible stack buffer overflow vulnerability in the making if the buffer's input is not limited elsewhere to 200 bytes minus the length of the static strings going into the buffer.  The three instructions at location 5 are moving additional values onto the stack for use by sprintf.  Location 6 is the MIPS-equivalent to a `call` in Intel syntax and calls `sprintf`.  The calling [convention](http://inst.eecs.berkeley.edu/~cs61c/sp05/hw/proj2/) for `sprintf` has arg0 as the destination for the write, arg1 as the format specifier, and as many additional arguments as needed to fulfill the format specifier.  Any arguments beyond arg2 and arg3 are passed on the stack (location 5).  Location 7 stores the return value of the `sprintf` call. The application is essentially running the following pseudocode:

```
var_15C = sprintf(var_F0, "echo \"%s,%s,%s,%s,NA,%s\" >> %s", attacker_controlled_args)
```
Register v0 will also contain the return value of `sprintf` (the number of bytes written) and a0 still points to the string buffer which now contains attacker-controlled data akin to `echo "attacker_data" >> /tmp/upnp_info"`.  The global pointer register is restored at location 8, which is part of a MIPS convention that [dictates](http://www.cs.umd.edu/class/sum2003/cmsc311/Notes/Mips/altReg.html) a function callee must preserve gp.  Location 9 loads the address of the imported `system()` function and location 10 calls that address.

Therefore, if an attacker can put anything into "attacker_data" that terminates the echo command, arbitrary commands can be run.  In this case, not surprisingly, those commands are run as root.  There are some speed bumps on the way to exploitation since the device is stripped of most file transfer utilities and the added port mapping has some restrictions that limit, but don't eliminate, code execution vectors.  Although only miniigd v1.07 was tested, every copy of the binary gathered from public sources had this version and the same vulnerability.

Some readers may be thinking they've previously seen this vulnerability, and they would be somewhat correct.  Determining whether this is a new vulnerability or not turned out to be quite a challenge and required significant research.  Ultimately, the responsible binary is miniigd and advertises itself as "OS 1.0 UPnP/1.0 Realtek/V1.3" on UDP:1900; however, its service description offers a "Server: miniupnpd/1.0 UPnP/1.0" banner on TCP:52869.  Additionally, miniigd appears to share a lot of code with LinuxIGD.  The command injection vulnerability was long ago fixed in both the MiniUPnP and LinuxIGD projects.  MiniUPnP actually removed all calls to system.  ![UPnP][upnp]

To further muddy the waters, the maintainer of the MiniUPnP project, Thomas Bernard, was consulted and his brief assessment was:  "I guess it [miniigd] is affected by the same vulnerabilities as the old version of miniupnpd is forked from." Doing a quick strings comparison gave him the impression that miniigd left "SSDP/description generation/SOAP/HTTP code from miniupnpd quite unchanged, but added table initialization into the binary (calling iptable through `system()`)."   Calls to iptable were long ago removed from both the MiniUPnP and LinuxIGD projects.

The most likely scenario is as follows.  miniigd was likely forked from LinuxIGD, or at a minimum, it was closely modeled after LinuxIGD.  Source code is available for LinuxIGD, but not miniigd; however, through static analysis much of the miniigd code appears essentially identical to LinuxIGD prior to LinuxIGD security improvements.

#### The Impact

It's difficult to predict which devices are affected without testing each device since vendors may, or may not, use or alter a provided SDK.  Additionally, it's unclear which SDK is provided for each chip.  The assumption must be that if the device has a Realtek wireless chipset, it should be considered vulnerable until proven otherwise.  By examining Realtek's product descriptions and consulting the compilation instructions for the rlxlinux OS provided by the open source rtl819x project (SDK v3.2.3), the rtl8196c, rtl8196e, rtl8196eu, rtl8198, rtl819xD and their descendants are almost certainly affected.

The impact to an affected device has not changed since the 1990's, when this class of vulnerability should probably have been eradicated.  However, the current explosion of devices following this type of development path drastically exacerbates the industry-wide impact.

Though the warning bell has been sounded previously, apparently not every developer has heard it.  There's absolutely no reason to echo strings to a file in a C/C++ program.  Simply writing to a file on the system, as is very commonly done, should be done using standard file I/O.  Not to mention there are safer ways to invoke the system commands, such as with execve().
According to the website Shodan (from which the heat map below is drawn), [almost 850,000 devices](https://www.shodan.io/search?query=realtek+port%3A%221900%22) that expose this functionality to the public Internet are possibly vulnerable.  ![shodan_vuln_heatmap][shodan]

Shodan data is not free of false positives, however, so this should be viewed as an upper bound. Data from the Sonar project (dated 2015-04-27) indicates there are nearly 158,000 devices that are almost certainly vulnerable.

```
zcat data.csv.gz |grep 5265616c74656b2f56|cut -d"," -f2 |uniq |wc -l
```
Therefore, the number of actually vulnerable devices that can be publicly addressed is likely between 158,000 and 850,000.  There is no good reason to expose this functionality on a public interface, but it happens frequently -- sometimes on purpose and sometimes accidentally.  According to [UPnPhacks](http://www.upnp-hacks.org/stacks.html), some Realtek boards have a "problem with firewalling." On RTL86xx based boards, at least, UPnP listens on the WAN interface in addition to the internal network.  This is supported by the fact that multiple devices using various UPnP/IGD implementations have the same problem.  The vulnerability may not be present if the product vendor removed or replaced the IGD functionality. However, there is no broad-stroke way to confirm the actions taken.  Obviously the vulnerability is significantly mitigated if miniigd is listening only on the internal network, since it is not directly exposed.  However, there have been so many problems with UPnP implementations, attackers will still have a field day.  Users and administrators should not consider themselves safe just because miniigd is not listening on the public interface.

Ultimately, any device with a Realtek chip is likely vulnerable at least from the internal network.  The vulnerability is also likely to be long-lived since the vendor has in our experience been slow to respond to security reports, SDKs are traditionally slow to receive security updates, these SOHO devices don't typically have automatic update capabilities, and users infrequently apply manual updates.  The end products often do not have long market lifespans, and thus hardware suppliers are reluctant to fund ongoing engineering efforts.  However, a single SDK often supports whole families of hardware, if not the entire product line, so vulnerabilities will simply port from one product generation to the next.  Even if deployed devices are unlikely to be patched, at least the next generation of a device should not contain the exact same vulnerabilities, especially vulnerabilities that have been popular for decades.

#### Conclusion

Although the vulnerability discussed here is not novel, nor is it technically complex, it appears to have a far-reaching impact.  The impact is also likely to long outlive the devices themselves since Realtek is seems slow to respond to security reports, the SDKs do not seem to receive security updates, and most device vendors use only a handful of chipset vendors.

SDK suppliers are just as responsible as product vendors and should be held accountable by device users as well as by product vendors.  This is especially true since the same SDK situation exists in the mobile device industry.  In the mobile industry, baseband hardware vendors benefit from the fact that product vendors, such as the mobile carrier, also have a vested interest in keeping researchers (and users) away from low-level details in order to keep devices locked to carrier.

However, it's clear that SDKs are mostly ignored when it comes to security updates and their developers do not appear to be taking any proactive, or in some cases even reactive, steps to help ensure the security of the ever-growing "Internet of Things."  At a minimum SDKs should be actively maintained if the SDK is still being provided to product vendors.  Ideally, this article will open the door for discussion of SDK security updates and how the process can be improved for IoT as well as mobile devices.  For some SDK providers, it appears the kitchen sink may not be the only thing not included.

*Many thanks to TippingPoint security researcher Ricky Lawshae who discovered this vulnerability and volunteered his time and his TEW-731BR to this effort.*

[shodan]: https://raw.githubusercontent.com/kernelsmith/about/master/pubs/media/Shodan.png
[upnp]: https://raw.githubusercontent.com/kernelsmith/about/master/pubs/media/PNP.png
