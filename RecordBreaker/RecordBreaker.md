# _Overview

_RecordBreaker or RacoonStealerV2 is the new version of the stealer Racoon that we can buy as malware-as-a-service at the black markets under $300. Widely used in mass campaigns or used by criminal groups where they try to infect repositories or attach this malware usually compressed in ZIP/RAR format. Its main objective is to reach the largest number of victims and contains different phases including process injection, binary downloading and information theft._

# _Technical Analysis

RecordBreaker usually appears in infected repositories or in attachments as a compressed file that when opened will execute a binary, which depending on the version, will make connections outside in different ways in order to geolocate, check for internet connection or download files, if this phase is fulfilled, it will execute an injection in a legitimate binary, usually related to .NET as RegAsm, InstallUtil or Regsvcs from where it will perform the information theft.

![image](https://user-images.githubusercontent.com/91592110/193622883-b7c05e8b-f640-40dd-aea7-253e3e333fb9.png)

An example of execution flow is the following in which after a zip, a binary is executed that makes a request to a malicious IP and then launches a sleep encoded of 12 seconds to inject RegAsm.exe

![image](https://user-images.githubusercontent.com/91592110/193624249-b3b4d9a4-bf6d-4768-a1e6-729731d2ddf7.png)

In some versions it performs fake PNG/JPG downloads, if it fails to make the connection, the next phase is not performed.

![image](https://user-images.githubusercontent.com/91592110/193624765-df4165ac-a11f-40ed-8341-67557c14fc44.png)

RecordBreaker generates persistence by creating a task with a name similar to the structure of a CLSID

![image](https://user-images.githubusercontent.com/91592110/193625182-99dbf25c-1837-4d64-b3e6-ff633c885e30.png)

Task name example:

```
\{0A2620E2-3469-4C1A-AD19-BD835A6FA571}
```

After this initial phase, it performs an injection, commonly using processes related to .NET, we can see how it will perform a process suspension, and then write in the (WriteProcessMemory) modify the thread context to get all the info of this (SetThreadContext) and when it has done all the operation release the thread in which it is writing.

![image](https://user-images.githubusercontent.com/91592110/193626223-03d7dacf-e53e-457b-b320-38335471a8d3.png)

Processes to be considered for injection:

> - RegSvcs.exe
> - RegAsm.exe
> - AppLaunch.exe
> - InstallUtil.exe
> - aspnet_compiler.exe

Once injected into the legitimate process, RecordBreaker will start working, in the first phase, it will resolve imports, since by itself, it only has the ability to load other APIs/libraries with _GetProcAddress_ + _LoadLibraryW_. When it has all the new libraries and APIs loaded, we will get a better understanding of the code, and the malware, new capabilities.

![image](https://user-images.githubusercontent.com/91592110/193627753-deba8711-f8a2-4754-ba95-9474d86713c7.png)

We can see how it performs a reinfection control using Mutex with a hardcoded string

![image](https://user-images.githubusercontent.com/91592110/193628250-8c7c8255-f58d-431d-b12e-674a5dc34f0c.png)

And we can see how it obtains data that will be used later encrypted in RC4.

![image](https://user-images.githubusercontent.com/91592110/193628432-b8b24203-0a2e-4c4f-a545-6c5b309d0613.png)

After a decryption phase, it collects data from the machine, user, search engines, etc. Which it will collect on the basis of an internal configuration.

In this phase, we can also see, depending on versions, how it performs download requests to malicious IPs to bring more functionalities to the code.

![image](https://user-images.githubusercontent.com/91592110/193629025-5883e72f-9933-4eaf-94f8-bd3b2d3f2aa2.png)

To obtain the data, RecordBreaker will use SQLite to perform queries and obtain all the information it is interested in, and then save it in files that it will send to the C&C.

![image](https://user-images.githubusercontent.com/91592110/193629354-02df0565-2a14-4855-90e2-3bd570b2bf9b.png)

The main information that RecordBreaker usually steals is:

> - Browser (Cookies, User info, Passwords)
> - Telegram info
> - Bank Information (Cards/Accounts)
> - CrytpoWallets

All this information will be sent to a C&C server and since you have performed a persistence, it can continue to increase this information :)

# _IOC

### _SHA256
```
502e941d65f743b781b0214c9c37b8d4cca4b27fa7d62be943a63a9de93812d3
7f2e0645947bc96cd2f5edd2260db48f09102dcfd0fcd85896d287c1b621770d
7dc73e29c582c16283115de0c5d03ecc102b47b82f4b4e957f2630d935967b61
B5137e4be20605c7ad8b5bc1045210c9c42ae4190be76aab1bc72e0a71c703d2
Ba69e1ed08c7288821223595c9b220bc5c53d0485930958a1db415a3f7f56945
767cdc4f8adf3bdfeec2879b0976476dcc0aeeeee5e524d3c2e4ade70c181e9c
207256464e91fde4a35aad23a7a56ef9f9438cd3ef946418f6e06a00e70d7808
D184612e2f0b0b065986b94869296330d56e356e2d1ef077461e064bdcb4d3a2
0050ab6912e9c63bb65930ef52094fca855f8dab9210f74b1d9e2c6c61c9d5b0
dba0be52e9c4d97b164f7c131710f4761b16b2c4ddacafe5cfe10bfbc6148c3c
3bf0e1d823e337ba4372bd55bb57d93789bda09c5bf3d8b82a308781c8f9a09f
3339d515567a632170ca67cf4ee006f24166f9fb47b317b44abada6f5174d1f9
69eac55ff687843f9d856e7facdef36d1c53faa5df6ff3eab874a1426a5efcea
4dd0e5762e6ae6553863f594262e8a1d3f48635d1f49ced7975fc5842fa69a14
```

### _IP

```
89.208.104.46
88.119.170.241
85.192.63.46
206.166.251.254
172.111.36.191
102.130.114.185
91.201.25.172
94.131.107.239
193.106.191.223
```

### _Domains

```
http://rgyui[.]top
http://acacaca[.]org
```

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:

