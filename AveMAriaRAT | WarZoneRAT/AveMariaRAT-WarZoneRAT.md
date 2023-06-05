# _Overview

_AveMariaRAT, also known as WarZoneRAT, is one of the most famous and widely used RATs in recent years. It can be purchased with a license and monthly subscriptions ranging from $16 to $38 on its website. This tool is used and modified by various groups, ranging from disorganized or resource-limited individuals known as script kiddies to highly relevant criminal groups or APTs._

_Some of the notable groups that have been observed using AveMariaRAT include:_

* Tomiris ( 🏴 )
* Carbanak | Anunak ( 🇺🇦 )
* Aggah ( 🏴 )
* BlindEagle | APT-C-36 ( 🇨🇴 )
* Confucious ( 🇮🇳 )
* SideWinder ( 🇮🇳 )
* HazyTiger | Bitter ( 🇮🇳 )
* FIN7 ( 🏴 )
* SandWorm Team | Voodo Bear ( 🇷🇺 )
* Kasablanka ( 🏴 )

_This malware, used by the mentioned groups, can infiltrate the infrastructure in various ways, from exploiting Spear-Phishing to compromising websites where it is downloaded. Once on our devices, the RAT has capabilities to escalate privileges, bypass UAC, evade defenses like security software, gather sensitive information from the device and user, and inject itself into processes to maintain active communication with the C&C server operated by the attacker_

# _Technical Anlysis

As I mentioned earlier, the threat actor commonly utilizes compromised emails or web pages with the ultimate goal of getting you to download or execute the malware sample after passing stage number 1.

An example step-by-step process of various samples I have analyzed from different versions of AveMariaRAT could be as follows:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/7b0fd7ea-31b3-4e3f-918f-6a22cd64e22a)

The executions vary greatly depending on the version, but typically it tries to determine if it is already running on the machine using a Mutex

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/0839fa96-a46f-485f-b1a1-44cac58f1697)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/4a9ea3be-b4f3-47b9-ae45-1e2967427a1e)

After that, it can perform anti-dbg/Anti-VM tasks or directly check for security software that may be installed on the system

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/619ef405-4c83-4a9f-a9fe-785684b27f5e)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/bd1a6e15-ee06-48f3-8dd7-d0d62b513864)

A common practice in AveMaria RAT is privilege escalation or bypassing UAC. The most common method involves abusing sdclt.exe, which, similar to CompMgmtLauncher (Windows console), searches for a library or component in a registry that is normally not present. This is exploited for hijacking by introducing a library that will be loaded, and these binaries usually have the capability to elevate themselves, allowing them to execute with higher privileges, thus running the malware with the desired privileges

Here you can see this parallelism with a PoC I made some time ago and we can see that it seeks the same in both cases:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/b72185d5-ad06-4245-a594-b0e234cedd40)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/5796da24-e061-4dba-ac08-d771a9c7dfa0)

The overall picture of how this is done with sdclt is as follows:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/5cbc41c7-9720-4e12-bfb4-e27476a6fb7b)

Depending on the sample, there is usually a subsequent resource loading for the injection. This can be found in the resource section of the binary, and as seen, it checks if you are already an administrator, so the previous step should have been executed

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/311152d1-cecb-45cc-b9d7-3102106e3eb5)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/5ab64b8a-c286-4e63-94b1-3d2d19d21703)

A common practice I have observed in WarZone RAT is modifying network elements, as well as modifying keys like _\CurrentVersion\Internet Settings_, where it increases the maximum number of simultaneous connections to a server. The objective for this is evident

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/e37b4a51-e562-4b72-a543-c2e9550fee7d)

Additionally, it is also quite common to see the modification of Zone.Identifier, which allows the attacked machine to receive malicious files from untrusted sources. In other contexts, the connection would be rejected or warned as an unadvisable request. However, after the modification, the malware can send and execute any type of file, bypassing any previous warning or restriction.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/62fe4010-fdb8-4d70-8ba8-1dbad1a58c4a)

Furthermore, it is also common for the malware to make changes in security checks of certain browsers, such as Internet Explorer (IE). By changing the registry key, it can increase access to certain addresses to complement its previous behavior

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/3268f312-3a1b-4883-9ef8-4e1cde5e9c51)

After this, it is common for the malware to establish persistence. Depending on the sample, it may create a copy of itself and then add the path to \CurrentVersion\Run (or similar) registry key, or it may leave a copy of itself or a script in a startup folder that will be executed on system startup

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/e331bb0a-b437-476b-b59a-4d51b7dc2df5)

In some samples, I frequently observe the modification of folder attributes where the malware launches its copies or files it intends to use or execute. It is common to launch them in a folder like Documents or Downloads and modify the attributes to point to another location, such as ApplicationData or ProgramData. For example, it would be seen as Documents:ApplicationData. This action allows the malware to evade restrictions or security policies of the initial folder where the files are launched and potentially remain undetected. The telemetry data always reflects this pattern

