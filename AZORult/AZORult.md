# _Overview

_AZORult is one of the best known malware within the Stealer family. It is usually sold on Russian forums for prices ranging up to $100. This malware has been used by a large number of important threat actors, including some dedicated to crime such as FIN11 or TA505 (GracefulSpider) or others that are part of a state-sponsored model such as GorgonGroup from Pakistan._

_This malware usually starts from an initial point as documents via Spear-Phishing or compromised web pages and is characterized by performing different file drops that later will execute and check the connection to the C&C, after this, it will steal information and create persistence or a backdoor to, before performing the exfiltration of the data, have opportunities to persist in the system and thus, the actor continue obtaining information from the system to which the affected computer belongs and sometimes pivot._

# _Technical Anlysis

As mentioned above, this malware is most commonly found after Spear-Phishing or a Web site that has been compromised. After this, its main objective will be to obtain data from elements such as search engines, FTP or emails.

An example of what its steps would be is as follows:

![image](https://user-images.githubusercontent.com/91592110/224556672-90418d79-8cd2-4e0a-b6d8-12f6a0d6fb19.png)

Once we have a general understanding of what this malware is and what are its steps, we visualize how several samples would be executed to have the widest possible context, since, being a malware that several groups use and with the possibility of being able to buy it, we find different versions of AZORult coexisting

After reviewing dozens of samples the most common is to see executions of this one doing the first phase launching several cmd.exe to support itself in the execution while dropping other files in temporary paths or using sleeps through PS to avoid the sandbox analysis timeout or delay the execution.

![image](https://user-images.githubusercontent.com/91592110/224557218-f12f529b-0504-4c20-8c6e-de773316e5ab.png)

During this phase, different files are launched from behind in folders such as:

```
C:\Users\<Username>\AppData\*\Temp|Roaming
C:\ProgramData 
C:\Users\<Username>\
 ```
 
Here we will see different files among which, depending on the version, we can find scripts and other binaries that support the execution or the following file, which will be AZORult.

Before the execution of the Stealer, depending on the version, it performs different actions such as:

> Create tasks to create persistence:

```
schtasks /create /tn /tr "<FilePath>" /sc minute /mo 1 /F
```

>Modify Office settings, where we can see how the Resilience or the MRU that would make changes which will indicate that you will not be able to recover Office files that you have open and that your Office history will disappear. In this case, performed on Word, which indicates that you are covering your back, as one of the samples came from Spear-Phishing

![image](https://user-images.githubusercontent.com/91592110/224557923-144f20d2-a3ec-4a3f-ad33-2fc3a2e3794c.png)


> Killing other processes:

```
Taskkill /F /IM winword.exe
```

Or run the AZORult, which is your final objective as your initial "Dropper" part. It is worth noting, that the AZORult seen, mostly were either .NET obfuscated with SmartAssembly or AutoIT (The most common) or NullSoft, I also found some in C++

![image](https://user-images.githubusercontent.com/91592110/224557733-9b213c37-b7bb-4df3-84e8-caddd9a03d07.png)

Once the stealer is deployed, it will perform some actions in one way or another, since, as I said, several versions usually coexist at the same time. We can see how they usually have obfuscation and/or anti-analysis techniques

![image](https://user-images.githubusercontent.com/91592110/224558128-eaed7574-a10d-48f1-b7d5-ca517d792d47.png)

We can see from anti-dbg where it is observed if there is any thread with the DBG, or locating the HEAP flags, in short, I have seen different ways to avoid that we analyze it at low level.

Subsequently, we can see how he tries to avoid reinfection with Mutex, but not all samples used the mutex.

![image](https://user-images.githubusercontent.com/91592110/224558271-4effb092-e566-4158-846c-3056f964f70f.png)

During the rest of the execution, I notice how it actually tries to control at several points which processes are running on the system, usually linked to anti-analysis as well since it allows us to see if there are any applications that we do not want to be running

![image](https://user-images.githubusercontent.com/91592110/224558344-9f31784c-3927-45b2-ad39-a147198542f2.png)

To later look for permissions that it has in execution via Token to then be able to execute elements in a different thread with the context from which it has obtained all the information related to the credentials of the main process.

![image](https://user-images.githubusercontent.com/91592110/224558593-d72e2364-d89e-4f0f-bd8b-3ca1b3e8043c.png)

In other words, we can check if the process(Thread) in execution has enough privileges to take the thread context and execute whatever we want in the thread with the same privileges, or with the privileges of another user :)

I also find the ability to control the device by remotely shutting it down or suspending it using the Suspend + Force flag quite interesting

![image](https://user-images.githubusercontent.com/91592110/224559188-d6dc4fb4-e33c-4e7e-bd87-65ebc095e2d0.png)

![image](https://user-images.githubusercontent.com/91592110/224559201-4f586858-a1fe-46c5-9264-b02467f898f5.png)

In the meantime, we forget the most important parts here, which are, the information theft, where according to different samples we can see that it obtains information from elements such as:

* Mail informaton
* Wallets
* FTP
* Browsers information (Cookies, History...)
* SSH (Putty|WinSCP)

![image](https://user-images.githubusercontent.com/91592110/224559341-381f2024-c36a-4977-a4b7-66970b6c5db8.png)

Once you have obtained everything you wanted you make requests to the C&C with all the data you have obtained. It is worth noting that most samples I have found of AZORult, before running most of its functions had a check where checks if it reached the C2, if this did not happen, automatically stopped the execution, this is quite common because it avoids that if some analysts focus on the Sandbox and the C2 falls, we can not analyze the content of what comes next, besides generating an extra layer of protection, as sometimes analysts analyze malware without internet traffic.

![image](https://user-images.githubusercontent.com/91592110/224560382-e12c1e15-a931-4261-91ed-790d33a5fa18.png)

I have found myself analyzing quite a few samples that did not have C2 and I have had to bypass the checks or directly understand with the dissaasembly that I was doing with the context of the rest of the samples, since as you know, C&Cs come and go and usually fall relatively quickly due to the great work done by the community and the companies reporting them.

Finally, it is usually observed in VT how the samples I am analyzing are related to each other to check if it leads me to IP/Domains that are highly reported, to find more samples and therefore, different versions, to see if there is any collection where I can get more context from the intelligence part, and so on

![image](https://user-images.githubusercontent.com/91592110/224560624-6703aef0-ad96-47a7-b803-2f7965d50380.png)

# _Summary of behaviour

Chain:

```
Dropper > Infection > C2 communication > Information theft > Persistence and backdoor creation > Encryption > Data exfiltration
```

Office Manipulation:

```
(PrntPrc) Winword|Excel | TempFile > (ChildPrc) cmd.exe | powershell.exe > (cmd contains) \Resiliency /f
(PrntPrc) Winword|Excel | TempFile > (ChildPrc) cmd.exe | powershell.exe > (cmd contains) \File MRU /v
```

Persistence:

```
(Cmd) schtasks /create /tn /tr "<FilePath>" /sc minute /mo 1 /F
```

Timeout | Ping abuse to auto-delete:

```
(Cmd) cmd.exe /c C:\Windows\system32\timeout.exe 3 & del "<FilePath>"
(Cmd) cmd.exe /c ping 127.0.0.1 && del "C:\Users\admin\AppData\Local\Temp\<FolderPath>\<FileName>.exe" >> NUL
```

Suspicious file reading sensitive information:
```
(Path Temp|Roaming|ProgramData)Prc > ReadFile > (Path contains) \Wallet\ | \Wallets\ | \Recentservers.xml | \accounts.xml
(Path Temp|Roaming|ProgramData)Prc > QueryReg > (Reg contains) \monero | \Bitcoin | \BitCore | \LiteCoin | \WinSCP | \Url History 
```

# _IOC
```
8424aa8b6fda143bd0e2e82ea906b2aee8cf49e416308cd92bd76bdcd46b866f
38c78ebf970f2fc711eddcfa9ab6562c8ccbcfb053e5ececaa695650cf7d8727
97710410be07f6ab12c607e9378bb399bdbe3012da245805212e2b1995065c17
Fd8deb7f3c15bd91961790834864db01b5459a019777266c919465b0cac3751f
9af44ae397fce9e4da5effb82fcecaeadc7dcb412d030c5e0e135639b3686efb
37d4d7a7b84e4f6ead2e950ba252c23fa360a3176f49184942da3046fa693452
C7930d104f9f1e522835dcbd6aecd707b6bdc27ec4f34149d32b90978e4a6878

Bllsl2[.]shop
Bllsl2[.]shop/bll/index.php
Nghfh[.]com
Nghfh[.]com/em/index.php
171.22.30[.]164/standright/index.php
85.31.45[.]29/ongod/index.php
64.52.171[.]230/index.php
209.208.65[.]177/index.php
185.225.73[.]49/office/index.php
Domcomp[.]info
Domcomp[.]info/1210776429.php
arthurcambell.ac[.]ug
arthurcambell.ac[.]ug/azne.exe
Nanaa[.]tech/index.php
movescx[.]top
cointra[.]ac[.]ug
safetygear[.]pk
scientific[.]pk
karimgousa[.]ug
mistitis[.]ug
goldrush[.]ug
beachwood[.]ug
citypharmacylv[.]com
ddlakava[.]ac[.]ug
cracksmsa[.]ug
lastimaners[.]ug
marksidfgs[.]ug
kenmil.ac[.]ug

```

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:
