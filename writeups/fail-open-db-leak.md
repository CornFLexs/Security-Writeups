### Technical Write-up: From Phishing to C2 Takeover

Author: Dikshansh Agrawal

Target Category: Malware Analysis & C2 Infrastructure Penetration

### 1. Executive Summary
This report details the end-to-end analysis of a phishing-based malware campaign. The attack chain began with a weaponized PDF masquerading as an Adobe update, which facilitated the delivery of a secondary executable. This executable established persistence and exfiltrated local data via a VBScript pulled from a public GitHub repository.

Upon identifying the attacker’s backend infrastructure—a Python-based "Data Receiver" server—vulnerability research was conducted on the web interface. A misconfigured OpenAPI/Swagger endpoint was discovered, leading to an Arbitrary File Upload vulnerability. By exploiting this, I gained a reverse shell on the attacker's server to assess the extent of the data breach and the impact on the organization.
