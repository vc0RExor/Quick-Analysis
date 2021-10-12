# _Overview

_This malware, it's normally into a office file (Such docx or xls), the user downloads it across to a malicious mail that try to trick us to launch the download. After that, when we try to open this file, It try to execute the content, when it does, the macro will launch and you will see the malicious behavior._

# _How malware doc accesses

As I've explained before, our Malware, comes through mail, the victim, recieves an attached file in a fake mail, when the user accepts the content, the office file (In our case is a .XLS) is downloaded, at this point, the malware writter have a malicious file at your disk, the next step is execution, to do it, the user needs to open the file in a MS Office (Or similar program).
The victim, open the malicious file and will see a ***Fake Document Signed to protect the file*** that "needs" allow the content. By default, this option is locked because it's common to use macros to launch malware or use it as downloader.

![Alt text](https://user-images.githubusercontent.com/91592110/136939248-f1fcec91-b056-441e-9a14-75c29a752c93.png)

![image14](https://user-images.githubusercontent.com/91592110/136941194-6cc7bf21-41ed-415c-a462-3f32b0c357f1.png)

Now, when enable the content, the macro will run in your computer, and the malicious behavior starts.

# _Macro Anlysis & Extraction 

After macro launches, You will see the Process _Excel.exe_ executing a _Rundll32.exe_ , it's a good indicator, to see that a docu have a macro (As launching a shell).

![image2](https://user-images.githubusercontent.com/91592110/136939682-cb1d0548-ed65-4481-868d-1ec073eacb0e.png)

Now will we analyze the macro contents to find interesting information, scanning the XLS document will find PE elements that indicates that into we have malicious content.

![image3](https://user-images.githubusercontent.com/91592110/136939736-1a2368e5-3547-4cf5-ab71-5faba80677f7.png)

After a first overview, we persists using more tools and found an _"AutoExec"_ called as _WorkBook_ into XLS, a register that calls a DLL, and a VBA obfuscated, we can see the path _%APPDATA%\Roaming_ that is a path used normally by malware as _%TEMP%_ where later gains persistance using a regkey or for dropping more malware, we can see a string called _"Goka"_ it will be interesting in a few moments

![image4](https://user-images.githubusercontent.com/91592110/136939787-37a9f328-2094-4eb2-9d25-10f8aceab5b2.png)

![image5](https://user-images.githubusercontent.com/91592110/136939794-583e85fd-166f-4846-b97b-2791399dad97.png)

How we have a _Workbook_ into the XLS, we can research into and extract all of them, and we will see contents as imports, URLs...

![image6](https://user-images.githubusercontent.com/91592110/136939924-1db8f681-de14-4bc6-863c-8c0ab109f672.png)

![image7](https://user-images.githubusercontent.com/91592110/136939940-cdc9c6ae-2191-466b-abb3-91c19cbd2508.png)

Now we have good IOC, a domain _(hXXp://asengjewelry[.]com/)_ and indicators that the Macro will try to download a file (DownloadToFileA) and use it to gain persistence (DllRegisterServer)

Looking for this domain, We find that it's detected by any engines in VirtusTotal, and the domain is from Thailandia, who contains several hosts, common in Malware, his goal is to persist in the time, using the same attack but switching the host or to use some at the same time, our host is thz09

![image8](https://user-images.githubusercontent.com/91592110/136940007-b267690b-2226-4789-be8f-fd8369dd0882.png)

![image9](https://user-images.githubusercontent.com/91592110/136940017-d2e08762-9751-499c-ad4a-17f13baf705e.png)

# _Behavior

At this point, we now that this file is Malware, but we can research a little bit in his behavior, by now, we know this:

> - Excel will run a Rundll32.exe after macro execution
> - Will persist using regkeys
> - Uses PE files as DLL
> - Tries to connects and download a file into a malicious URL

Launching the XLS again, accepting the macro execution, we see again the _Rundll32.exe_

![image10](https://user-images.githubusercontent.com/91592110/136940484-04cd0395-48c7-4c9b-9e39-725456ab49c9.png)

Later, Rundll32.exe, will find the DLL downloaded by the domain _asengjewelry.com_, trying to download a fake .jpg, but this is the dll

![image11](https://user-images.githubusercontent.com/91592110/136940525-32217181-7d8a-49dc-85d7-672de0588d97.png)

![image12](https://user-images.githubusercontent.com/91592110/136940549-5b6c260a-97ad-4ade-bb4f-74393e555ff8.png)

Once the dll is downloaded, and we already have it a in the path that we saw at the previous point, drops it in Roaming, the DLL is the Goka.zzxxcc, and will persist using the COM creating a regkey, normaly, that campaign uses Schtasks to do that, usually, the malwares, uses _\CurrentVersion\RUN_, services or Tasks to gain persistence on our computers

![image13](https://user-images.githubusercontent.com/91592110/136940662-86d9cb95-2577-42af-b2db-5dfd430372c8.png)

This Malware is usual at this world, it's normall see Malware writters using tricks as change the file extension, phising, fake Mail or public domains to infect machines, this files are really useful to gain persitence, and later... Open backdoors, make botnets and so on



> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:
