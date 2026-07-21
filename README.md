# MyDFIR-Microsoft-Challenge

## Azure Environment Setup

Created a Resource Group called **MyDFIR-Luke-RG** to keep all of the project resources organised in one place.

Deployed a **Windows 11 virtual machine** named **MyDFIR-Luke-VM** into the resource group and enabled **Remote Desktop (RDP)** to allow remote access. This VM will be used throughout the project to generate Windows events and simulate security scenarios.

Created a **Log Analytics Workspace** named **MyDFIR-Luke-LAW** to collect and store logs from the virtual machine and other connected Microsoft services.

Enabled **Microsoft Sentinel** on the Log Analytics Workspace to provide SIEM capabilities for monitoring, threat hunting, alerting, and incident investigation.

Finally, confirmed that Sentinel had been successfully enabled by viewing the workspace in **Microsoft Defender**.

<img width="2536" height="1256" alt="image" src="https://github.com/user-attachments/assets/5c54bf15-0bd2-4929-baa0-ce0221f6ee13" />
