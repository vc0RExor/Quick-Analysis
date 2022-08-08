# _Overview

_Tofsee is a malware used for mass campaigns, which does not have an associated group or actor. It has gone through different phases, but has generally been used to create Botnets or SpamBots, as well as mining actions. The starting point of Tofsee is usually a Loader or an email using the SpearPhishing technique that will launch the malware. It has been in all its stages a really worked malware, with code obfuscations, packaged samples, anti-analysis techniques, which led to a backdoor to perform minings taking advantage of the botnet features or to perform Spam._

# _Technical Anlysis

After the loader or phishing we would see Tofsee running on the computer, an overview of how the current versions of Tofsee work can be seen in the following schematic diagram

![image](https://user-images.githubusercontent.com/91592110/183301004-460ecac2-de36-4c90-bbb2-3d665c0ef7fb.png)

The actual way of working of the current versions usually has different variations, but in essence it works similar, after the execution of Tofsee, a copy of itself is created in SysWow64 (or equivalent), which then moves to temporary folders, usually the sample of the temporary folder and the one of SysWow, will not have the same name. After this, it creates a service using sc.exe, usually with a name and/or description related to elements of the network. Later, it makes modifications in the FireWall to add an svchost to a completely permissive rule, this svchost is the one that would have injected in the process. After this execution, you will be able to perform your tasks within an svchost which, observing processes would be difficult to discern also having permission to send and receive all types of traffic through the Firewall and with a service that allows us to launch the backdoor as if it were a system service.

```
.
├── (Parentprc) Tofsee.exe
|     ├─ (Childprc) (Moved | Dropped) <RandomName>.exe
|             ├─ (Childprc) Netsh/cmd/sc
|             ├─ (Childprc) (Injected) Svchost 
|                     ├─ (Net) C&C

```

A large number of samples have been reviewed to narrow down the current versions of tofsee as much as possible, an example of what we would see when viewing one of them in PEstudio would be as follows

![image](https://user-images.githubusercontent.com/91592110/183301107-e6b9b874-27fc-411b-ad82-5e0c9733b75d.png)

At the first step, we see how it launches a copy of itself to SysWow64 which it then moves to a temporary folder, the commands used, launched by cmd.exe are the following:

```
"C:\Windows\System32\cmd.exe" /C mkdir C:\Windows\SysWOW64\<File Dropped (itself)>
"C:\Windows\System32\cmd.exe" /C move /Y "C:\Users\user\AppData\Local\Temp\<random name>" C:\Windows\SysWOW64\<File Dropped (itself)>
```
![image](https://user-images.githubusercontent.com/91592110/183301167-0903a506-59a7-4041-ba14-eb01d0254566.png)

After having two files in different locations, it uses one of them (SysWow location) to make the modifications in the defenses, as well as the persistence using a service. For the creation of the service, we can see that it creates with own start a service "Wifi Support", as we had commented before, usually it is habitual that it is related to something of the network, trying to avoid to be found.

The command used, launched by sc.exe is as follows:

```
"C:\Windows\System32\sc.exe" create <Name of file dropped> binPath= "C:\Windows\SysWOW64\<Path file moved>\<Random name> /d\"C:\Users\<username>\Desktop\<Random Name>"" type= own start= auto DisplayName= "wifi support"
```
![image](https://user-images.githubusercontent.com/91592110/183301204-2e5da69d-f222-4250-bc7d-05f83ef8ffb0.png)

Once it has created the service,  It has ensured that the backdoor will remain on the computer launched as another service and going completely unnoticed, so need to modify FireWall rules to prevent its communications to the outside from having any problems. For this, it launches through netsh.exe the creation of a rule that allows all the traffic for a svchost process (the one that is injected). 

The command used is:

```
"C:\Windows\System32\netsh.exe" advfirewall firewall add rule name="Host-process for services of Windows" dir=in action=allow program="C:\Windows\SysWOW64\svchost.exe" enable=yes
```

![image](https://user-images.githubusercontent.com/91592110/183301250-cc434e70-68f1-43dc-8a5e-ceda1fffb134.png)

![image](https://user-images.githubusercontent.com/91592110/183301252-ea93585f-2a51-470d-a9ce-0333cdbfb5e5.png)

In addition, we can see, that it enters it in exclusions in registry key, being the path with random name the place where it was previously self-dropped

![image](https://user-images.githubusercontent.com/91592110/183301277-258e0f5c-1482-40a7-b250-570810ddf4e4.png)

![image](https://user-images.githubusercontent.com/91592110/183301279-22ef33e1-124b-4206-b19f-7cd6ef08f88b.png)

While all the above processes are being launched, we have the other binary in a temporary path performing other actions, such as injecting an svchost, the same one we have seen that has been introduced in exclusions and a rule has been created in the FireWall. During a normal execution, we would find an svchost without a parent process, which, after looking at it in depth, we would see that it is another of the binaries launched by Tofsee by locating it by PID

![image](https://user-images.githubusercontent.com/91592110/183301407-cf23b299-1b0c-4ff6-9cf9-5d2320efb97b.png)

This binary is the other binary that Tofsee worked with and had previously moved to %temp%.

![image](https://user-images.githubusercontent.com/91592110/183301423-29bbdd7f-d5ba-4712-8265-ca6ce4abf1ec.png)

Once in this phase, you have the Tofsee functionalities inside a legitimate process, with persistence created and with fully open traffic on the FireWall that you will use to connect to a C&C server.

The most common destinations in the campaigns used at recent months are the following (Russian or Chinese IPs/domains are commonly used by Tofsee):

```
svartalfheim.top
lazystax.ru
```

![image](https://user-images.githubusercontent.com/91592110/183301459-911f0845-42e9-4bfa-9db7-ff8feb352951.png)


# _IOC

### _C&C

```
svartalfheim.top
lazystax.ru
```

### _SHA256

```
9ff3eb5bac86aef0116488ac380f9d7ea15d27f9d580462fcf3612293525f50f
2f5b289a8dcb26ed9389a49687e513f162ed3145469a5cb90f0aab45c699c3d9
22179b5cece54e42dbc249c5112994e0e760c2435f3547579d04d19882b79b03
3c38e00f572800dfdcf676a141e4b98903977368f8870cd29221b3320b640ed4
E64afadba25eededfb3259f10671cf5551e53341e13702489a7c334fcf6514b0
A96edd53cb70eb51f8bb9fbd0b9d0777e6b65c5203fb3b73229431b49da155e4
F6bf44f37a819ce566e217cf94a3de32a404cf303700f82788b44f9fde8e0937
Ddaada491b8cf4d1187cb01078c5f3fd167e76c324d3e0db83753a6922e739f4
1dc4c40d2a971bcfba32e21ab5ff5c127aa1cea66a72176b753c8c9d0d54fc25
820b43708e064c1a6eb1eeb411b011e900fcd162afdd55d077ef619777a9d12b
D5a45f5fbe4d0679d208908a1282e6675456cf565b427d886cab0b2fdf92c21b
5a3ac08cf1bdee0dfe30bcd306c5613a7526eda1a1eaec00d76f3681b25f8694
5b3ed204bf794afc4d32c750e74a219c4730b2a96ded36c6ea2753581158ab11
5b5cae86c3a28fc013bce2e327c424168e212220b8b284714bbccf9926e7cb6e
5bb633fef2f50ca5ec2302ea37800f68c76596ef770c394e706f57a5d655feac
5bcc5563c17c7805651b50c509b6862ef554948336fbec1f99f9059a65d6b703
5c3b9c929cac14277c3f810b57a487a6093470e3b4fd56ac65ab2d3f6b4e9959
460a0c6a2f32fd82774d45670a9110f046b3fbf4093d17ca378b0f4f63de6e0e
93ead5f86a3eda573a375afebaa3651c2f48c4381e657a0819176a79843749cb
40ca993b529a245ebf6ad5deb1b11beffc24a8ddd5c908e341ae886a2df3351c
C526be053d1279fd214ac204c23ce79ee034af6d6213fff69f31a19a53f5bba2
46359601942519b156cf35e91a252abb4381c695ec053216efc948729d2eb2ab
6644c412c44c8686437b1fa3ad6d5698c1071ac133ccd060b1062df37b081f1e
9c2a732f02510a2524d1ffb52ea6c96a93c3ee6bb3ad777181596f370c030da3
C5a593ba8d3006cbd55a0b41436e055eeed50a122b2a0f8d28fa30624565dc48
B10c82428c7284ce3ab78edeaf6582fcbc93e3a647559fba49bdc1589df13ab9
A5aaf507390c8ab2bd12849e68a740b19c97e5bfdfd3459ca0f120490fee3fd0
e248be07f11c33ff0af5bdd36d2bca1ae9c392223bd5b14c600b15637e02c5f7
a06f640a6317ffeaed88cf7a08c8680a4bc4abe69286bce68f03c19ba319e103
```

### _IP

```
31.41.244.126
31.41.244.127
31.41.244.128
43.231.4.7
46.173.223.212
98.136.96.76
111.121.193.242
```

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:
