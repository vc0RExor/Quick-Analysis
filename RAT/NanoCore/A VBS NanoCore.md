# _Overview

_NanoCore is a RAT (Remote Admin Tool) used by cybercriminal groups such as APT33 (Refined Kitten) whose InitialAccess is varied, although it has been most commonly used through fake emails with .zipx extensions or with fake formats, which is commonly called phishing (T1566) or in this case, since it contains a file in the email and its objective is execution on disk to go further, it would be more accurate to call it Spearphishing (T1566.001)._

The main potential of NanoCore is usually minning or steal data from the computer and user once it has gained access to the disk, but once it is inside, it could perform any action from the outside, and, of course it depends of version.

To this analysis, I have divided into two parts, which is usually a common practice in which we observe first statically everything we can get in the shortest possible time and then a dynamic in which we will see how it behaves, although, we will lose information if we do not monitor properly or not debug.

# _Static Analysis: Obfuscated JavaScript

We start from a VisualBasicScript (VBS) which is quite obfuscated although we can distinguish some interesting words like "_http_" or "_OwerSheL_" and some "_replace_" that will help us in the deobfuscation, for now we can't see much so we have to get on work.

![image](https://user-images.githubusercontent.com/91592110/139471648-e3465f5e-b7ac-47a7-bbb2-4dedd01dfb4e.png)

After working the script in the deobfuscate and get the most in the shortest possible time, we can see in a small scheme how this first stage works. On it, we find an IP [ _52[.]231.98[.]236_ ] to which it will make a download request using "_DownloadString_", as expected, using powershell and after this, it will execute the content that downloads

![image](https://user-images.githubusercontent.com/91592110/139471880-9c73a086-1d7c-4188-ab2a-a330e2712a18.png)

At a controlled environment, we visit the website against which the request is made, we may find another similar script (the one executed previously) which, as usual in this type of loaders, hides another obfuscated script creating script chains so that the analysts lose interest or increase the difficulty of the analysis and that campaigns last as long as possible.

![image](https://user-images.githubusercontent.com/91592110/139471974-26258efa-b8bc-46d3-8bf0-6f9fde4efe27.png)

Later, we will find another obfuscated script that will try to do the same as the previous one, complicate the work and make us know the minimum of the attack.

![image](https://user-images.githubusercontent.com/91592110/139472315-1da288ab-bdc6-4f9f-b0a3-69b6f969ae3d.png)

After the previous script, we move on to the next one, in which we can see how it makes another request to the same IP but this time using another file (_Server.txt_) that at the time of analysis was already supposed to be another obfuscated file, which we will see now. After this obfuscation, we can see that the attacker manages to generate persistence in RUN by introducing a New.vbs file in it, which is none other than the same file as before that will execute again every time the system is launched, what would be Persistence (TA0003) whose more concrete technique would be T1547.

We see that it also collects in variables the RegKey _\Explorer\User Shell Folders_ to generate persistence, simply, you can enter a registry key in the previous key pointing to _\Public\Run\New.vbs_ and you will get the desired execution at each session startup.

Persistence Using Regkey pointing to vbs:
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
```
![image](https://user-images.githubusercontent.com/91592110/139473111-03257fef-3043-4948-8eca-85cc15a29f0c.png)

For a better understanding, a scheme of the deofuscation would be the previous one, in which we observe in cascade, in a summarized way, how the deofuscation has been done, after this, the connection to the IP releases another file that as we can see, alerts us of a PE signature with the typical _0x4D5A_

As we have seen in the previous image, we take out the binary and, as expected, we get the NanoCore client, as usual, with an obfuscator on top.

![image](https://user-images.githubusercontent.com/91592110/139473921-6e9e5944-f900-4158-bf8a-38a4f5710eb9.png)

# _Dynamic Analysis: Monitoring execution

After the static section, we can know how it is going to behave and we have clear where to put the focus, so we observe how the powershell is going to be executed, in which, we see how it makes the request to the first web and if we did not have the file physically, we would have to extract it manually from this request. 

An important aspect is the use of aspnet_compiler.exe, since it is usual in this type of RAT to inject the code (or perform process hollowing) of the final binary in a legitimate software, we will see that it stays started after the execution of the loader, the reason is simple, the RAT has already started and is doing its task which, at first glance you can not see anything abnormal. We can also see injected MSBuild.exe.

![image](https://user-images.githubusercontent.com/91592110/139474281-fc36362d-8231-437a-9472-72653033ed13.png)

After running the powershell, we can see the same request 2 times to obtain the succession of obfuscated scripts, in which at the end, we obtain the RAT

![image](https://user-images.githubusercontent.com/91592110/139474330-46e85380-c300-4e40-a1f1-393bd8d9645b.png)

![image](https://user-images.githubusercontent.com/91592110/139474364-c60ddb80-fa11-48af-8fac-f436a2a17239.png)

After this, we see how it has introduced the file in _\public\run_ so it has generated the persistence in the registry key that we had mentioned in the previous section.

![image](https://user-images.githubusercontent.com/91592110/139474555-9a495ae9-6740-40a9-929e-16b649f667fd.png)

Then, we will see, as usual in this Malware, it collects certain information obfuscating it and introducing it in some path of \AppData\ or \Public\ with extensions .dat, usual directories of use in this type of actions to avoid as far as possible that it is not found.

![image](https://user-images.githubusercontent.com/91592110/139474569-668aac42-5e51-412f-895d-4bb6d38e3cab.png)


> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:



