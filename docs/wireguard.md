
# Wireguard setup and config

## **Concepts**

Wireguard is a kernel level virtual network interface that allows users to create a peer to peer VPN. The protocol uses ed25519 keys for encryption, and UDP as the session protocol.

Reference Link: [https://upcloud.com/community/tutorials/get-started-wireguard-vpn/](https://upcloud.com/community/tutorials/get-started-wireguard-vpn/)  
  
[https://dev.to/tangramvision/what-they-don-t-tell-you-about-setting-up-a-wireguard-vpn-1h2g](https://dev.to/tangramvision/what-they-don-t-tell-you-about-setting-up-a-wireguard-vpn-1h2g)

----------

  

**Peer Connection Info**

Every Wireguard peer needs to have a list of white-listed peers / nodes that it will accept connection from. The peer information required is show as below:

    [Peer]
    PublicKey = xxx
    AllowedIPs = 1.2.3.4/32
    Endpoint = IP ADDR:WIREGUARD_PORT

  

**PublicKey**  The ed25519 public key of the peer you are connecting to.  
  
**AllowedIPs**  The list / block of ip addresses the peer is listening on for Wireguard connections.  
  
/32  prefix is used to indicate that there is a unique ip address for each peer.  
  
**Endpoint**  The endpoint that consist of the public ip of the server and the Wireguard UDP port that the wireguard service is listening to.

**Wireguard Service Info**  
  
Each wireguard interface needs a single config.ini file to configure it  
  
The minimum file required is shown below:

    [Interface]
    ListenPort = 12345 # Wireguard port
    PrivateKey = <xxx>
     
    [Peer] ...

**ListenPort**  The UDP port for the wireguard interface to listen to  
  
**PrivateKey**  The private key, generated by wireguard, for the interface to use to decrypt communications  
  
**PeerInfo**  
The Peer Info block is below the [Interface] block, each interface can have any number of peers

## Installation

**Ubuntu 20.04 and above**

 Install wireguard
 `sudo apt install wireguard wireguard-tools wireguard-dkms`

## Configuration

This guides outlines the configuration of wireguard  between 2 machines(machines A, machine B)
Example: Private IP for A(1.2.3.3), Private IP for B(1.2.3.4)

1.  Create new private key
	`wg genkey > wg0.priv`

2. Retrieve public key
	 `wg pubkey < wg0.priv`
	 
3. Prepare the config file under /etc/wireguard folder
`(`
`cat <<-EOF`
`[Interface]`
`ListenPort =` `50010`
`PrivateKey = xxx # This machine`
`# The other machine`
`[Peer]`
`PublicKey = xxx # Other machine pub key`
`AllowedIPs =` `1.2.3.4/32`
`Endpoint =` `IPADDR:WIREGUARD_PORT`
`PersistentKeepalive =` `25`
`EOF`
`) | tee /etc/wireguard/wg0.ini`

4. Configure the network interface
	

> The IP addr add command adds a /24 ip prefix, which should not overlap
> with the following network
> -   Cloud Provider Private IP
> -   Other wireguard networks

    ip link add wg0 type wireguard
    
    To update the peers for the wireguard:
    wg setconf wg0 /etc/wireguard/wg0-config.ini
    
    # Set a private ip that is unique from the rest of the servers
    ip addr add 1.2.3.3/24 dev wg0
    
    ip link set wg0 up

5. Test the new ip configuration after setting up the wiraguard on both machine A and B

On the machine A
`ping 1.2.3.4` 

On the machine B
`ping 1.2.3.3`

If the ping passes through successfully, connection of wireguard is established throuugh these 2 machines