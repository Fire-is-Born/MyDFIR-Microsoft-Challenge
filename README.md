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

<img width="2125" height="813" alt="image" src="https://github.com/user-attachments/assets/d0a098a6-f60a-4145-8145-37409ef21034" />



I created a custom Microsoft Sentinel workbook to practise using KQL and learn how security data can be presented in a way that is useful to a SOC analyst.

When building the dashboard, I kept three main ideas in mind:

**Visibility** – Can I see what is happening across different parts of the environment, such as users, endpoints and email?

**Volume** – Can I see how much activity is happening, such as failed logins or security alerts?

**Trends** – Can I spot increases or changes in activity that could be worth investigating?

For this dashboard I decided to start with authentication activity, security alerts and email threats.

### Top 10 Accounts by Failed Login Attempts

I started by looking at failed Windows logins. **Event ID 4625** is generated when an account fails to log on, so I used it to find the accounts with the most failed attempts.

```kusto
SecurityEvent
| where EventID == 4625
| extend Account = toupper(Account)
| where isnotempty(Account) and Account != "\\"
| summarize Count = count() by Account
| top 10 by Count desc
```

While looking at the results, I noticed the same account could appear with different capitalisation, such as `\administrator` and `\ADMINISTRATOR`. I used `toupper()` to make these consistent before counting them and also removed an invalid account entry.

I displayed the top 10 results as a **donut chart** to quickly show which accounts had the most failed login activity.

### Security Alerts by Severity

Next I wanted a simple overview of the alerts that had actually been generated and how serious they were.

```kusto
SecurityAlert
| summarize AlertCount = count() by AlertSeverity
| order by AlertCount desc
```

This groups the alerts by severity and counts each group. The dataset contained **8 High, 2 Medium and 1 Low severity alerts**.

I used a **bar chart** for this because it makes it easy to compare the number of alerts at each severity and quickly see that High severity alerts make up most of the results.

### Actions Taken on Detected Phishing Emails

I also wanted the dashboard to cover something other than Windows and authentication events, so I looked at the email security data in `MailGuard365_Threats_CL`.

The dataset used both `Phish` and `Phishing` as threat verdicts, so I included both when filtering the results.

```kusto
MailGuard365_Threats_CL
| where ThreatVerdict in ("Phish", "Phishing")
| summarize EmailCount = count() by Action
| order by EmailCount desc
```

I grouped the detected phishing emails by the action that was taken against them. This returned **11 phishing emails: 10 were allowed and 1 was quarantined**.

I displayed this as a **donut chart** because there are only two outcomes and I wanted to make the difference between them easy to see. The fact that most of the detected phishing emails were allowed also gives me something interesting to investigate further.
