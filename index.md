
**Active Directory • Splunk • Sysmon • Kali Linux • Atomic Red Team**

## Project Overview

Built an Active Directory security lab to simulate credential-based attacks, collect Windows security telemetry, and investigate malicious authentication activity using Splunk and Sysmon.

**Environment:** Windows Server • Windows 10 • Kali Linux • Splunk Enterprise • Sysmon • Splunk Universal Forwarder • Atomic Red Team • Oracle VirtualBox

**Security Skills:** Active Directory • SIEM Monitoring • Log Analysis • Threat Detection • Windows Event Logs • Attack Simulation • Incident Investigation







# Lab Diagram

![AD Project drawio](https://github.com/user-attachments/assets/7f01f634-2d8e-44f5-a0c4-7692bd9f230f)

This diagram illustrates the lab setup and the data flow between each system. 

![862bf9d882d38eda731787348bf9deb7](https://github.com/user-attachments/assets/626561bd-6e50-4059-9897-921c8c2a1360)

I used Oracle VM to host all the servers and connected them on a NAT network

# Active Directory Configuration

For my windows server I set a static IP address (192.168.10.7) and verified connectivity to my Splunk server via the command line. In Server Manager, I installed Active Directory Domain Services and set the domain name to JSP.Local. In Active Directory Users and Computers, I created two users, Terry Pond and Jerry Smith, as well as two groups, HR and IT. I also installed Sysmon to monitor system activity and configured the Splunk Forwarder to send data to my Splunk server.

![fb0419984e90344f1ad6a5fd6019b181](https://github.com/user-attachments/assets/25d5e680-4fe7-4300-9583-97110a126a1e)
![d53fc56ed78a76a346b76140323e06f8](https://github.com/user-attachments/assets/bb307381-4cfc-4575-b677-ce52bb69015d)
![1b2761e5a42d6e81865eac4a337dc1a3](https://github.com/user-attachments/assets/2f47482c-5d88-4ad1-bddb-ab4c5bff2c76)![758a02ea5baacbab939203bdfa6704b1](https://github.com/user-attachments/assets/919af5f9-610d-418b-a217-f7c72e5c0179)














# Splunk Server Configuration

On the Splunk server, I opened the /etc/netplan/00-installer-config.yaml file to configure the network and set a static IP (192.168.10.0/24). After installing Splunk Enterprise, I ran sudo ./splunk enable boot-start -user splunk, which enables Splunk to start with the 'splunk' user upon server reboot, preventing any permission issues. I then checked the status of the Splunk service by running sudo systemctl status splunk to confirm that the service was active.

![0a56b967f32fc4234ce0e908678d428f](https://github.com/user-attachments/assets/854c7420-ab2e-4903-ae1d-9ef048ca519b)
![45c88f4add6aa17cba8c660bb394a87b](https://github.com/user-attachments/assets/a37519be-7c53-4e68-b576-380002afb600)



# Windows 10 Configuration

On my Windows 10 server (target server), I set a static IP address of 192.168.10.100. After that, I installed Splunk Universal Forwarder and Sysmon64. I then edited the inputs.conf file to create an index named "endpoint," so Splunk knows where to send the data. I also created a custom index in Splunk and configured all my traffic to route to the "endpoint" index. Lastly, I used PowerShell to download Atomic Red Team to test the system's defenses.

![bcfaadf413656ddb558afe7955a571ec](https://github.com/user-attachments/assets/12fb6d1f-dcd7-4b75-879f-66ea85eb067e)
![3716d3d0ebc40877a4b8fa1d53b7e10b](https://github.com/user-attachments/assets/6c44eb4a-3a69-4a8d-80b8-4acfd1a2498d)
![207a77159afeeea9e51e891847d9d8f7](https://github.com/user-attachments/assets/0e8bd271-41eb-4686-8da6-ebb3d8b6074d)





# Kali Linux Configuration

According to Verizon, 89% of web application hacking attempts involve brute force attacks or credential abuse. The 2024 Data Breach Investigations Report also indicates that brute force attacks make up 21% of all basic web application attacks.

On my Kali machine (the attacker machine), I set the IP address to 192.168.10.250 and installed Crowbar, a well-known brute force attack tool. Afterward, I unzipped the rockyou.txt.gz file, which provides a text document containing multiple passwords for use in the attack. I then ran the command crowbar -b rdp -u tpond -C passwords.txt 192.168.10.100/32 to instruct Crowbar to launch a brute force attack, specifying RDP (Remote Desktop Protocol) for the user 'tpond' using the passwords.txt file. This command targeted my Windows 10 machine and resulted in a successful login.

The use of the Crowbar brute force tool in this demonstration is strictly for educational and authorized testing purposes only. It is intended to demonstrate security vulnerabilities and highlight the importance of securing systems. Unauthorized access to systems or accounts, as well as using Crowbar to target any network, server, or device without explicit permission, is illegal and unethical.

![ec52880bd54476893377e156ec0fe085](https://github.com/user-attachments/assets/30f65528-8462-4184-ae7e-18e51e50669d)
![318d25b2f86d89d8825e90135b20eb8d](https://github.com/user-attachments/assets/9aa43855-6839-4d06-8475-a9a78687919e)
![a376e330f02352567ebb0634c10f99d0](https://github.com/user-attachments/assets/0ac30a59-51ba-4ab0-9ce1-ae090e6aec43)
![9824a83103461e1cc00a6a5ca5e53875](https://github.com/user-attachments/assets/b5a00c1f-400f-43a3-b16e-d30438e88699)




# Final Results


After launching a brute force attack on my Windows 10 machine, I checked Splunk for activity on the IP address (192.198.10.10:8000). My Splunk index, named "endpoint," on port 9997, allows me to track attack information by using the query "index=endpoint." Upon reviewing the network data, I observed suspicious activity originating from my Kali machine, which I had assigned the IP address (192.168.10.250).

Digging deeper into the logs, I noticed two standout Windows Security Log event IDs: 4625 and 4624. Event ID 4625 indicates a failed login attempt, while 4624 represents a successful login. This suggests that the failed login attempts were the result of incorrect passwords, and the successful login likely occurred after a brute force attack succeeded. A spike in failed login events is always a critical indicator worth further investigation.

Depending on the work environment, there are several approaches you can take to address a successful brute force attack. One of the first steps is to disconnect the compromised machine from the network to prevent malware from spreading and to stop the attacker from gaining further access. Next, review system logs to trace the attacker's actions after the successful login.  It’s essential to lock down the compromised account, force a password reset, and immediately implement Multi-Factor Authentication (MFA) for added security. Conduct a thorough malware scan on the affected system, apply security patches, and update software. If necessary, restore the system from a clean backup to ensure integrity and stability.

In the Atomic folder within Atomic Red Team, you can find specific test IDs used to evaluate your system's defenses against particular threats. These tests can be paired with the MITRE ATT&CK framework to gain deeper insights into how attackers operate and how to strengthen your defenses. By aligning Atomic Red Team tests with MITRE ATT&CK techniques, you can better understand vulnerabilities and improve your system's overall security posture.

![3885d1bfd656fcaaa5ea3a39b006188d](https://github.com/user-attachments/assets/12a2c7a9-4f2e-49a7-a7c7-88740e1a53eb)
![1f46b739bfc3a32fd011a91618a45838](https://github.com/user-attachments/assets/4b90c5c3-412c-43dd-99b3-1171a8f6f8b5)
![98b7ad089da2b37145372e1b69de645e](https://github.com/user-attachments/assets/1d6e9e40-1dfa-40b3-911f-017a87db8bef)
![ea4c923e98be6e458da0b278a4422392](https://github.com/user-attachments/assets/d927ef14-4193-4bad-9699-e93a75fb9ef9)
![0061c9dd2df7646c390e4a3ba5704ef1](https://github.com/user-attachments/assets/1f1588fe-c747-4e0e-b5f0-5ee5188d6f33)
![e55ed26e348ebd33773e03ca8bfbeee5](https://github.com/user-attachments/assets/1af71176-b84b-4ece-aef7-1038dbd8f4dd)
![cb61a999dce4c2cc284002cdd49f799a](https://github.com/user-attachments/assets/b81b26cd-84da-41a7-bfe5-21704fa2c12a)


























```
Thank you for taking the time to review my project.
```
