# February 7, 2026

## Wazuh

It is time to implement a server security monitoring tool. For this purpose, I chose Wazuh - a free, open-source platform designed for threat detection, event correlation, and centralized log aggregation.
The Wazuh implementation phase proved to be one of the most challenging in the entire project. This was due to the high level of architectural complexity, the need for correct component configuration, and issues encountered during the integration of the DMZ with the internal network.
<br></br>
Deployment Architecture:
<strong>DMZ VPS:</strong> Wazuh Agent
<strong>Internal VPS:</strong> Wazuh Manager, Wazuh Indexer, Wazuh Dashboard
<br></br>
During the deployment, I encountered issues with agent visibility on the manager side and dashboard instability. To resolve these issues:
- I added a UFW rule allowing communication on port 1514/tcp (attempts to configure UDP and modify configuration files failed in version 4.14.2-1)
- I updated the Wazuh service version on the Internal VPS from 4.7.5 to 4.14.2-1, as the agent on the DMZ VPS was running 4.14.2-1, which caused an incompatibility
- I configured the agent accordingly
- I relocated the certificates to the appropriate directories and granted them the correct permissions

Wazuh is currently actively monitoring:
- system logs
- SSH login attempts
- UFW firewall rule activity
- Fail2Ban actions and active bans

More screenshots will be added in the next changelog update.

<img width="1919" height="1031" alt="wazuh_dashboard" src="https://github.com/user-attachments/assets/a0b96ed7-70c5-49d8-8673-11cd1e66253a" />


# February 6, 2026

## Hytale Server Update (DMZ)

To ensure the Hytale server runs independently of my active SSH session, I decided to use the tmux tool, which allows me to manage terminal sessions and maintain background processes even after SSH disconnection.

Additionally, similar to the VPN configuration, I set up an autostart mechanism for the Hytale server upon system reboot. For this purpose, I created a dedicated system user, "hytale," restricted solely to executing the game server's startup script. SSH access for this user has been completely disabled.

The execution flow is as follows:
- a systemd service initializes a tmux session
- tmux creates a new session where the start.sh script is executed under the "hytale" user context
- the script is located in the /opt/hytale/server directory

During configuration, I encountered minor complications regarding:
- incorrect permissions for the "hytale" user
- over-complicated session-hardening measures which intended to increase security but, in practice, limited the proper operation of the server

<img width="396" height="33" alt="image" src="https://github.com/user-attachments/assets/4f763a08-2232-40bf-8bee-bdb31033f413" />
<br></br>
<img width="1204" height="256" alt="hytale_systemctl" src="https://github.com/user-attachments/assets/1c3c38a3-a6b3-446c-8cce-bb74b986a9d2" />
<br></br>
Video (low quality due to GitHub’s file size limit): https://github.com/user-attachments/assets/672a2064-f9c9-4261-beac-1feb1d0a7782
<br></br>
<img width="941" height="648" alt="hytale_tmux" src="https://github.com/user-attachments/assets/928a24eb-4ea6-4658-b7e0-c78454dd91a3" />


# February 5, 2026

## VPN S2S

As of today, I have successfully configured WireGuard for a site-to-site VPN connection. I chose this solution for its simplicity and the inherent security benefits it provides, such as a simple configuration and modern cryptography. Furthermore, I have enabled persistence and autostart for the service on both VPS instances. This ensures that communication remains uninterrupted after system reboots and provides a stable foundation for implementing granular access control based on the principle of least privilege.

<img width="715" height="344" alt="dmz_systemctl" src="https://github.com/user-attachments/assets/26990fd3-2012-44f4-98e3-e6354babc08d" />
<br></br>
<img width="673" height="340" alt="internal_systemctl" src="https://github.com/user-attachments/assets/5d3001f9-0e40-4d3c-8624-12604ba60582" />
<br></br>
<img width="1430" height="596" alt="wireguard" src="https://github.com/user-attachments/assets/cf06bef8-3ef2-47e5-bd67-b0209527382f" />

## Change details:
- configured site-to-site VPN
- added a new rule to the internal VPS firewall (listening on vpn port)
<br></br>


# Original Project State
DISCLAIMER: I consider the original project state to be the one that was current at the time the "secure-vps-dmz-internal-lab" GitHub repository was created.

## DMZ

In the initial state of the VPS DMZ, two ports are open:
- 22/tcp secure shell
- 5520/udp Hytale game server

I deployed a Hytale game server on the server after downloading Java 25 (Adoptium) and authenticating on the game website (the game server will also be part of the project).

Security:
- UFW - the firewall blocks incoming traffic by default and allows outgoing traffic and ports 22 and 5520
- Fail2Ban - due to the temporary security of SSH connections to the VPSs using passwords, despite their complexity, a program was needed to enforce rules designed to limit login attempts by blocking access to the secure shell for addresses that exceed a specified number of failed attempts within a given period



## Internal

In the initial state of the VPS, only one port is available, 22/tcp for secure shell

Security:
- UFW - the firewall blocks incoming traffic by default, but allows outgoing traffic and port 22
- Fail2Ban - due to the temporary security of SSH connections to the VPSs using passwords, despite their complexity, a program was needed to enforce rules designed to limit login attempts by blocking access to the secure shell for addresses that exceed a specified number of failed attempts within a given period
