# Azure Cloud Misconfiguration Hunt

## Project Overview

In this project, I performed a Cloud Security Posture Management (CSPM) assessment of a Microsoft Azure environment using Prowler. The goal was to identify cloud security misconfigurations, understand the risks associated with the findings, remediate selected high-severity issues, and validate the changes through a post-remediation security scan.

I deployed an Azure Storage Account in a dedicated lab resource group and configured Prowler in a Kali Linux virtual machine using Docker. Prowler authenticated to Azure using a Microsoft Entra ID service principal with Reader permissions.

The initial security assessment identified multiple configuration findings. I focused on two high-severity Azure Storage findings:

- Shared Key authentication was enabled.
- Public network access was enabled.

I remediated both findings through the Azure Portal and then performed a second Prowler scan to verify that the affected security checks changed from **FAIL** to **PASS**.

## Technologies Used

- Microsoft Azure
- Microsoft Entra ID
- Azure Storage
- Prowler
- Docker
- Kali Linux
- Azure CLI
- Cloud Security Posture Management (CSPM)
