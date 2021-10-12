# _ Initial access: Phishing

This PDF comes to our systems using Phishing techniques, it's common to see malicious files like docx, pdf, ppt, that tries to be downloaded using an UPS mail or a new policy on your Bank. Usually you will be expecting a shipping or it's not rare to see new changes on an account (Google, Amazon...) which are fake mails. Once the PDF arrives to you mail, it tryies to be downloaded cheating on you, later, when it's downloaded, it has disk access and if you launches the PDF using a pdf reader (Like acrobat, foxit...) It will run the malicious activity.

# _Overview

In the event that the user has not noticed that the mail is a fake URL or it's not a real mail, if you have already downloaded the PDF on your computer, we can see that it contains suspicious strings. We find interesting tags, URLs, pdf names, and so on, later we will analyze it, but we can start gathering IOC. As you can see, these URL are HTTPS, that's because is better to use a lot of hosts on a 443 domains and introduce paths used to malware, that will be closed after a few days, instead of use HTTP or random domains that some IPS/IDS will block instantly

![100000000000018C0000010CA4AF646E2A5294F8](https://user-images.githubusercontent.com/91592110/136981977-3d7eb64e-73f4-4dbf-9cb6-b9371d437539.png)

There are detected elements, but we have a lot of URLs to create blacklists and to add at the IOC

![100000000000029C000000D18992B14A95BDD269](https://user-images.githubusercontent.com/91592110/136982040-752d1ff3-5832-4e06-90d3-f83a368f793e.png)

In addition to URL and pdf names, we have several streams coded using different ways like /FlateDecode, it uses /Filter to give to the PDF information about how decode the chunk bytes, it's common to see these filters, using that you can hide domains, scripts and you can't see these using string or similar tools... let's say it makes analysis harder...

![100000000000009300000066EAE1BE83956AAF78](https://user-images.githubusercontent.com/91592110/136982073-c0db39a0-8eee-4a7e-b16f-815b4297f168.png)

Now, we have several indicators that tell us that this could be a malicious PDF, we need to investigate on behaviour and later we will analyze these tags internally.

# _Behaviour

Once we have a background where we know that this PDF will do malicious behaviour we need to detect how it works.

At runtime, the PDF will open several instances and it opens a window where it tries to check that we are not a robot, this is a FAKE authenticator because this is not a real form, it's a picture that tries to go to a cctraff[.]ru

![100000000000011D000000A999D35EC779D1DC73](https://user-images.githubusercontent.com/91592110/136982190-b430d2d1-7a23-4ede-9ead-af3486879bf1.png)

![behaviour](https://user-images.githubusercontent.com/91592110/136982493-a5f3e914-c8f9-4402-9c09-2a8351713034.png)

This PDF has 2 pages, at the second one, we have some paths, pdf names and domains and some of them are already detected

![100000000000011E000000F25419F8ECE5E37D6A](https://user-images.githubusercontent.com/91592110/136982304-415aa384-8dcd-48d1-84d6-532ac825c90b.png)

![10000000000002A8000000D27E26B1C05CCF0BC9](https://user-images.githubusercontent.com/91592110/136982317-06e93412-80d2-4e40-a6f2-97662169e379.png)

After we send the FAKE authenticator, it will take us to another fake authenticator (robotcheckion[.]online) who wants to allow notifications, but this is used to send us to a malicious web pages 

![10000000000002C500000274994292EA85D51EE1](https://user-images.githubusercontent.com/91592110/136982358-54235596-e60c-4b31-98a5-5741d71772d7.png)

Once launches the authenticator, we will find several URLs that tries to download another files or PDFs

![10000000000002010000001781968E2C8A449B6C](https://user-images.githubusercontent.com/91592110/136982522-85ba563e-3b36-46a7-bfb9-04e17afaf8ed.png)

![10000000000001A1000000203BD689BACD3B4360](https://user-images.githubusercontent.com/91592110/136982544-5f154188-60ef-4fab-a19d-5b984096b59e.png)

Sniffing and analyzing network, we can see a lot of chrome.exe instances (Chrome uses a lot of instances as you know but this malware uses many URL) to tryies to download any file, that's not rare because these malware could be catched fastly and needs to have some webpages in case any enterprise block them.

But we can see the URL access and the Fake authenticators:

![100000000000011A00000089197AEB424ACFB230](https://user-images.githubusercontent.com/91592110/136982631-78fb39f7-f794-41b5-aee4-5cfbb0011658.png)

![10000000000001030000007A13EDF8F71867B116](https://user-images.githubusercontent.com/91592110/136982644-26b603d6-a647-4e44-9d3d-c30ad7589205.png)

It's normal that when you are analyzing the sample you can't find any active URL or you need to search among of Web Pages that contain the malware, but you only need to be patient and find one that download the file to complete your analysis (If you have the downloadable file names, you can search out in [VirusTotal](https://www.virustotal.com/gui/home/search) or [AlienVault](https://otx.alienvault.com/browse/global) if all the domains are already closed)

Talking about registers, the malware adds some byte to chrome.exe using UserAssist and erase caches (That's not rare), but we don't have any persistence trick on this, this which would be the most worrying, not occurs, in case you did not know, count on UserAssist will be do using ROT-13, this practise and modify extensions value, you can see it on some Adware, but we don't have rare behaviours related with RegKeys on our Malware.

An example of ROT-13:

![1000000000000424000000121C900F137E0AF2F3](https://user-images.githubusercontent.com/91592110/136982941-59785539-e547-451e-a31b-7e30c7e62536.png)

![100000000000013E000000B4C3D61CB4A751E40C](https://user-images.githubusercontent.com/91592110/136982959-e81c1ff5-b6bc-44b3-9a87-ae6eac5330ba.png)

The behaviour it's rare to a legit PDF, now we have a good background to locate several URLs and we have information on how it works dynamically and we goes to an internal static analysis to know if this is the only thing that it does

# _Static Analysis

Now we need to know if the functionality it's based on search among URL list and download another files and PDF malwares or if there is something else.

First of all we see the PDF tags and we have interesting elements, some objects, URI (We're not surprised) and we have good news, no JavaScript there, for what it's possible that we are facing a Malware PDF Downloader

![10000000000000DD0000011CAA2C9B97155C111D](https://user-images.githubusercontent.com/91592110/136983050-f6e7bb93-3b5d-4dce-8c9e-49cd37b351e3.png)

We found several elements related with colors, buttons and so on, that's the graphic components of the PDF (The first fake Authenticator) some /Filters using /FlateDecode to decompress streams

![10000000000000BF000000C2451C5876AE498FEB](https://user-images.githubusercontent.com/91592110/136983104-56e388e9-4d14-4e58-a687-4934d8d8ccdf.png) ![1000000000000095000000A9D760ACC89F85AE1D](https://user-images.githubusercontent.com/91592110/136983123-87b9e62e-a4aa-4883-ac75-c479d708ae4d.png)

Later, using /Annot, we have reference to URI that we are looking for (Showing the object 17 and 19):

![10000000000002A3000000CAE6416424A280BCA2](https://user-images.githubusercontent.com/91592110/136983208-d902e43c-9c50-4844-a2bc-78a69881e7d1.png)
![1000000000000296000000D16E8A85589E004D88](https://user-images.githubusercontent.com/91592110/136983257-676a3bd5-0105-4c8d-a3c0-83a107a3ab47.png)

Also, all of URL are referred by the Object 40, as you can see, each of URL are represented by an obj, among values 17 and 32 (As you can see at the picture below) any object can download other Malwares referring to different domanins and files (Previous image)

![100000000000034A0000003AAC2E640B9EA7241A](https://user-images.githubusercontent.com/91592110/136983324-f22399a8-3546-47d9-b3f7-9192020bdf6e.png)

Into the PDF we don't have any more interesting , we have a lot of URL, PDF and file reference, and the rest are graphic content such as Fake authenticators, fonts, buttons, and so on.
This time, unfortunately, we don't have elements as JavaScript, because this Malware is based on a Downloader, Its purpose is to mislead the user into believing that this PDF is a portal to other web pages and that needs to verify that you are not a robot by making forms looking like a real authenticator in which will be downloaded using other malicious web pages other files or PDFs. To these Malwares typically using Phishing techniques it's advisable a good enterprise policy, avoiding specific domains and training workers so that they distinguish a real mail to a fake mail, let's not forget that the greatest weakness is the user.

# _IOC

```
hXXps://cctraff[.]ru/aws?keyword=download+pdf+to+excel+converter+full+version
hXXps://cdn.shopify[.]com/s/files/1/0432/0319/9138/files/gogigisoladavatafanitu.pdf
hXXps://cdn.shopify[.]com/s/files/1/0433/8050/6773/files/zobusijamojanag.pdf
hXXps://cdn.shopify[.]com/s/files/1/0268/8394/8740/files/rojiriloxiwuruzobo.pdf
hXXps://cdn.shopify[.]com/s/files/1/0268/8529/2223/files/bezesube.pdf
hXXps://cdn.shopify[.]com/s/files/1/0268/8529/2223/files/bezesube.pdf
hXXps://cdn.shopify[.]com/s/files/1/0437/6199/1841/files/sovavejipizivaduf.pdf
hXXps://s3.amazonaws[.]com/henghuili-files/aparato_reproductor_femenino_funciones.pdf
hXXps://s3.amazonaws[.]com/mijedusovineti/31436972639.pdf
hXXps://s3.amazonaws[.]com/henghuili-files/93071167754.pdf
hXXps://s3.amazonaws[.]com/jepinebawo/anatomy_knee_joint.pdf
hXXps://uploads.strikinglycdn[.]com/files/d11907ef-8a0e-4bb7-b17c-9ae1ff5d9692/pixazubepojo.pdf
hXXps://uploads.strikinglycdn[.]com/files/7e1ef169-b688-4b2c-b347-d5a633848cdd/21685658051.pdf
hXXps://uploads.strikinglycdn[.]com/files/fbc2e3e8-f23a-4c63-97f3-e0be6e5efc54/67514968450.pdf
hXXps://fidurelofomus.weebly[.]com/uploads/1/3/0/7/130740547/zumuxojezudag.pdf
hXXps://xesaranit.weebly[.]com/uploads/1/3/2/6/132696194/divuwixudibar.pdf
hXXps://cdn-cms.f-static[.]net/uploads/4389616/normal_5f900636493b8.pdf
hXXps://cdn-cms.f-static[.]net/uploads/4369494/normal_5f8b051093f1f.pdf
```


> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:
