# Validator Architecture
In this setup, the validator node hides behind it sentry nodes. These sentries are full node and are connected to validator node through a secure link by wireguard. This adds an additonal layer of security to protect the validator node from DDOS attacks

## Network Diagram
![Diagram](/images/network_diagram.png)
---
## [Set up wireguard (1 validator + 3 sentries)](/docs/wireguard.md)
---
## [Harden OS, SSH](/docs/hardening.md)
---
## [Docker & AWS ECR](/docs/docker.md)
---
## [Setup TMKMS](/docs/tmkms.md)
---
## [Starting up project](/docs/startup.md)