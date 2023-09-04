# _Overview

LaplasClipper (Laplas Clipper also known as Laplace Clipper) is a well-known malware that operates as a crypto clipboard hijacker. It has been in use since 2022. This malware can be purchased from its portal for as low as $49, with payment structured as a monthly subscription. LaplasClipper has been employed by various criminal threat actors to steal cryptocurrencies.

This malware gains access to devices through various methods. It has been observed being distributed through YouTube video links or compromised websites, as well as links to files containing LaplasClipper loaders. It has also been delivered via spear-phishing campaigns. In recent versions of the malware, it establishes persistence in registries and injects itself into files it creates to gain an advantageous position. From there, it monitors the clipboard, waiting for cryptocurrency wallet-related information to be added. It modifies this information to hijack the cryptocurrencies to the attacker's server.

As I said, Laplas has been involved in several executions related to other malware or loaders, some of them, which are related to active groups, are the following:

* VidarStealer
* SmokeLoader
* AresLoader
* RedLine Stealer
* AgentTesla

# _Technical Analysis

As mentioned earlier, LaplasClipper has various initial access vectors through which it can infiltrate affected devices. Its ultimate goal is to modify the clipboard to alter transactions related to cryptocurrencies.

An illustrative example of its steps is as follows:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/8b5d9b26-7636-4fed-944b-51c120b495a0)

With this diagram, I aim to provide a general understanding of how new versions of LaplasClipper function. I've reviewed recent samples and found no significant variations between them. Therefore, the execution tree of current LaplasClipper versions should be similar to what I present below:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/cb87f88f-a676-4f2d-9a23-151d062ab9bc)

We can observe the execution of a binary that establishes persistence in \CurrentVersion\RUN. It also launches a file named "ntlhost," which is subsequently injected. Notably, the name of the dropped binary ("ntlhost") and the writing path vary, representing the most noticeable differences I've identified, an example of this are the following paths:

```
C:\Users\<user>\AppData\Roaming\NTSystem\<MalBinaryDropped>.exe
C:\Users\<user>\AppData\Roaming\telemetry\<MalBinaryDropped>.exe
```

Regarding the persistence aspect, there isn't much mystery. The binary creates a file (which we will see shortly) in a temporary path. Depending on the version, it could be one or another path. It then modifies the registry key and adds the newly created path. This ensures that LaplasClipper executes with every login

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/8d9d4da4-9d12-4c7f-8fdd-577a5ea0475f)

At this stage, we've discussed the creation of a file. It's intriguing that relatively recent samples I've encountered are quite heavy. This is a common tactic used by some malware to deter analysis. Such samples slow down orchestrated analysis systems, increase the time taken for software reversing, and so on. Essentially, these samples contain unnecessary functionalities that don't justify their weight. This becomes more problematic when the malware creates a second file in temporary paths.

In terms of file writing, in the analyzed samples, multithreading is heavily employed. Threads are used to expedite the binary's writing process. This approach makes sense, given that the files launched in temporary paths are typically large. As I mentioned earlier, in this routine, we can observe how the binary gradually writes the file before releasing it, leaving the file in the path.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/04fe9b1f-fa46-4203-aacb-51fe8cfa36d3)
![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/8f747c20-8e5a-49f9-8373-eab53c2e5d25)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/076e7728-589c-41a4-bf0f-bbd164426620)

Following this, I started comparing the files. It seemed unusual and uncommon for malware to drop itself, so I anticipated that it would weigh much more. However, upon examining its functions, strings, and data content, I found that its functionalities were mostly the same, with the addition of a significant amount of data at the end.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/7d26723c-c8c5-493f-935d-bece1f6a084f)

The binary operates heavily in memory. Depending on the version, I encountered samples packed with MPRESS or ones with obfuscated sections that were gradually deobfuscated during runtime. Consequently, in addition to the functionalities Laplas already possesses (excluding packed versions), it imports numerous libraries. During runtime, it dynamically loads more libraries and imports using GetProcAddress + LoadLibrary.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/491e9417-a8d5-4772-95b7-b8ecad72f348)

Some of the loaded imports/functions include:

```
AbortSystemShutdownA
AbortSystemShutdownW
AccessCheckAndAuditAlarmA
AccessCheckAndAuditAlarmW
AccessCheckByTypeA
AccessCheckByTypeAndAuditAlarmA
AccessCheckByTypeAndAuditAlarmW
AccessCheckByTypeResultListA
AccessCheckByTypeResultListAndAuditAlarmA
AccessCheckByTypeResultListAndAuditAlarmW
AddAccessAllowedAce
AddAccessAllowedAceEx
AddAccessAllowedObjectAce
AddAccessDeniedAce
AddAccessDeniedAceEx
AddAccessDeniedObjectAce
AddAce
AddAuditAccessAce
AddAuditAccessAceEx
AddAuditAccessObjectAce
AddConditionalAce
AddMandatoryAce
AddUsersToEncryptedFile
AddUsersToEncryptedFileEx
AdjustTokenGroups
AdjustTokenPrivileges
AllocateAndInitializeSid
AllocateLocallyUniqueId
AreAllAccessesGranted
AreAnyAccessesGranted
AuditComputeEffectivePolicyBySid
AuditComputeEffectivePolicyByToken
AuditEnumerateCategoriesBySid
AuditEnumerateCategoriesByToken
AuditEnumeratePerUserPolicyBySid
AuditEnumeratePerUserPolicyByToken
AuditEnumerateSubCategoriesBySid
AuditEnumerateSubCategoriesByToken
AuditLookupCategoryGuidFromCategoryId
AuditLookupCategoryIdFromCategoryGuid
AuditLookupCategoryNameA
AuditLookupCategoryNameW
AuditLookupSubCategoryNameA
AuditLookupSubCategoryNameW
AuditQueryGlobalSaclA
AuditQueryGlobalSaclW
AuditQueryPerUserPolicyBySid
AuditQueryPerUserPolicyByToken
AuditSetGlobalSaclA
AuditSetGlobalSaclW
AuditSetPerUserPolicy
AuditSetSecurity
BackupEventLogA
BackupEventLogW
BuildExplicitAccessWithNameA
BuildExplicitAccessWithNameW
BuildImpersonateExplicitAccessWithNameA
BuildImpersonateExplicitAccessWithNameW
BuildImpersonateTrusteeA
BuildImpersonateTrusteeW
BuildSecurityDescriptorA
BuildSecurityDescriptorW
BuildTrusteeWithNameA
BuildTrusteeWithNameW
BuildTrusteeWithObjectsAndNameA
BuildTrusteeWithObjectsAndNameW
BuildTrusteeWithSidA
BuildTrusteeWithSidW
CancelOverlappedAccess
ChangeServiceConfig2A
ChangeServiceConfig2W
ChangeServiceConfigA
ChangeServiceConfigW
CheckTokenMembership
ClearEventLogA
ClearEventLogW
CloseCodeAuthzLevel
CloseEncryptedFileRaw
CloseEventLog
CloseServiceHandle
CloseThreadWaitChainSession
CloseTrace
CommandLineFromMsiDescriptor
ComputeAccessTokenFromCodeAuthzLevel
ControlServiceA
ControlServiceExA
ControlServiceExW
ControlServiceW
ControlTraceA
ControlTraceW
ConvertAccessToSecurityDescriptorA
ConvertAccessToSecurityDescriptorW
ConvertSecurityDescriptorToStringSecurityDescriptorA
ConvertSecurityDescriptorToStringSecurityDescriptorW
ConvertSidToStringSidA
ConvertSidToStringSidW
ConvertStringSecurityDescriptorToSecurityDescriptorA
ConvertStringSecurityDescriptorToSecurityDescriptorW
ConvertStringSidToSidA
ConvertStringSidToSidW
ConvertToAutoInheritPrivateObjectSecurity
CopySid
CreateCodeAuthzLevel
CreatePrivateObjectSecurity
CreatePrivateObjectSecurityEx
CreatePrivateObjectSecurityWithMultipleInheritance
CreateProcessAsUserA
CreateProcessAsUserW
CreateProcessWithLogonW
CreateProcessWithTokenW
CreateRestrictedToken
CreateServiceA
CreateServiceW
CreateTraceInstanceId
CreateWellKnownSid
CredBackupCredentials
CredDeleteA
CredDeleteW
CredEncryptAndMarshalBinaryBlob
CredEnumerateA
CredEnumerateW
CredFindBestCredentialA
CredFindBestCredentialW
CredFree
CredGetSessionTypes
CredGetTargetInfoA
CredGetTargetInfoW
CredIsMarshaledCredentialA
CredIsMarshaledCredentialW
CredIsProtectedA
CredIsProtectedW
CredMarshalCredentialA
CredMarshalCredentialW
CredProfileLoaded
CredProfileUnloaded
CredProtectA
CredProtectW
CredReadA
CredReadByTokenHandle
CredReadDomainCredentialsA
CredReadDomainCredentialsW
CredReadW
CredRenameA
CredRenameW
CredRestoreCredentials
CredUnmarshalCredentialA
CredUnmarshalCredentialW
CredUnprotectA
CredUnprotectW
CredWriteA
CredWriteDomainCredentialsA
CredWriteDomainCredentialsW
CredWriteW
CredpConvertCredential
CredpConvertOneCredentialSize
CredpConvertTargetInfo
CredpDecodeCredential
CredpEncodeCredential
CredpEncodeSecret
CryptAcquireContextA
CryptAcquireContextW
CryptContextAddRef
CryptCreateHash
CryptDecrypt
CryptDeriveKey
CryptDestroyHash
CryptDestroyKey
CryptDuplicateHash
CryptDuplicateKey
CryptEncrypt
CryptEnumProviderTypesA
CryptEnumProviderTypesW
CryptEnumProvidersA
CryptEnumProvidersW
CryptExportKey
CryptGenKey
CryptGen
[...]
```

Talking about loaded libraries, I have seen in different samples that before performing this action, it tries, in the new path it has created where it launches the file, to load several libraries that it then loads normally, but first it tries to load it from the source path, which creates a big noise in telemetry that is quite accessible for detection.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/b61103a8-0437-4e67-8d86-a0b7525da4ea)
![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/a2f8a300-6542-4769-8ea9-8281a7300aa4)

Summary of all tries:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/c80d965c-1947-47d9-b4a4-e3ab7f0e9ebf)

After this, in all versions of the malware, it somehow retrieves system information. While not its main focus, it collects elements such as the OS version, computer and user names, time, and device language. These elements are usually more for victim identification concerning the command and control (C&C) rather than data that a RAT or pure stealer might extract. Notably, the most interesting information I discovered was the OEM version being used on my machine.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/99845245-f0b9-4c18-b4aa-358a0db99e34)

Subsequently, with the new capabilities loaded into memory, the file written in a temporary path, persistence established, and basic victim machine information acquired, it proceeds with injection. Even in this stage, it doesn't perform any novel actions. It loads the file to inject into memory, opens the file written in the path (which, as we remember, was disproportionately large), and opens it in a suspended state to write to it. It then releases it using ResumeThread, at which point we'll see it running with the filename it dropped earlier

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/cd4b6c53-17fc-4efe-ad7c-cbe7ce7da68a)

After this, we'll observe the malware delving into network functions. I've captured various types of traffic from different samples. Here, it's evident how it makes a request to an address commonly used by LaplasClipper: 

> Clipper[.]guru. 

Following this, it sends data to the attacker, including information about our machine and a generated identifier.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/4cd1553b-b7eb-4e6e-9a1c-97579c60edfd)

Here's what occurs in this phase, having ensured no information slipped through. The malware constantly monitors the clipboard. It uses regular expression (Regex) patterns to detect certain content, as shown below:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/576214d9-cf13-45c3-891e-eb40bc56d160)

Subsequently, with the established connection I mentioned earlier, along with the obtained information, it maintains control of the clipboard. It waits for the victim machine to perform cryptocurrency wallet-related actions. This means that the malware simply waits for one of those patterns to be written to the clipboard. When this happens, it changes the wallet address to one controlled by the attacker. For instance, if you were a LaplasClipper victim attempting a cryptocurrency transaction, the malware would automatically alter the transaction to the attacker's server.

An example I recently came across on Twitter by Jane (follow her at @Jane_0sint) demonstrates this process clearly on Any.Run.

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/e8a7ecde-4b28-4491-b0ff-571433a89cc2)

Following this, having accumulated a considerable number of IP addresses from different analyzed samples and having spent some time researching this malware across networks, I began searching for all these servers.

The first step was to examine the domain it initially connected to, and I found that everything was associated with laplas.app. However, my attempts to access the portal proved unsuccessful, so I dug deeper to locate these servers. It turned out that they had all been moved temporarily. I recall seeing an image like this on Twitter, credited to the cyber colleague Chris Duggan (follow him at @TLP_R3D).

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/db5c7340-37fd-4ce9-9e9e-2e58db57f3dc)

I attempted to check if the situation was the same now, and indeed, I found the same information:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/dd70f1ac-4c4a-4bb3-a4a8-2c90b4986bd5)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/25c9db67-e468-4603-9cfe-f79ae52bb047)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/f1fe3388-3409-4a55-97f9-be1a1cb458df)


I then tried to see if I could reach the servers that the analyzed samples were connecting to (telemetry indicated that everything was fine, but I like to verify everything). I could indeed see both the previously discussed Regex patterns and the requests.

Following this, I began searching for the portal and found both Telegram groups associated with the creators and the web portal. The web portal had moved from Laplas but retained the exact same functionality. It only changed how it is accessed. Internally, it functions the same and sends information in a similar way. However, it's now controlled from a different hosting location:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/9be55bad-bee6-4328-bbdd-9f8bd11261a2)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/7fe8e695-3178-4f07-a8e6-36b0b0e58306)

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/36ceaf34-10c9-48a8-bc18-cc26383737a2)


For better clarity, I've created the following map to consolidate the information:

![image](https://github.com/vc0RExor/Quick-Analysis/assets/91592110/963635c1-4cae-4bfa-8013-05c74a757033)

LaplasClipper has seemed to me a very interesting malware, of which there is not much information, that is why I have ventured into it, I am sure that a large number of criminal groups will make use of it, as it works quite fast and is quite stealthy, we will keep track of this malware and the use they make of it, as well as if new versions appear to keep getting detection possibilities.

Finally, I would like to thank you for reading this analysis and for supporting me :)

# _TTP

```
[T1140] Deobfuscate/Decode Files or Information
[T1027] Obfuscated Files or Information
[T1547.001] Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder
[T1129] Shared Modules (No estoy seguro, para las cargas de librerias en runtime)
[T1095] Non-Application Layer Protocol
[T1106] Native API
[T1543] Create or Modify System Process
[T1082] System Information Discovery
[T1055] Process Injection
[T1115] Clipboard Data
```

# _IOC

```
78362eb5c4529001a3bc7ecab62b904afef81c63b6778ba00b99eb3398140dab
11c3e7a62b3e78c6ec720aea618bf0a3854ad42535f888532c3e206f3724db4c
4230177379ff0422741a5714ba02dbeccdac0edc6d2c1e4123827f23ff179e64
bdf326424f960a66d01dd645db9fd335a157ceb86d7f482ff15205fa7d9cc7b0
f022037056b50b4baf5db8ba0a437494662dc93cee9421ed12471e14a58a0d50
22b198c5fc1e073ef00fc7a44ca20db5f44630f4e0e746abcf2060207d7129d9
d30e2337e87b5bad478d20dea2fa51d38a4a9506542bdaaea7640dcc68a4432c
a3f665043305d67f64f7386a8bcd89dc5ce86a76a6b5042827af58cd8b4e10f2
17ca2de661fa07dd83a55a5005c61eb8aee1e9cab56e9a13bc36a27f4b785554

clipper[.]guru

85[.]192[.]40[.]252
206[.]189[.]229[.]43
185[.]209[.]161[.]61
168[.]100[.]10[.]236
45[.]66[.]230[.]149
```

> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:

