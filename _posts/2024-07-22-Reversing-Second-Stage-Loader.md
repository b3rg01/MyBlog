---
title: "Reverse Engineering WannaCry Initial Stage"
date: 2024-07-16
---


### Description
---

<div src="margin-top: 60px;"></div>

In the early summer of 2017, WannaCry was unleashed on the world. Widely considered to be one of the most devastating malware infections to date, WannaCry left a trail of destruction in its wake. 
WannaCry is a classic ransomware sample; more specifically, it is a ransomware crypto worm, which means that it can encrypt individual hosts and had the capability to propagate through a network on its own.
Here’s my own analysis of this particular specimen.

In this blog post, I will analyze the initial stage of WannaCry ransomware sample. Please note that I'm new at this and I will try to provide a detail technical analysis of the initial stage to the best of my abilities. 
Hope you will enjoy this!!

<div src="margin-bottom: 60px;"></div>


### Basic Static Analysis
---

<div src="margin-top: 60px;"></div>

In this section I used a simple tool named `pe studio` to get a general understanding of the loader. After unpacking I also used a command line tool named `floss`

 <img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240716223035.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;" > 
- In this screenshot we can clearly see that `pestudio` is not able to properly counts the imports and the libraries which indicates to me that this sample might be packed
- also we can note that there is a high entropy. I will have to unpack it to be able to proceed my analysis

 <img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240716223435.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;" > 
- We can also see that the raw size is lower than the virtual size which might be another indicator that this sample is packed

  >  Before proceeding further in our basic static analysis we will perform the unpacking steps with x32bdg, since we are focusing on the analysis, I will not put the process in this blog

- Interesting strings, Before unpacking
 <img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240721140409.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;" > 
- After unpacking, I can see some interesting parts of some URLs
	- `/photo.png?id=%0.2X%0.8X%0.8X%s`
	- `boldidiotruss.xyz`
	- `nizaoplov.xyz`
	- `153ishak.best`
	- `ilu21plane.xyz`
- I can also see some interesting libraries being imported which indicates to me that something is being downloaded
	- `WinHttpQueryDataAvailable`
	- `WinHttpConnect`
	- `WinHttpSendRequest`
	- `WinHttpCloseHandle`
	- `WinHttpSetOption`
	- `WinHttpOpenRequest`
	- `WinHttpReadData`
	- `WinHttpQueryHeaders`
	- `WinHttpOpen`
	- `WinHttpReceiveResponse`

<div src="margin-bottom: 60px;"></div>

### Basic Dynamic Analysis
---

<div src="margin-top: 60px;"></div>
 <img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717061554.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;" > 
- By detonating the malware we notice that it tried to create a file named `photo.png`
 <img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717062937.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;" > 
- We can also see that a `TLSv1.2` connection is made to `boldidiotruss.xyz`




<div src="margin-bottom: 60px;"></div>

### Advanced Static Analysis
---

<div src="margin-top: 60px;"></div>


### Advanced Dynamic Analysis
---

<div src="padding-top: 60px;"></div>


### Rules & Signatures
---

<div src="margin-top: 60px;"></div>


### Conclusion
---

<div src="margin-top: 60px;"></div>


