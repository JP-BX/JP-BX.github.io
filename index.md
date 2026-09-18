**Active Directory • Splunk • Sysmon • Kali Linux • Atomic Red Team**

## Project Overview

Built an Active Directory security lab to simulate credential-based attacks, collect Windows security telemetry, and investigate malicious authentication activity using Splunk and Sysmon.

**Environment:** Windows Server • Windows 10 • Kali Linux • Splunk Enterprise • Sysmon • Splunk Universal Forwarder • Atomic Red Team • Oracle VirtualBox

**Security Skills:** Active Directory • SIEM Monitoring • Log Analysis • Threat Detection • Windows Event Logs • Attack Simulation • Incident Investigation







# Lab Architecture

![AD Project drawio](https://github.com/user-attachments/assets/7f01f634-2d8e-44f5-a0c4-7692bd9f230f)

This diagram illustrates the lab setup and the data flow between each system. 

![862bf9d882d38eda731787348bf9deb7](https://github.com/user-attachments/assets/626561bd-6e50-4059-9897-921c8c2a1360)

I used Oracle VM to host all the servers and connected them on a NAT network

## Active Directory Configuration

Configured a Windows Server domain controller to provide centralized identity and authentication services for the lab environment.

- Assigned the domain controller a static IP address of `192.168.10.7`
- Installed **Active Directory Domain Services (AD DS)**
- Created the `JSP.local` Active Directory domain
- Created test user accounts and security groups to simulate an enterprise environment
- Installed **Sysmon** to capture detailed Windows security telemetry
- Configured the **Splunk Universal Forwarder** to send security events to the Splunk SIEM server
- Verified network connectivity between the domain controller and Splunk server

![fb0419984e90344f1ad6a5fd6019b181](https://github.com/user-attachments/assets/25d5e680-4fe7-4300-9583-97110a126a1e)
![d53fc56ed78a76a346b76140323e06f8](https://github.com/user-attachments/assets/bb307381-4cfc-4575-b677-ce52bb69015d)
![1b2761e5a42d6e81865eac4a337dc1a3](https://github.com/user-attachments/assets/2f47482c-5d88-4ad1-bddb-ab4c5bff2c76)![758a02ea5baacbab939203bdfa6704b1](https://github.com/user-attachments/assets/919af5f9-610d-418b-a217-f7c72e5c0179)














## Splunk Server Configuration

Configured an Ubuntu Server to host **Splunk Enterprise** and serve as the centralized SIEM platform for the lab.

- Assigned the Splunk server a static IP address of `192.168.10.10/24`
- Configured the network interface using **Netplan**
- Installed and configured **Splunk Enterprise**
- Enabled Splunk to start automatically at system boot
- Verified the Splunk service was running successfully using `systemctl`
- Configured the server to receive security telemetry from endpoints running the Splunk Universal Forwarder

![0a56b967f32fc4234ce0e908678d428f](https://github.com/user-attachments/assets/854c7420-ab2e-4903-ae1d-9ef048ca519b)
![45c88f4add6aa17cba8c660bb394a87b](https://github.com/user-attachments/assets/a37519be-7c53-4e68-b576-380002afb600)



## Windows 10 Target Endpoint

Configured a Windows 10 system as the primary target endpoint for attack simulation and security monitoring.

- Assigned the endpoint a static IP address of `192.168.10.100`
- Configured the domain controller (`192.168.10.7`) as the DNS server
- Installed **Sysmon** to capture detailed endpoint security telemetry
- Installed and configured the **Splunk Universal Forwarder**
- Configured `inputs.conf` to forward Windows and Sysmon events to the `endpoint` index in Splunk
- Created a dedicated `endpoint` index in Splunk for centralized log analysis
- Installed **Atomic Red Team** using PowerShell to generate controlled security events for detection testing

![bcfaadf413656ddb558afe7955a571ec](https://github.com/user-attachments/assets/12fb6d1f-dcd7-4b75-879f-66ea85eb067e)
![3716d3d0ebc40877a4b8fa1d53b7e10b](https://github.com/user-attachments/assets/6c44eb4a-3a69-4a8d-80b8-4acfd1a2498d)
![207a77159afeeea9e51e891847d9d8f7](https://github.com/user-attachments/assets/0e8bd271-41eb-4686-8da6-ebb3d8b6074d)





## Kali Linux Attack Simulation

Configured a Kali Linux system to act as the attacker machine and generate controlled authentication activity against the Windows 10 target endpoint.

- Assigned the Kali Linux system the IP address `192.168.10.250`
- Verified network connectivity to the Splunk server and Windows target
- Installed **Crowbar** for credential-based attack simulation
- Used the `rockyou.txt` wordlist as the password source for testing
- Simulated an **RDP brute-force attack** against the Windows 10 endpoint (`192.168.10.100`)
- Generated authentication events for analysis and detection in Splunk
- Successfully authenticated to the test account during the controlled attack simulation

### Brute-Force Simulation

The following Crowbar command was used to simulate repeated RDP authentication attempts against the Windows 10 target:

`crowbar -b rdp -u tpond -C passwords.txt 192.168.10.100/32`

The attack generated Windows authentication telemetry that was forwarded to Splunk for investigation.

> **Lab Safety:** All attack simulations were performed against systems within my isolated home lab for authorized educational and security testing purposes.
![ec52880bd54476893377e156ec0fe085](https://github.com/user-attachments/assets/30f65528-8462-4184-ae7e-18e51e50669d)
![318d25b2f86d89d8825e90135b20eb8d](https://github.com/user-attachments/assets/9aa43855-6839-4d06-8475-a9a78687919e)
![a376e330f02352567ebb0634c10f99d0](https://github.com/user-attachments/assets/0ac30a59-51ba-4ab0-9ce1-ae090e6aec43)
![9824a83103461e1cc00a6a5ca5e53875](https://github.com/user-attachments/assets/b5a00c1f-400f-43a3-b16e-d30438e88699)




## Detection & Investigation Results

After executing the controlled RDP brute-force attack, I used **Splunk Enterprise** to investigate the authentication activity generated on the Windows 10 target endpoint.

### Splunk Investigation

I searched the dedicated endpoint index using:

`index=endpoint`

Analysis of the Windows Security logs revealed repeated authentication attempts originating from the Kali Linux attacker system (`192.168.10.250`).

### Key Windows Security Events

| Event ID | Description | Observation |
|----------|-------------|-------------|
| **4625** | Failed Logon | Multiple failed authentication attempts were recorded during the brute-force simulation |
| **4624** | Successful Logon | A successful authentication event was recorded following the failed attempts |

The combination of repeated **Event ID 4625** events followed by **Event ID 4624** demonstrated the authentication pattern generated during the controlled brute-force attack.

The Splunk logs also identified the source IP address `192.168.10.250`, allowing the activity to be correlated back to the Kali Linux attacker system.

### Incident Response

In a production environment, investigation and response to similar activity could include:

- Identifying the affected user account and source IP address
- Reviewing surrounding authentication events for additional suspicious activity
- Temporarily disabling or securing a compromised account when appropriate
- Resetting exposed credentials
- Enforcing **Multi-Factor Authentication (MFA)**
- Isolating an affected endpoint if compromise is suspected
- Reviewing endpoint telemetry for signs of post-authentication activity
- Blocking confirmed malicious sources where appropriate
- Continuing monitoring for additional authentication attempts

### MITRE ATT&CK Mapping

The simulated credential attack demonstrates behavior associated with:

**T1110 – Brute Force**

MITRE ATT&CK mapping provides a standardized way to associate the observed authentication activity with known adversary techniques.
![3885d1bfd656fcaaa5ea3a39b006188d](https://github.com/user-attachments/assets/12a2c7a9-4f2e-49a7-a7c7-88740e1a53eb)
![1f46b739bfc3a32fd011a91618a45838](https://github.com/user-attachments/assets/4b90c5c3-412c-43dd-99b3-1171a8f6f8b5)
![98b7ad089da2b37145372e1b69de645e](https://github.com/user-attachments/assets/1d6e9e40-1dfa-40b3-911f-017a87db8bef)
![ea4c923e98be6e458da0b278a4422392](https://github.com/user-attachments/assets/d927ef14-4193-4bad-9699-e93a75fb9ef9)
![0061c9dd2df7646c390e4a3ba5704ef1](https://github.com/user-attachments/assets/1f1588fe-c747-4e0e-b5f0-5ee5188d6f33)
![e55ed26e348ebd33773e03ca8bfbeee5](https://github.com/user-attachments/assets/1af71176-b84b-4ece-aef7-1038dbd8f4dd)
![cb61a999dce4c2cc284002cdd49f799a](https://github.com/user-attachments/assets/b81b26cd-84da-41a7-bfe5-21704fa2c12a)

## Project Takeaways

This project provided hands-on experience building and monitoring an Active Directory environment while simulating and investigating credential-based attacks.

Through the lab, I gained practical experience with **Active Directory, Splunk Enterprise, Sysmon, Windows Security Events, Kali Linux, attack simulation, SIEM investigation, and MITRE ATT&CK mapping**.

The project demonstrated the complete security monitoring workflow from **attack simulation and telemetry collection to detection, investigation, and incident response analysis**.

























