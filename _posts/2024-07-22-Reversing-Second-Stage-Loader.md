---
title: "IcedID - Reverse Engineering Second Stage Loader"
date: 2024-07-22
---


### Description
---

<div style="margin-top: 20px; margin-bottom: 20px;text-align: justify;">
	

In the ever-evolving landscape of cyber threats, the IcedID loader stands out as a formidable banking Trojan with sophisticated capabilities. Originally known for its role as a banking malware, IcedID has grown to serve as a versatile loader for deploying additional malicious payloads. This analysis delves into the intricate mechanisms of IcedID, from its initial delivery through phishing or exploit kits, to its operational intricacies and data exfiltration methods.

Key areas of focus include the loader’s evasion techniques, persistence strategies, and communication with command-and-control servers. By dissecting IcedID's functionality and evolution, this analysis aims to shed light on its impact on targeted systems and provide insights into effective detection and mitigation strategies.

</div>

### Basic Static Analysis
---

<div src="margin-top: 20px; text-align: justify;">
	

In this section I used a simple tool named `pe studio` to get a general understanding of the loader. After unpacking I also used a command line tool named `floss`

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240716223035.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
- In this screenshot we can clearly see that `tudio` is not able to properly counts the imports and the libraries which indicates to me that this sample might be packed
- also we can note that there is a high entropy. I will have to unpack it to be able to proceed my analysis

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240716223435.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
- We can also see that the raw size is lower than the virtual size which might be another indicator that this sample is packed

> ⚠️ Before proceeding further in our basic static analysis we will perform the unpacking steps with x32bdg, since we are focusing on the analysis, I will not put the process in this blog

- Interesting strings, Before unpacking
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240721140409.png" style="margin-left: 20px;box-shadow: 10px;display: block;border: 2px solid transparent; border-radius: 8px;" > 
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
</div>

### Basic Dynamic Analysis
---

<div src="margin-top: 60px;"></div>
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717061554.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
- By detonating the malware we notice that it tried to create a file named `photo.png`
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717062937.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
- We can also see that a `TLSv1.2` connection is made to `boldidiotruss.xyz`




<div src="margin-bottom: 60px;"></div>

### Advanced Static Analysis
---

<div src="margin-top: 60px;"></div>

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717081349.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 

- Here we can see that loader is calling the function `SHGetFolderPathA` and by looking at the documentation I can see that the second argument indicates the folder of interest in this case it is aiming for `CSIDL_COMMON_DESKTOPDIRECTORY`, which represents a common folder that appear on the desktop for all users and the typical path is `C:\Users\Public\Desktop`

- We can also see that based upon the result the content of `pcVar5` will be different and at the end it will be concatenate to the folder path

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717081349.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 

- After that, we see that the `GetUsernameA` is used to fetch the current user, we also see that there is an attempt to create the directory
- At the end, we can see that the string `\\photo.png` is appended to the current `folder_path` string
  
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717085042.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 

- We can see the file path that was constructed earlier is used to create a file
  
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717085910.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 

- There's some VM detection being performed by checking information about the CPU running the sample


  > ⚠️ Note that some functions have been renamed by me to facilitate the clarity of my analysis

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717090929.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
	
- After the VM detection, there's string formatting operation that is performed to construct the URL path to the `.png` file and also to the URL hostname
- we can see that the result of the concatenation of the full URL is being sent to a function that will handle the communication with the possible the command & control center

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717092802.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
	
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240717092848.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 
	
- In these screenshots, we can clearly see the code the of the communication  with the C2 
- Based on my analysis I deduct that a call his being made to verify that the command and control is active and running correctly, because of the parameters of the functions I can see that no data is actually being sent since it's only performing a get request

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240721122821.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 

- Now, I thought to myself why would they be communicating via an URL that looks like you're fetching an image? Because, in reality you're really just communicating, sending and receiving information stealthily. Think about it, if you look in Wireshark and see some request made to this URL you would only think that a picture his being fetched, but in reality, you're data is probably being exfiltrated. I might be wrong in my analysis but that is what I get from it

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240721123829.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" > 

- By looking at everything from a higher perspective, here's what can I make of the general process of this Loader
	1. We create the path where we will store the malicious file
	2. we decrypt probably the shellcode that will be stored in that file temporary
	3. we create the file picture
	4. we decrypt something
	5. we check if the command & control center is active
	6. we write in the data that decrypted in that file
	7. we setting up the shellcode in memory and then we execute it


> ℹ️ In my basic dynamic analysis we don't see the call made to the url in question, that may be because the malware is able to detect that I'm using a vm, so it does not pursue the communication, we will confirm it later when we will be doing the Advanced Dynamic Analysis with our favorite debugger x32dbg.

<div src="margin-bottom: 60px;"></div>

### Advanced Dynamic Analysis
---

<div src="margin-top: 60px;"></div>

<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240721131650.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" >
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240721131953.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" >
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240722105526.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" >
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240722110943.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" >
<img src="https://b3rg01.github.io/MyBlog/docs/assets/Pasted image 20240722111231.png" style="margin-left: 20px;margin-top: 20px;margin-bottom: 20px;box-shadow: 10px;border: 2px solid transparent; border-radius: 8px;" >

<div src="margin-bottom: 60px;"></div>

### Rules & Signatures
---

<div src="margin-top: 60px;"></div>

```
rule IcedId_Ldr_Detection {
    
    meta: 
        last_updated = "2024-07-21"
        author = "8erg"
        description = "Yara detection rule for IcedId Loader"

    strings:
        $string1 = "Now Wardoor"
        $string2 = "Now Wardoor.exe"
        $string3 = "c:\Sizeanger\CreatePick\mixpractice\Sciencescience\KeyContain\farterm\Tiresubtract\CenterSkinMass.pdb"
        $PE_byte = "MZ"
        $magical_string = "1023442870282056"
	    

    condition:
        $PE_byte at 0 and
        ($string1 or $string2 or $magical_string)
}
```

<div src="margin-bottom: 60px;"></div>


### Conclusion
---

<div src="margin-top: 60px;"></div>