```
<Original folder where the samples are launched>:<Folder with modified attributes>
```
![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/a9aeac68-5c88-40af-9cff-7cc3d9ae40b0)

During the analysis of different samples, I notice the consistent use of various anti-debugging techniques along with frequent sleep commands that significantly slow down debugging. This forces me to bypass them entirely

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/9d37ae62-f70a-4340-986a-0ef007d5bce4)
![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/ac9ae255-665f-4e65-9472-3ae1bfb3a72f)
![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/d899c227-b042-4e23-bece-440be9eafe76)

Once the sample achieves persistence, the files are launched in the desired folder, and the appropriate privileges are obtained, it commonly proceeds with the injection process. Injection serves multiple purposes: to remain hidden, complicate analysis, and ensure better persistence by residing within the target process. If the process being injected into has elevated privileges, the injected code will also inherit those privileges

Here we can see different injections performed by this RAT based on different compilations. Commonly, it extracts the code from a specific section and constructs it in memory for subsequent injection. Alternatively, it may assemble or deobfuscate the code from a resource. The target process varies depending on the samples, but I have observed processes related to .NET (such as AppLaunch, csc, RegAsm...) or the RAT itself (either a sample launched with a different name in another folder or self-executing and injecting)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/2ec84d3b-b39b-4aa0-a0a3-1d8fe0e8d858)

After this, the malware payload will be within another process and can act more freely. At this point, the most common action is to gather relevant information about the device and/or the user, such as the operating system type, machine name, language used, etc.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/69179cec-8458-4aeb-bc6e-dfe68fe6b39d)

Once the previous steps have been completed, the attacker focuses on establishing an external connection. Typically, the malware will have different addresses to attempt to gain access. It will commonly try to establish a connection by opening a socket to the chosen address. In the samples I have analyzed, I have observed that the malware often enters a loop where it attempts to connect to different domains from its list at regular intervals. The malware will remain waiting for instructions from the command and control (C&C) server

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/000c8f31-7b81-44ad-b145-0da187865539)

Once the connection is established, the attacker has achieved the desired objective: a malware injected into a process, whether legitimate or not, from which they can operate. They have modified network parameters, controlled security software, obtained data about the target system, and established persistence for each system login

# _TTP

```
[TA0003][T1060] - Persistence using startup folder 
[TA0003][T1060] - Persistence Using CurrentVersion\Run Registry 
[TA0005][T1089] - AV software check 
[TA0009][T1056] - Steal sensitive information 
[TA0007][T1057] - Check language 
[TA0005][T1497] - Anti-analysis techniques 
[TA0005][T1055] - Process Injection 
[TA0004][T1548] - UAC bypass abusing of sdclt 
[TA0005][T1055] - Load binary from resources 
[TA0005][T1112] - Increase the max. connections to a server abusing of Internet Settings
```

# _IOC

```
135e5dcc50f0857af71511756ec63b393f070fd188506da08012d0201360f890
07dd531c1198ecf78a9d85e26db1f642de2c06d7234f46f97941afbd28bb742f
0f6d6875d6ca1793369166534b041daec3f946d83df7c788ad913999ffd81eeb
389e52a5612f9242ae4162ba51323010a97641be291893dcb9bc261ca26acb27
ef5b814562290c60063075b290966060a79e0cc9e81cd6448d49af5c5879175f
90ebeaa9a68ea0c1bf9aff1f7902d545fd5623af7aba90d8cbc53ece47f43f51
96237eb7f3c5304d26fb06feafab631b64a274eb1037f51b58af586040154572
6c1cc9a94713e2b614dcd99baa05c62f7cab2bb8abdb030b85a1dc539eb21dbc
db382f9a496a46db8d4eaa52ff67355a1d54ccf8531ba6d5ef06c5b445d5d436
73fc341bbc5be844d20c51d8ee5356b9d44a628d0aef4e95df08b057ea6cadba
3bebe61724ca5dc55a37b7c851aa645dbf7c64615e523f7dd2b901ff27d7fae0
3cb44a87566ee76eabd616840ed7d0f5ca8c7ec4d0f40f17642935e60af7074e
719342deb77a6f03f09db95f137db0de14beb7f98c80372c21238a74d5f58b97

5[.]206[.]225[.]104
146[.]70[.]94[.]3
95[.]214[.]27[.]90
161[.]129[.]33[.]242
173[.]212[.]207[.]73
104[.]223[.]19[.]96

warzonlicen1304[.]ddns[.]net
dreams2reality[.]duckdns[.]org
gbotowaya[.]duckdns[.]org
helpme20[.]duckdns[.]org
newnex[.]3utilities[.]com
zpec[.]ru
osairus[.]duckdns[.]org
lacasadelpan2024[.]duckdns[.]org
```

Thank you very much for reading, happy hunting :)

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:

