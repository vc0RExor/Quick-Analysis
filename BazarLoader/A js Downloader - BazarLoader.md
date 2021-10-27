# _Overview

_BazarLoader is known to be used in campaigns by cybercriminal groups, such as WizardSpider, used to perform what is commonly known as InitialAccess (TA0001), usually sending emails containing malicious documents or scripts, with the objective, to download or execute malwares such as Ransomware._

# _Technical Anlysis

We start from a script, specifically JavaScript(JS) that could be launched through a fake ZIP used in phishing campaigns in which, we introduce a fake file with the appearance of a document or a compressed with an invitation message or UPS/FedEx mail, and so on. That will make us download and have it on disk, which is its main stage to achieve its next steps.

The file is quite obfuscated but we focus on the possibility of an eval that will launch the content or part of the content of the script

![image](https://user-images.githubusercontent.com/91592110/139117972-f89cdb86-d476-4225-a1a9-fe589e9cfa4f.png)

![image](https://user-images.githubusercontent.com/91592110/139118019-e0f37dfc-396c-4f25-955d-39ffcea28596.png)

After executing the JS, it tries to open a shell and execute an encoded PS, it tries to make a request to a malicious URL, which, by means of an _IEX_ (Invoke-expression) will execute a download using _"downloadstring"_.

![2](https://user-images.githubusercontent.com/91592110/139118829-c6e74179-b738-4965-97bc-0bcbd8401201.png)

We can see that this URL [_hxxp://menoiras[.]space_] is related to different malicious files, as well as reported by several entities.

![image](https://user-images.githubusercontent.com/91592110/139119166-52c5ec05-e6d5-41cf-9501-c79523392660.png)

![image](https://user-images.githubusercontent.com/91592110/139119176-f3f9f7e9-325d-4207-952d-09446ea0077f.png)

We also see that the download goes through 2 points, one in which it will try to perform a download on the path index.php in the previous URL, which, as we see, will perform a download in temp of a _.dat_ against _main.php_ and then run the _rundll32.exe_ and start it with _StartW_, which will be a common export in the binary.

![image](https://user-images.githubusercontent.com/91592110/139119594-d49ce1e0-20c4-4734-8ca1-9552723b4b02.png)

We verify that the file we have in the captured package and the one we get from the download is indeed the same:

![4 1](https://user-images.githubusercontent.com/91592110/139121109-6d30758b-1221-4e19-b0ff-03ab52176598.png)

Subsequently, it executes the dll using the _rundll32.exe_ which is commonly known as the BazarLoader backdoor (_BazarBackdoor) that will perform from here _CobaltStrike_ movements and we can see anomalous traffic and recognition phases in our computer, which could trigger a Ransomware that, considering the APT could be a Ryuk or a Conti, both this step and the previous one may change the version as variations of BazarLoader are used.

![image](https://user-images.githubusercontent.com/91592110/139120287-bb7a12f9-3e6e-482b-8f0a-dc647c11fde8.png)

In these phases after the infection of the PS download we can see different malware associated with this campaign, from Trickbots, through CobaltStrike and computer exploitation, to Ransomwares in more advanced stages.


> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:

