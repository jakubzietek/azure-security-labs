\# Lab 06 — Network Segmentation and Lateral Movement Control with NSGs



\## Overview



This lab demonstrates \*\*network-level segmentation and lateral movement prevention\*\* within an Azure Virtual Network using \*\*Network Security Groups (NSGs)\*\* and explicit traffic direction control.



The objective is to prevent implicit trust inside a virtual network by enforcing:

\- Clear application and data tier separation

\- Explicit allowed communication paths

\- Explicitly denied reverse and lateral traffic

\- Validation using Azure-native diagnostic tooling



Validation is performed using \*\*Azure Network Watcher\*\*, avoiding inbound access, public IPs, or bastion hosts.



---



\## Goals



\- Enforce \*\*east–west traffic segmentation\*\* inside a VNet

\- Prevent \*\*lateral movement\*\* between workloads

\- Prevent \*\*reverse initiation\*\* from data tier to application tier

\- Apply \*\*explicit allow and deny rules\*\*

\- Validate behavior using \*\*Network Watcher Connection troubleshoot\*\*



---



\## Architecture Summary



\### Network Layout



| Subnet | Purpose |

|------|--------|

| `snet-app` | Application tier |

| `snet-data` | Data tier |



\### Workloads



| VM | Subnet | Role |

|---|------|----|

| `vm-app` | `snet-app` | Application workload |

| `vm-data` | `snet-data` | Data workload |



No virtual machines have public IP addresses.



---



\## Security Design



\### Segmentation Principles



\- Application tier \*\*may initiate\*\* connections to the data tier

\- Data tier \*\*must not initiate\*\* connections to the application tier

\- Application tier workloads \*\*must not communicate laterally\*\*

\- All traffic is \*\*explicitly evaluated\*\* by NSGs

\- Default allow behavior is not relied upon



---



\## Implementation



\### Network Security Groups



Two NSGs are used, each associated at the \*\*subnet level\*\*.



---



\### NSG — Application Subnet (`snet-app`)



Inbound rules enforce lateral movement prevention.



!\[NSG App Rules](screenshots/nsg-snet-app-rule-list.png)



Key controls:

\- Explicit deny for traffic originating from `snet-app` to `snet-app`

\- No implicit trust within the application tier



---



\### NSG — Data Subnet (`snet-data`)



Inbound and outbound rules enforce unidirectional access.



\#### Inbound Rules

!\[NSG Data Inbound Rules](screenshots/nsg-snet-data-inbound-rule-list.png)



\#### Outbound Rules

!\[NSG Data Outbound Rules](screenshots/nsg-snet-data-outbound-rule-list.png)



Key controls:

\- Allow inbound traffic from application subnet

\- Explicit \*\*outbound deny\*\* from data subnet to application subnet

\- Reverse initiation is blocked at the source



---



\## Validation



Validation was performed using \*\*Azure Network Watcher → Connection troubleshoot\*\*, avoiding interactive VM access.



---



\### Validation 1 — Application to Data (Allowed)



Test:

\- Source: `vm-app`

\- Destination: `vm-data`

\- Protocol: TCP

\- Port: 22



Result:

\- \*\*Reachable\*\*

\- Confirms intended application → data communication path



!\[App to Data Allowed](screenshots/success-connect-app-data.png)



---



\### Validation 2 — Data to Application (Denied)



Test:

\- Source: `vm-data`

\- Destination: `vm-app`

\- Protocol: TCP

\- Port: 22



Result:

\- \*\*Unreachable\*\*

\- Traffic blocked by outbound NSG rule on data subnet

\- Reverse initiation prevented



!\[Data to App Denied](screenshots/unreachable-data-app.png)



---



\### Validation 3 — Application Lateral Movement (Denied)



Test:

\- Source: `vm-app`

\- Destination: another IP in `snet-app`

\- Protocol: TCP

\- Port: 22



Result:

\- \*\*Unreachable\*\*

\- Lateral movement within application tier blocked



!\[App to App Denied](screenshots/unreachable-app-app.png)



---



\## Security Outcomes



\- No flat network trust

\- Explicit east–west traffic control

\- Lateral movement prevention enforced at network layer

\- Reverse initiation blocked at source

\- Validation performed without weakening security posture



---



\## Lessons Learned



\- Network Security Groups are \*\*directional and stateful\*\*

\- Blocking traffic at the \*\*source subnet\*\* is the most reliable segmentation strategy

\- Inbound-only controls are insufficient for strong isolation

\- Azure Network Watcher provides authoritative, non-intrusive validation

\- Proper segmentation significantly limits post-compromise movement



---



\## Notes



\- Validation was performed using Azure-native diagnostic tooling

\- No public IPs or inbound administrative access were required

\- Resources can be safely removed after validation to control cost



---



\## Next Steps



\- Centralized routing and segmentation (Hub-and-Spoke)

\- NSG + UDR interaction analysis

\- Infrastructure-as-Code implementation (Terraform)

\- Integration with firewall-based inspection



