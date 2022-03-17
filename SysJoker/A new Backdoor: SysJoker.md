# _Overview

_SysJoker is a backdoor which appeare for the first time at the beginning of 2022 whose power resides in being cross-platform. Its main objective is to install itself in our computer and perform espionage and/or data theft tasks. Currently it has not yet been attributed to any group or campaign._

# _What we see at a glance

At this sample written on C++ will be several mentioned functionalities before

SHA256: 
```
1ffd6559d21470c40dcf9236da51e5823d7ad58c93502279871c3fe7718c901c
```

![exeinfo](https://user-images.githubusercontent.com/91592110/158830817-202c7da0-5bbe-463a-b3b8-de53991d2cb9.png)

We can see at a glance how a powershell is executed after the binary, which will have different utilities and will be used in different points

![ps1](https://user-images.githubusercontent.com/91592110/158831357-e02fa45e-021e-4443-8de8-2a4f3ab1f556.png)

![ps2](https://user-images.githubusercontent.com/91592110/158831379-0b64a19d-52c7-4295-9247-653f430fae5a.png)

![ps3](https://user-images.githubusercontent.com/91592110/158831388-60d0d35c-fef3-48ea-8290-5d2fad118fd6.png)

# _Analysis

Seeing how it works from the beginning, SysJoker will perform a folder creation, usually in ProgramData or in subfolders of AppData in which it will leave a copy of itself named after legitimate software, in this case using igfx, in its legitimate variant related to _Intel Graphics Common User Interface_.

_Execution flow:_

```
.
├── (Parentprc) SysJoker.exe
|     ├─ (Childprc) Powershell.exe
|     ├─ (Childcmd) powershell.exe copy '<Sysjoker source path>' '<Sysjoker destination path>'

```
![0](https://user-images.githubusercontent.com/91592110/158833215-43fe28c7-bc4b-4e18-9fed-9821493086e9.png)

After this, we will see how the dropped file is executed in the previously mentioned folder, and it will obtain different important elements of the equipment such as: MAC, OS and Network. It will save it in different .txt files, which will be in the same path where the copy is running. As we can see, it sometimes uses the internal WMIC tool to get certain data and dump it.

_Commandlines:_

```
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" getmac | Out-File -Encoding 'Default' 'C:\ProgramData\SystemData\temps1.txt' ; wmic path win32_physicalmedia get SerialNumber | Out-File -Encoding 'Default' 'C:\ProgramData\SystemData\temps2.txt'

"C:\Windows\System32\Wbem\WMIC.exe" path win32_physicalmedia get SerialNumber

"C:\Windows\System32\cmd.exe" /c wmic OS get Caption, CSDVersion, OSArchitecture, Version / value > "C:\ProgramData\SystemData\tempo1.txt" && type "C:\ProgramData\SystemData\tempo1.txt" > "C:\ProgramData\SystemData\tempo2.txt

"C:\Windows\System32\cmd.exe" /c wmic nicconfig where 'IPEnabled = True' get ipaddress > "C:\ProgramData\SystemData\tempi1.txt" && type "C:\ProgramData\SystemData\tempi1.txt" > "C:\ProgramData\SystemData\tempi2.txt
```

![1](https://user-images.githubusercontent.com/91592110/158833441-25716da1-ce42-48df-a8a8-7f06c6e994e8.png)


After the previous step, the files will be deleted since the information will be hardcoded before being sent to a C&C server, this information will be dumped to a supposed dll, which, we can see that it is simply coded information.

![2](https://user-images.githubusercontent.com/91592110/158833531-ed22f635-7a93-4ba1-9406-68c69f103f6a.png)

![exeinfo2](https://user-images.githubusercontent.com/91592110/158833555-b4aac916-5406-4746-ae15-d68f32096295.png)

After this, it will persist on the computer by adding the previously copied file to the registry key _CurrentVersion\Run_ with which it will get execution again in a forced way (/F) every time we start the system.

_Commandline:_

```
REG ADD HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run /V igfxCUIService /t REG_SZ /D "C:\ProgramData\SystemData\igfxCUIService.exe" /F
```

![3](https://user-images.githubusercontent.com/91592110/158833676-78210d1a-bf1c-483d-91e5-9d3b9988bed7.png)

As for communications against C&C, it has been seen to use different domains _drive.google.com_ or _github_ to make it more difficult to detect traffic and perform rule creation. In short, the Sysjoker does not bring anything relatively new, but it has a quite characteristic methodology that makes it quite recognizable. 

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:

