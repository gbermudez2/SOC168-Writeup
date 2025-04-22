# SOC168 Writeup - LetsDefend
## Command Injection Vulnerability

This is a writeup for a Command Injection vulnerability, determining whether it's a true/false positive.
For this case, we are presented with a rule trigger for Command Injection.

![image](https://github.com/user-attachments/assets/043178a1-bf09-494e-ab8a-e4b78ec8e9c8)

It's important to take notes on important details of the case, including possible IoCs:
- Event Time: Feb 28, 2022, 04:12 AM
- Source IP: 61.177.172.87
- Destination IP: 172.16.17.16
- The requested URL (https://172.16.17.16/video/) doesn't tell us much, so we need to dive into the logs.

## Researching IP Reputation

I decided to take the Source IP and research its reputation on VirusTotal.

![image](https://github.com/user-attachments/assets/cd44da50-bad1-4c9f-a36a-aab51d1e7a90)

- 5 vendors flagged the IP as malicious. Not a good sign!

Let's check AbuseIPDB for clarification and user reports:

![image](https://github.com/user-attachments/assets/0f7a295c-4241-435b-8e6e-81e2d3572993)

- *87,440 reports!* That's a staggering number.
- The user reports have a pattern of brute-forcing attempts and SSH attempts. **It's more than safe to assume this is a malicious IP.**
- The domain chinatelecom.cn inherently isn't malicious, which is likely meant to throw analysts off.
- **By this logic, it's also safe to assume that this is NOT a planned test.**

## Utilizing a basic search with the source IP:

To look through logs, we can use the source IP from the case information to perform a Log Management search.

![image](https://github.com/user-attachments/assets/625f906f-52cb-4ab1-8201-ed32de15fe99)

The search returns 5 logs involving the source IP.

If we look at the log that triggered the security event, we can see the HTTP response info.

![image](https://github.com/user-attachments/assets/ea9cc822-3904-4f60-a59d-8640478e46fc)

- The HTTP POST parameters show the command injection the attacker used: ?c=whoami
- The attacker attempted to make the server use the command "whoami" to print out the user, presumably to assess the privileges of the server.

**It's not easy to tell if these attacks were successful, so I decided to take a look at the following logs:**

![image](https://github.com/user-attachments/assets/03cc9b43-a4cc-41fb-89dd-78b4d1a3c746)
![image](https://github.com/user-attachments/assets/99eb22b5-df5d-4f1a-bc82-da580d9110b7)
![image](https://github.com/user-attachments/assets/0eb81579-99c3-4982-ac51-3198d6d7d5cb)

- The same pattern of commands is in the HTTP POST parameter.
- The attacker uses "uname" to view system info.
- "cat /etc/passwd" was used to list all the users' info on the system.
- "cat /etc/shadow" was used to list the encrypted passwords of all the users.
- _This is also a reliable sign if an attack is malicious, since the passwd and shadow file are prime targets for attackers to exfiltrate credentials._

**Now, based on these commands, we can look at the targeted endpoint, 172.16.17.16, and see if they were executed successfully:**

![image](https://github.com/user-attachments/assets/4ad7d9e1-858b-4e06-995f-8e021cecfcf4)

- **The commands were executed under root on the web server.** The attack was successful!
- The web server has to be contained.

![image](https://github.com/user-attachments/assets/4d52dfa7-99fb-4a67-a465-41f79c96ea08)

## Case Report

In our report, we have to include the Attacker IP, and the web server that the attacker targeted:

![image](https://github.com/user-attachments/assets/63c316cb-a747-49e0-bd46-8ff4853ef9af)

![image](https://github.com/user-attachments/assets/557e9f8b-e88b-4a10-8b26-618bd0fa8aee)

### Success!

