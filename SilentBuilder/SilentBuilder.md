# _Overview

_SilentBuilder is a campaign that is being used to launch bankers such as Emotet to increase the Epoch5 botnet as well as the usual tasks of this malware. The similarities between other loaders that launch Emotet and that, once the banker is in our computer, tries to contact C&C, we could understand that this is a typical modus operandi of the criminal group Mummy Spider or TA542._

# _How it Works

The attacker will access our system after a phishing email, more specifically SpearPhishing [T1566.002], as it will contain an attachment such as an XLS or DOCX, after this, since the document will contain macros or hidden functions, it will download a file, usually a dll, once downloaded it will be launched on the computer abusing Regsvr32.exe to search a list of C&C servers.

![0](https://user-images.githubusercontent.com/91592110/165115747-2f771d13-1043-4af2-825c-a55c5ae62601.png)

# _Static Analysis

Once the document is downloaded, we find, in my case, an xls, which after a glance we can see that it contains interesting functions that will run automatically when opened.

![1](https://user-images.githubusercontent.com/91592110/165120891-d9d71949-a3a3-4926-be9b-9ca350d44b4e.png)

An interesting fact is that in this sample we see that it has the usual warning that will launch the functions and, in addition, another warning made by the attacker that will be a simple image.

![2](https://user-images.githubusercontent.com/91592110/165121338-1c2a8930-f0f8-40e5-9826-813890d61234.png)

After this, we see a completely blank document, with no pages, no macros… Inquiring, we see that it does have internally pages with characteristic names and that they were hidden.

![3](https://user-images.githubusercontent.com/91592110/165122164-89f7db54-ba5e-4d0a-94dc-810e1ebda71b.png)

Once again the sheets are empty... After reviewed the document and by changing the color of all the pages we found all the functions obfuscated and disordered.

![4](https://user-images.githubusercontent.com/91592110/165122974-c0cdd14b-01ff-4923-9d28-78c60223957e.png)

In one of the sheets, we find the most important function, which would deobfuscate most of the functionality that will have the functions of the document.

We obtain, as we can see, functionalities for downloading a supposed library (nhth.dll) from different domains:

![5](https://user-images.githubusercontent.com/91592110/165123309-199782a9-80bb-4d94-90ef-eabc19f95af3.png)

# _Dynamic Analysis

_Once we have an idea of how the document is going to work internally, let's check if we are right._

We see that once the excel is launched, it makes a request to a domain and downloads a file (we observe in the network traffic the MZ header typical of Windows PE). After this, we can see that it downloads the dll in \users\\< YourUser >\\ , and then it will move it to the path \AppData\Local\\< RandomName >\\ with another name < RandomName >.adj

![image](https://user-images.githubusercontent.com/91592110/165127834-d79cec5f-9b27-482d-97d2-df88463f176a.png)

We can see that if we compare the file obtained from the network traffic and the one found in \users\ or in \AppData\ , it is the same file.

![image](https://user-images.githubusercontent.com/91592110/165128128-1d3fd203-18f4-407d-b00b-461d3a4a3512.png)

After this, we will see that the dll will try to contact a list of C&C servers.

![image](https://user-images.githubusercontent.com/91592110/165128479-8e606913-199b-41f1-adec-fc5831206764.png)

If we look at the origin of all the addresses it tries to contact, we can see that it has servers in most of America, Europe and Asia, among others.

![7](https://user-images.githubusercontent.com/91592110/165128942-55d00be4-4d06-4b20-84b3-41377f135589.png)

How long will they continue to exploit Emotet? Who knows...

# _IOC

### _Download Emotet

```
fccatinsaat.com
freemanylaluz.com
futaba.youchien.net
fabulouswebdesign.net
freewebsitedirectory.com
dominionai.org
```

### _C&C

```
159.203.141.156
79.143.187.147
189.232.46.161
51.91.76.89
119.193.124.41
176.104.106.96
1.234.21.73
82.165.152.127
167.172.253.162
153.126.146.25
216.158.226.206
103.75.201.2
188.44.20.25
101.50.0.91
159.65.88.10
176.56.128.118
72.15.201.15
203.114.109.124
212.237.17.99
192.99.251.50
50.30.40.196
173.212.193.249
189.126.111.200
195.154.133.20
58.227.42.236
46.55.222.11
45.176.232.124
195.201.151.129
151.106.112.196
209.250.246.206
131.100.24.231
1.234.2.232
164.68.99.3
51.91.7.5
167.99.115.35
5.9.116.246
185.8.212.130
31.24.158.56
45.142.114.231
79.172.212.216
45.118.135.203
146.59.226.45
178.79.147.66
159.8.59.82
158.69.222.101
50.116.54.215
196.218.30.83
129.232.188.93
45.118.115.99
51.254.140.238
209.126.98.206
107.182.225.142
134.122.66.193
185.157.82.211
110.232.117.186
197.242.150.244
103.43.46.182
212.24.98.99
201.94.166.162
104.131.11.205
138.197.109.175
187.84.80.182
206.189.28.199
160.16.142.56
183.111.227.137
103.132.242.26
103.70.28.102
172.104.251.154
```


> :t-rex: [vc0=Rexor](https://github.com/vc0RExor)  :detective:
