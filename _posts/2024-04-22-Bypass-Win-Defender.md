---
title: "Bypass Windows Defender With Xor Encryption"
date: 2024-04-22
---

# Prelude
If you're new in malware development, just like me you might've heard that it was possible to bypass windows defender with a simple encryption. 
I could not believe it, so I had to try it and why not put in a blog to describe my experience at the same time!
In this blog I will take you through it step by step. At the end, I will also introduce a clever way of fooling a user into clicking on our malware that will appear as an image.

# Environment Setup
If you want also want to perform this simple attack you will need to have to virtual machines
  - Windows Machine (victim)
  - Kali Linux or any other machine with metasploit on it (Attacker)


