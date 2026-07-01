<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Detect a Brute-Force Attack with Wazuh

**Project Link:** [View Project](https://nextwork.ai/projects/39700e72-4559-40e2-b28e-5d560a52bafb)

**Author:** suhaibfaiyaz  


---

![Image](https://nextwork.ai/grateful_silver_mysterious_dill/uploads/39700e72-4559-40e2-b28e-5d560a52bafb_c2m0m0pr)

## Project Overview: Building a SOC Detection Pipeline

### Why this project matters for cybersecurity analysts

Security analysts rely on SIEM platforms to monitor environments, detect threats, and investigate security incidents. 

This project provides practical experience with those responsibilities by building a Wazuh-based SOC lab that collects logs, generates alerts, and helps identify suspicious activity such as brute-force login attempts.

## Deploying a Live Wazuh SIEM Stack with Docker

### Launching the manager, indexer, and dashboard

Wazuh consists of several components that work together to collect, process, store, and visualize security events. 

Instead of installing each component manually, Wazuh provides a pre-configured Docker deployment that starts all required services automatically.

In this,
- Download the official Wazuh Docker deployment files from GitHub.

[ git clone https://github.com/wazuh/wazuh-docker.git -bv4.14.5 ]

[ cd wazuh-docker/single-node/ ]

- Generate the certificates required for secure communication between services.

[ docker compose -f generate-indexer-certs.yml run --rm generator ]

- Start the Wazuh containers using Docker Compose
Access the Wazuh web dashboard.
- Verify that the SIEM platform is running successfully

![Image](https://nextwork.ai/grateful_silver_mysterious_dill/uploads/39700e72-4559-40e2-b28e-5d560a52bafb_22hfb6x8)

### Confirming all three containers are running

"docker compose ps"
The command showed that all three core Wazuh services were running successfully..

CONTAINER ID                    IMAGE                           STATUS
9bfd3c4a3bc2     wazuh/wazuh-manager:4.14.5          Up
ea2e89129ae7     wazuh/wazuh-indexer:4.14.5             Up
0f7b9f015be1       wazuh/wazuh-dashboard:4.14.5       Up

The Wazuh environment was deployed successfully and ready to receive security events from monitored endpoints.

## Connecting a Windows Endpoint to the SIEM

### Installing the Wazuh agent on the Windows host

The Wazuh agent acts as a bridge between the Windows host and the Wazuh manager. 

It continuously gathers information such as login attempts, system activity, and security events, then forwards that data to the manager for analysis and alert generation.

Once connected, the Windows machine becomes a data source for the SIEM. Security events generated on the system are collected by the agent and sent to the Wazuh manager, allowing suspicious activity to be detected and investigated in real time.

![Image](https://nextwork.ai/grateful_silver_mysterious_dill/uploads/39700e72-4559-40e2-b28e-5d560a52bafb_ym8attpe)

### The agent's role in real-time endpoint monitoring

The Wazuh agent is responsible for collecting security-related information from the Windows system.
- - - - -
>>It continuously monitors sources such as:

     - Windows Event Logs
     - Login and authentication events
     - System activity
     - File integrity changes
     - Installed software and system inventory

>>The collected information is forwarded to the Wazuh manager, where it is analyzed against detection rules and threat intelligence data.

>>When suspicious activity is identified, the manager generates alerts that can be investigated through the Wazuh dashboard.
- - - - -
This process enables real-time visibility into endpoint activity and helps security analysts detect and respond to potential security incidents.

## Simulating a Brute-Force Attack and Investigating Alerts

### Generating Windows Event ID 4625 failed logon events

In this step, I simulated a brute-force login attack by intentionally generating multiple failed login attempts on my Windows machine.

The goal was to create real Windows security events that could be collected by the Wazuh agent and analyzed by the SIEM. This allowed me to observe how failed authentication attempts are recorded, detected, and investigated within the Wazuh dashboard.

After generating the failed logon events, I reviewed the resulting alerts in Wazuh and examined the event details to understand how the SIEM identifies suspicious login activity.

![Image](https://nextwork.ai/grateful_silver_mysterious_dill/uploads/39700e72-4559-40e2-b28e-5d560a52bafb_likvy1iw)

### Examining alert details: username, source IP, and rule level

After generating the failed login attempts, I investigated the alerts in the Wazuh dashboard to understand what information had been captured.

The alert showed that the target username was fakeuser, which was the non-existent account used during the simulation. The source IP address was 127.0.0.1 (localhost), indicating that the login attempts originated from my own machine.

The alert was assigned a Rule Level 5, reflecting the severity of the detected event. Wazuh also identified the event as Windows Event ID 4625, which represents a failed logon attempt.

By examining these details, I was able to identify who attempted to log in, where the request originated, and why the authentication failed.

## Authoring a Custom Detection Rule Mapped to MITRE ATT&CK

### Writing Rule ID 100002 with correlation logic

In this step, I created a custom Wazuh detection rule to identify repeated failed login attempts that may indicate a brute-force attack.

Instead of alerting on every individual failed login, the rule looks for a pattern of multiple failed authentication attempts occurring within a short period of time. When this condition is met, Wazuh generates a higher-severity alert, making it easier to distinguish suspicious activity from normal user mistakes.

The custom rule was also mapped to MITRE ATT&CK Technique T1110 (Brute Force) to classify the detected behavior using an industry-recognized attack framework.

### Frequency, timeframe settings, and T1110 mapping explained

frequency="5": This is the threshold count. It tells the Wazuh manager that the parent logon-failure rule (Rule 60122) must trigger at least 5 times before escalating the event.
timeframe="60": This is the observation window. It defines a rolling time window of 60 seconds.
In combination: They perform event correlation. A single failed login is a normal human mistake, but 5 or more failures clustered tightly within 60 seconds represents a clear pattern of an automated attack.

Why it is mapped to MITRE ATT&CK T1110:

Industry Standardization: MITRE ATT&CK is a globally recognized database of real-world adversary tactics and techniques. T1110 is the standard industry classification code for Brute Force.

Analyst Context: Adding this tag allows security analysts viewing the dashboard to instantly recognize the threat category and tactic (Credential Access). It ensures consistent labeling during incident triage and security auditing.

## Automating Threat Response with Active Response

### Configuring firewall-drop to block attacker IPs automatically

What I configured:-
I configured an <active-response> block in the Wazuh manager's ossec.conf file.
- Command: <command>firewall-drop</command> to call the built-in firewall block script.
- Location: <location>local</location> to run the block script directly on the endpoint agent where the attack is detected.
- Rules ID: <rules_id>100002</rules_id> to link the response directly to my custom Level 10 brute-force detection rule.
- Timeout: <timeout>300</timeout> to automatically lift the firewall block after 5 minutes.

What evidence confirms it executed:-
- Configuration Verification: I verified that the configuration was successfully written inside the manager container's filesystem by running a grep search on /var/ossec/etc/ossec.conf, which successfully returned the firewall-drop blocks.
- Manager Health: Running docker ps confirmed the container successfully restarted and is running cleanly, meaning there were no XML syntax errors in my edits.
- Safety Whitelist Logic:

## Setting Up the Environment: Docker and Git

### Installing the tools needed to run the SIEM stack

Before we can deploy Wazuh, we need to install the tools required to run it.

Wazuh is distributed as a collection of Docker containers, which are lightweight packages containing all the software and dependencies needed for each Wazuh component.

Docker Desktop allows us to run and manage these containers on Windows.

We also need Git, which is used to download the official Wazuh deployment files from GitHub.

By the end of this step, we will have:

Docker Desktop installed and running
Git installed and accessible from the command line
A working Docker environment verified by running a test container

This prepares the system for deploying the Wazuh SIEM stack in the next step..

![Image](https://nextwork.ai/grateful_silver_mysterious_dill/uploads/39700e72-4559-40e2-b28e-5d560a52bafb_7a0d989s)

### Verifying Docker and Git installation

After installing Docker Desktop and Git, I verified that both tools were working correctly.
--------------------------------------------
C:\Users\suhai>docker --version
Docker version 29.5.3, build d1c06ef

C:\Users\suhai>docker compose version
Docker Compose version v5.1.4

C:\Users\suhai>git --version
git version 2.54.0.windows.1
--------------------------------------------
These results confirmed that Docker, Docker Compose, and Git were installed successfully and ready for use in deploying the Wazuh SIEM environment.

## Reflections and Key Takeaways

### Tools and concepts covered in this project

The key tools I used include
- Wazuh SIEM Stack: Spun up the Wazuh manager, indexer, and dashboard to analyze security events in real-time.
- Docker Desktop:Used to deploy, run, and manage the containerized SIEM services on my local machine.
- Wazuh Agent: Installed and configured the WazuhSvc service on my Windows host to actively collect system and security events.
-Windows Event Viewer: Inspected raw local security logs to verify that simulated failures were generating Event ID 4625 entries on the operating system.
- PowerShell: Utilized standard and elevated windows shells to navigate directories, resolve installer paths, and run an automated brute-force simulation script.

### Time to complete and biggest challenges

This project took me approximately 2 to 3 hours to complete.

The most challenging part was troubleshooting the actual mechanics of the Wazuh agent &manager communication. Instead of just blindly following a script, I had to act like a real security analyst to solve several actual hurdles:

Installer Path Quirks: Resolving the background installation failure by unblocking the MSI package and using the $PWD absolute path shortcut variable so msiexec could locate the file.

Rule ID Differences: Digging through the raw JSON logs to find out why rule 18106 wasn't firing, which led to me discovering that my environment triggers rule 60122 for Windows logon failures, & adjusting my custom rule logic to match.

No-Editor Environment: Overcoming the lack of text editors like vi inside the lightweight Wazuh manager container by implementing terminal redirection & Heredocs to safely write configuration parameters.


### Next steps and skills to explore

I did this project today to learn how to deploy a local SIEM stack using Wazuh, connect a real Windows agent, simulate a brute-force attack via automated PowerShell loops, and write custom event correlation rules to detect rapid password-guessing patterns. I also learned how to configure Active Response to dynamically trigger a firewall-drop containment action on endpoints and how system safety allowlists protect critical network nodes.

Another skill I want to learn is how to write automated playbooks using Ansible to deploy and manage security configurations, agents, & firewall rules consistently across multiple servers in an enterprise environment, or how to write custom detection rules for other attack vectors like privilege escalation and malware execution

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/39700e72-4559-40e2-b28e-5d560a52bafb)*
