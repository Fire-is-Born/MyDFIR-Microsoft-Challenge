# MyDFIR-Microsoft-Challenge

## Azure Environment Setup

Created a Resource Group called **MyDFIR-Luke-RG** to keep all of the project resources organised in one place.

Deployed a **Windows 11 virtual machine** named **MyDFIR-Luke-VM** into the resource group and enabled **Remote Desktop (RDP)** to allow remote access. This VM will be used throughout the project to generate Windows events and simulate security scenarios.

Created a **Log Analytics Workspace** named **MyDFIR-Luke-LAW** to collect and store logs from the virtual machine and other connected Microsoft services.

Enabled **Microsoft Sentinel** on the Log Analytics Workspace to provide SIEM capabilities for monitoring, threat hunting, alerting, and incident investigation.

Finally, confirmed that Sentinel had been successfully enabled by viewing the workspace in **Microsoft Defender**.

<img width="2536" height="1256" alt="image" src="https://github.com/user-attachments/assets/5c54bf15-0bd2-4929-baa0-ce0221f6ee13" />


Connected the **MyDFIR-Luke-LAW** Log Analytics Workspace to **Microsoft Defender XDR** and configured it as the primary Microsoft Sentinel workspace.

Using the **Content hub**, installed the **Azure Activity** solution. Azure Activity Logs capture subscription-level events, including resource creation and deletion, configuration changes, administrative actions, service health events, and other operations performed through Azure Resource Manager. These logs provide visibility into changes made across the Azure environment and can be used to detect suspicious or unauthorised administrative activity.

With the Azure Activity solution installed, the environment is now ready for the **Microsoft Sentinel Training Lab** configuration.

### Configure Azure Activity Logs

Configured **Azure Activity Logs** to be sent to the **Log Analytics Workspace** via **Subscription → Activity Log → Diagnostic settings**. Enabled the **Administrative**, **Security**, and **Alert** log categories, allowing Microsoft Sentinel to ingest subscription-level events for monitoring and investigation.

### Microsoft Sentinel Training Lab Setup

The **Microsoft Sentinel Training Lab** is a Microsoft-provided lab that loads pre-recorded security data into Sentinel and provides exercises for investigating threats, creating detections and working with security incidents.

I deployed the lab using the provided Azure template but ran into an issue where a previous deployment had created an **automation rule**. Even after deleting the deployment, Azure continued to detect the rule as a duplicate, causing new deployments to fail.

After trying to clean up the existing resources and redeploy, I eventually switched to a fresh Azure account and deployed the lab in East US, where the deployment completed successfully.

<img width="2444" height="813" alt="image" src="https://github.com/user-attachments/assets/afe515cd-dcae-4f4d-8093-78e03f9e3553" />

Used this query to return a small sample of records from the sign-in logs. This helped me understand what data and fields were available before writing a more targeted query.

<img width="1894" height="698" alt="image" src="https://github.com/user-attachments/assets/67a1ad43-7352-4214-a2c3-2f162abe9569" />

Used this query to identify users with failed sign-in attempts and count how many failures were associated with each account. The results are then sorted to make accounts with the highest number of failed attempts easier to spot.

<img width="1923" height="537" alt="image" src="https://github.com/user-attachments/assets/b4ff4adf-b010-45b7-8cf7-25b275eb865f" />

Wrote this KQL query to identify sign-in attempts involving disabled user accounts using result code 50057. I then used project to display only the relevant fields, making the results easier to review during an investigation.
<img width="1787" height="543" alt="image" src="https://github.com/user-attachments/assets/c7101d13-0dac-46bb-860a-78a647326f0f" />

## Creating a Failed Login Visualisation

<img width="2148" height="1073" alt="image" src="https://github.com/user-attachments/assets/953e4665-5348-4228-8212-978a45a09576" />

Created a visualisation in the Microsoft Sentinel workbook to identify which accounts had the highest number of failed login attempts.

Windows **Event ID 4625** represents a failed account logon. I used the following KQL query:

```kusto
SecurityEvent
| where EventID == 4625
| summarize Count = count() by Account
| sort by Count
| take 10
```

The query searches the `SecurityEvent` table for Event ID 4625, groups the results by account and counts the number of failed login attempts associated with each one. It then returns the top 10 accounts.

I displayed the results as a **pie chart** and removed the automatically generated **"Others"** group so the visualisation remains focused on the accounts with the most failed login activity.

## Dashboard Design Principles

Going forward, I will keep three main principles in mind when deciding what information to include in the SOC dashboard:

**Visibility** – Can I see what is happening across the environment? This could include endpoints, users and identities, email, cloud resources and network activity.

**Volume** – Can I see how much activity is occurring? This includes things such as failed logins, security alerts, incidents and authentication attempts.

**Trends** – Can I identify when activity is increasing or changing over time? For example, an increase in failed logins, a sudden spike in alerts or unusual activity associated with a particular account.

The aim is to create a dashboard that helps an analyst quickly understand what is happening and identify activity that may require further investigation, rather than simply displaying as much data as possible.
