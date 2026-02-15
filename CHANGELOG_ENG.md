# February 15, 2026

## ChatGPT API (DMZ -> Internal)

The day came when a script was implemented on the Internal VPS enabling prompting to ChatGPT (model 4o) via VPN for the VPS's DMZ. The initial architecture assumed sending the prompt from the Hytale server, but unfortunately, at the current (early) stage of development of this game, and due to my skills, or rather lack thereof, I am unable to create a plugin/mod that would add a new command to the game server and operate in the manner I've described (even with the tools available), including ensuring the security I expect from the technologies built in this project.

(Disclaimer: Artificial Intelligence (ChatGPT) played a significant role in introducing the API to ChatGPT, which greatly helped me implement this solution due to the hurdles I encountered with Python module compatibility and my lack of knowledge of this programming language)

### What did I create?

1. A new user on the Internal VPS named "api".
2. Isolated Python environment (venv) and module installation:
   - fastapi
   - uvicorn
   - openai
   - pydantic
3. A script written in Python that runs after every reboot thanks to systemd.
4. A configuration file that stores my ChatGPT API key and the token needed to send the prompt.


### How does the script work?

1. Validates the presence of the API key and token in the configuration files and then embeds them in variables needed for the script to continue running.
2. Collects logs in the "ai.log" file.
3. Runs the FastAPI application exposed by the ASGI server (uvicorn) on address 10.10.10.1 and port 8000.
4. Provides an endpoint: POST /ask , which accepts JSON in the format: {"prompt": "Request body"}
5. Checks the x-token header sent by the DMZ (returns 403 Forbidden if bad, continues if good).
6. Sends a query to the gpt-4o-mini model via the official OpenAI SDK.
7. Logs token usage to monitor costs.
8. Returns the response in JSON format to the DMZ.

### Additional Security

- port 8000 is only accessible from the VPN address 10.10.10.2 (DMZ)
- the api user does not have SSH access
- the API key is not in the repository or source code
- the service is managed by systemd

<img width="761" height="713" alt="API-before-prompt" src="https://github.com/user-attachments/assets/07e9c78e-5433-48ab-857c-cee47c6b7966" />
<br></br>
<img width="756" height="867" alt="API-after-prompt" src="https://github.com/user-attachments/assets/beeabf3e-7130-4846-9266-00e5cad38413" />
<br></br>
<img width="844" height="394" alt="api_systemd" src="https://github.com/user-attachments/assets/ae74077b-360b-4428-b457-a17c750752b4" />
<br></br>
<img width="836" height="323" alt="api_env_conf" src="https://github.com/user-attachments/assets/b6aeb387-4eba-4667-8559-8f2a343e694a" />
<br></br>
<img width="841" height="826" alt="python-api" src="https://github.com/user-attachments/assets/b4e91219-0951-4db7-b0a5-2285b0e3af01" />
<br></br>
<img width="844" height="570" alt="python-api2" src="https://github.com/user-attachments/assets/0645e170-57cb-4e69-924a-3952c339a58f" />



# February 11, 2026

## Updating SSH Keys

The previous day, I implemented a key-only authentication method for both the DMZ VPS and the internal VPS. I also changed the firewall rules to allow only my home IP address to connect to the internal VPS (SSH), and only the internal IP address to log in via SSH to the DMZ. A KVM from the VPS provider is acting as the breaking glass. In an emergency, it will serve as an access point to the DMZ system.


<img width="472" height="118" alt="SSH-password-internal" src="https://github.com/user-attachments/assets/c0e498e7-744f-4c0a-bc72-d1ffa942905a" />
<br></br>
<img width="503" height="103" alt="SSH-key-internal" src="https://github.com/user-attachments/assets/5579eff6-bef5-4a7d-adbc-0d701935e1c9" />

(Above are two examples of logging in to the internal VPS from my home network, the first one was blocked when trying with a password, the second one was correct with privatekey)
<br></br>
<br></br>
<img width="663" height="158" alt="ssh-pass-mypc-dmz" src="https://github.com/user-attachments/assets/7b3bfbe2-8d8a-4b92-858e-4ac64f9d7439" />

(The screenshot above shows logging in from my home network to the dmz. The firewall allows access only from the internal VPS IP address)
<br></br>
<br></br>
<img width="435" height="66" alt="ssh-pass-internal-dmz" src="https://github.com/user-attachments/assets/2a8cad04-0f27-468b-ba04-19234cf696f9" />
<br></br>
<img width="528" height="78" alt="SSH-key-internal-dmz" src="https://github.com/user-attachments/assets/1bf292f5-8686-4390-967f-ca1e597bc7d8" />

(Above two examples of logging to the DMZ VPS from Internal VPS, the first one blocked when trying with a password, the second one correct with privatekey)
<br></br>
<br></br>

## Patching the Hytale Server

After the server reboot, it turned out that blocking the user hytale from accessing /bin/bash was a critical mistake,
which prevented tmux from starting. Most likely, a previously started manual instance of this process must have remained active, which distracted me and created the impression that the process was working correctly.
It's difficult for me to say for sure, as this entire project is my first Linux-based infrastructure, which means I'm constantly experimenting and learning new things.
As of today, everything is working properly. The Hytale Server starts after a reboot, while the SSH connection block for the hytale user is defined only in configuration files.
<br></br>

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


<img width="689" height="173" alt="wazuh_agents" src="https://github.com/user-attachments/assets/42809731-69f1-4557-8a2e-0a5e556e7877" />
<br></br>
<img width="1919" height="1031" alt="wazuh_dashboard" src="https://github.com/user-attachments/assets/a0b96ed7-70c5-49d8-8673-11cd1e66253a" />
<br></br>
(To log in to the dashboard page, I need to connect to the Internal VPS IP from my home network on port 443 and then provide my login details)

<br></br>

<img width="1919" height="965" alt="wazuh-agent" src="https://github.com/user-attachments/assets/f6279338-6368-494a-b40f-ff763d6978cd" />


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
