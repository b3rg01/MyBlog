---
title: "Reverse Engineering WannaCry Initial Stage"
date: 2024-07-16
---


### Description
---

In the early summer of 2017, WannaCry was unleashed on the world. Widely considered to be one of the most devastating malware infections to date, WannaCry left a trail of destruction in its wake. 
WannaCry is a classic ransomware sample; more specifically, it is a ransomware crypto worm, which means that it can encrypt individual hosts and had the capability to propagate through a network on its own.
Here’s my own analysis of this particular specimen.

In this blog post, I will analyze the initial stage of WannaCry ransomware sample. Please note that I'm new at this and I will try to provide a detail technical analysis of the initial stage to the best of my abilities. 
Hope you will enjoy this!!


###  General Details
---

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240709173746.png" style="margin-top: 20px" > 


### Basic Static Analysis
---

*The full list of images can be found in the Appendices.*

>  For this part I used a command line tool named floss to get the strings from the ransomware executable.

- By looking as some of the strings, we can see some interesting library related to encryptions such as:
	- `CryptAcquireContextA`
	- `CryptGenRandom`
	- `CryptServiceA`
	- `CryptReleaseContext`
	- `CryptGenKey`
	- `CryptDecrypt`
	- `CryptEncrypt`
	- `CryptDestroyKey`
	- `CryptImportKey`
- We can also see that the malware will probably modify or add some registry based on the library imported
	
- In the strings, we can also find words such as :
	- `mssecsvc.exe` (already flagged for been used for ransomware infection)
	- `tasksche.exe`
- We also see alot of base64 encoded strings
- We can see some randomly named folder that will dynamically named on runtime
	- `C:\%s\%s`
	- `C:\%s\qeriuwjhrf`
- We can also some interesting urls, such as :
	- `http://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com`
- An interesting string that let's us know for sure that we're dealing with the WannaCry malware : 
	- `c.wnry`
	- `t.wnry`
	- `msg/m_croatian.wnry`
	- `msg/m_dutch.wnry9`
	- `msg/m_english.wnryF`
	- `msg/m_korean.wnry`
	- `msg/m_latvian.wnry`
- Interesting strings:
	- `115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn`
	- `12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw` 
	- `13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94`
- Suspicious commands:
	- `icacls . /grant Everyone:F /T /C /Q` (Granting everyon acces to ACL)


### Basic Dynamic Analysis
---

- When triggering the malware with internet nothing happens, as far as encrypting my files, but I see that some calls are made to a malicious url, probably to drop something else on my file system
- When triggering the malware without internet connection, i can see that my file system get automatically encrypted
- There's seems to be a lot of call to different DNS

*Triggering the malware with fake internet simulation*
