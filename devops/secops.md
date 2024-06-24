

### What are major SIEM capabilities?

<details>

1. Log collection, Log analysis and Log correlation
2. IT Compliance, Application monitoring
3. Real-time alerting, Dashboard and reporting
4. Ingest a near-real-time feed of telemetry describing activity in the environment
5. Structure information, store it and generate statistics for baseline behaviour levels
6. Surface alerts (incidents) for when a recent stream of data either deviates from the standard baseline, or matches a certain criteria that an enterprise wants to detect.

</details>

### Could you describe an example of a situation where Splunk would perform alerting / incidents?

<details>
Certainly! Splunk is a powerful SIEM tool that excels in real-time monitoring, alerting, and incident management. Here's an example scenario to illustrate how Splunk would perform alerting and handle incidents:

#### Scenario: Financial Services Company

##### 1. **Initial Setup**:
- **Data Sources**: The company configures Splunk to collect logs from various sources, including firewalls, intrusion detection systems (IDS), web servers, application servers, and databases.
- **Use Cases**: The company defines several use cases for monitoring, such as detecting unauthorized access, monitoring sensitive data access, and identifying suspicious network activity.

##### 2. **Configuration of Alerts**:
- **Unauthorized Access**: An alert is set up to trigger when there are multiple failed login attempts followed by a successful login from the same IP address within a short time frame.
- **Sensitive Data Access**: An alert is configured to monitor and trigger when there are unusual access patterns to sensitive financial data, such as after-hours access or access from an unusual location.
- **Suspicious Network Activity**: An alert is configured to detect high volumes of traffic from a single IP address or network segment, indicating a potential Distributed Denial of Service (DDoS) attack.

#### Example Incidents:

##### Incident 1: Brute Force Attack

**Detection**:
- Splunk collects logs from the company's authentication system.
- An alert is triggered when the system detects multiple failed login attempts followed by a successful login.

**Alert**:
- Alert Name: "Potential Brute Force Attack"
- Alert Criteria: More than 5 failed login attempts followed by a successful login from the same IP within 10 minutes.

**Response**:
- The incident response team receives the alert via email and the Splunk dashboard.
- The team uses Splunk to investigate the IP address, identifying the user account targeted and the source of the attempts.
- The user account is temporarily locked, and the IP address is blocked.
- A root cause analysis is conducted to understand how the attacker obtained the correct credentials, and additional security measures (e.g., multi-factor authentication) are implemented.

##### Incident 2: Unauthorized Data Access

**Detection**:
- Splunk monitors access logs for the company's financial database.
- An alert is triggered when sensitive financial data is accessed during non-business hours.

**Alert**:
- Alert Name: "Unusual Access to Financial Data"
- Alert Criteria: Access to financial data between 10 PM and 6 AM by non-administrative users.

**Response**:
- The incident response team receives the alert and reviews the access logs in Splunk.
- They identify the user account and the data accessed.
- The team contacts the user to verify if the access was legitimate. If not, they investigate further for potential insider threats or compromised accounts.
- Access controls and monitoring rules are updated to prevent future incidents.

##### Incident 3: DDoS Attack

**Detection**:
- Splunk collects network traffic data from firewalls and IDS.
- An alert is triggered when there is an unusually high volume of traffic from a single IP or a range of IPs.

**Alert**:
- Alert Name: "Possible DDoS Attack"
- Alert Criteria: Traffic volume exceeding a predefined threshold from a single IP or subnet over a short period.

**Response**:
- The incident response team receives the alert and immediately starts mitigating the attack by implementing rate limiting and blocking offending IP addresses using network devices.
- They use Splunk to analyze the attack pattern, identifying the attack vectors and potentially compromised systems.
- Post-incident, the team reviews and strengthens DDoS protection measures, such as improving rate limiting, deploying DDoS protection services, and updating incident response plans.

#### Benefits of Using Splunk for Incident Management:

- **Real-Time Monitoring**: Splunk provides real-time visibility into security events, enabling quick detection and response.
- **Centralized Log Management**: Aggregates logs from various sources, providing a comprehensive view of the security landscape.
- **Customizable Alerts**: Allows the creation of specific alerts tailored to the organization's security needs.
- **Advanced Analytics**: Uses machine learning and advanced analytics to identify patterns and anomalies.
- **Incident Investigation**: Facilitates detailed investigation through powerful search capabilities, enabling root cause analysis and remediation.

#### Conclusion:

Using Splunk for alerting and incident management helps organizations proactively detect and respond to security incidents. By setting up customized alerts and leveraging Splunk's robust analytics, companies can effectively monitor their environment, quickly address potential threats, and enhance their overall security posture.

</details>