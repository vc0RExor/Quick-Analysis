# _Overview

_RagnarLocker is a Ransomware normally associated with the APT Viking Spider whose InitialAccess is varied, but as usual, they perform direct attacks trying to exploit systems or after the abuse of legitimate applications or by implanting malware inside these, after these movements, the most common is to gain maximum access and control within the attacked company to encrypt as many computers as possible._

# _Technical Analysis

At the first steps, we find a common function in the Ransomwares that tries to determine which country runs the Malware, this is because certain groups use targets depending on the country and avoid certain countries, as we can see it locates which country we belong to using _GetLocaleInfoW_ and compares it to an internal list of countries as an exclusion, in the case that our country was in the list and the function returned any of the list, it would end the execution.

![image](https://user-images.githubusercontent.com/91592110/139923142-19aa7ac8-2b13-4c2e-8a89-103490001972.png)

List of country languages:

```
Belorussian
Azerbaijani
Ukrainian
Moldavian
Georgian
Armenian
Turkmen
Russian
Kyrgyz
Kazakh
Uzbek
Tajik
```

Later, it obtains the computer name and user data, as well as the MachineGUID of the computer using the Microsoft RegKey Crypthography or the ProductName using the RegKey _Windows NT\Current Version_, something that Ransomwares usually do to obtain information from the computer that they can then use to identify the machines.

![image](https://user-images.githubusercontent.com/91592110/139928520-4eced539-28fe-463e-ab68-0a50fc2722d1.png)

![image](https://user-images.githubusercontent.com/91592110/139928541-f62a525d-4347-44e9-a4f4-7cb21694671c.png)

![image](https://user-images.githubusercontent.com/91592110/139928549-15463579-6611-49eb-be85-bd27251436d5.png)
![image](https://user-images.githubusercontent.com/91592110/139928561-80278c35-7098-46a9-a70e-ff63b7f67340.png)

After identifying disks to be encrypted, it is dedicated to enumerate services, in which, we can see that it uses the _EnumServiceStatusA_, in which it will ask the DB of the control manager (previously opened with _OpenSCManagerA_) and will compare each one of the services with the internal list, in case it finds something related to its exclusion list it will close it using _CloseServiceHandle_.

![image](https://user-images.githubusercontent.com/91592110/139928691-6c4ee9ef-2ef1-4984-8d96-2fc38ec4d895.png)

List of exclusion services:

```
vss
sql
memtas
mepocs
sophos
veeam
backup
pulseway
logme
logmein
connectwise
splashtop
kaseya
```

After that, it will create the usual ransom note in which it will ask for money (usually cryptocurrencies) to ransom the encrypted files, as we can see, it will first create the file by obtaining from memory the name of the txt and will perform the first creation in _\Public\Documents_, directory obtained through the _CSIDL_ identifier using _SHGetSpecialFolderPathW_, after creating it it will rescue from memory both the data it contains in a predefined way and the hash it will create as our identifier that will be used to contact the attackers, in this case using _qTox_.

![image](https://user-images.githubusercontent.com/91592110/139929143-3fbf3f58-6d6f-4c50-9792-0b8fe929cf16.png)

![image](https://user-images.githubusercontent.com/91592110/139929262-fc805c87-0d4f-43be-ab0f-151eeb950f7c.png)

![image](https://user-images.githubusercontent.com/91592110/139929384-5bc991ae-fe43-487c-917d-8ad61c0acd39.png)

Later, it will try to encrypt the files avoiding some folders, files and extensions that it will also check in memory, avoiding touching what it does not need to encrypt or that could alert of its presence, it will make a loop to check the files in each case.

![image](https://user-images.githubusercontent.com/91592110/139929648-c8bb803a-7bbc-483c-b65d-2410815bf0dd.png)
![image](https://user-images.githubusercontent.com/91592110/139929663-fc42bb04-a67a-43f6-8e67-05de7e463315.png)

![image](https://user-images.githubusercontent.com/91592110/139929675-b29517f8-a1df-4096-83a8-f6ad45208a99.png)

An example of what a short file would look like before and after being encrypted, in which we can see how it will introduce the keys and the RAGNAR tag at the end.

![image](https://user-images.githubusercontent.com/91592110/139929936-57087b2c-f250-438c-8039-22412f371d2d.png)

![image](https://user-images.githubusercontent.com/91592110/139929941-080fad57-ef6c-4195-a230-2d3a28a182df.png)

Finally, to execute the txt in the default session in which the RagnarLocker has worked, it performs an _Interactive window station_ in which we will see how it gets the session identifier, the process that is running the Ransomware, duplicate your token, get the session, and so on, to spawn the file in the session and that, at the end of its operation, we know what are the steps and what has happened in each of the affected computers.

![image](https://user-images.githubusercontent.com/91592110/139930089-c224d2a2-3e72-487b-bf7f-5a2d4bddfea7.png)

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:



