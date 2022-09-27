
# Tendermint KMS (TMKMS)

Key Management System for  [Tendermint](https://tendermint.com/)  applications such as  [Cosmos Validators](https://cosmos.network/docs/gaia/validators/validator-faq.html).

Provides isolated, optionally HSM-backed signing key management for Tendermint applications including validators, oracles, IBC relayers, and other transaction signing applications.

## About

This repository contains  `tmkms`, a key management service intended to be deployed in conjunction with  [Tendermint](https://tendermint.com/)  applications (ideally on separate physical hosts) which provides the following:

-   **High-availability**  access to validator signing keys
-   **Double-signing**  prevention even in the event the validator process is compromised
-   **Hardware security module**  storage for validator keys which can survive host compromise

## Installation
> In this example, we will be using user as root and tendermint project will be used in a docker container

1.  Install tmkms dependencies
    
    - Install rust, gcc, libusb
        ```
        curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
        source $HOME/.cargo/env 
        sudo apt update
        sudo apt install git build-essential ufw curl jq snapd --yes
        sudo apt install libusb-1.0-0-dev -y
        ```
        
2.  Install tmkms from repo
    
    ```
    cd $HOME
    git clone https://github.com/iqlusioninc/tmkms.git
    cd $HOME/tmkms
    cargo install tmkms --features=softsign
    ```
        
3.  Initialise the relevant project, for example osmosis project → this will generate the config, data folder + `priv_validator_key.json` and `node_key.json`and `priv_validator_state.json`
    
4.  Create a new folder to store the tmkms configurations, secrets(private keys, secret keys) for the project(osmosis)
    
    `mkdir /root/kms/osmosis`
    
5.  Init a new tmkms service for the project
    
    `tmkms init /root/kms/osmosis`
    
     This will create schema, secrets, state, tmkms.toml config files in the /root/kms/osmosis folder
        
6.  Create `secret_connection_key` using tmkms for the project
    
    `tmkms softsign keygen /root/kms/osmosis/secrets/secret_connection_key`
    
7.  Copy `priv_validator_key.json` from the project into tmkms folder, remember to backup `priv_validator_key.json`
    
    ```
    cp /blockchain/osmosis/osmsosis-1/config/priv_validator_key.json /root/kms/osmosis/secrets

    + 

    BACKUP priv_validator_key.json to somewhere safe
    ```
    
8.  Import private key(`priv_validator_key.json`) of validator into tmkms
    
    `1tmkms softsign import /root/kms/osmosis/secrets/priv_validator_key.json /root/kms/osmosis/secrets/priv_validator_key`
    
    1.  This will generate a newly `priv_validator_key` which will be what TMKMS will use to sign for your validator
        
9.  Remember to backup `priv_validator_key.json` and at this point, you could delete the `priv_validator_key.json` from both your validator node and tmkms node and store it safely offline in case of an emergency.
    
10.  Modify the `tmkms.toml` , the configuration shown in the tmkms.toml is a template
    
    `nano /root/kms/osmosis/tmkms.toml`
    
    1.  Change the chain id, account_key_prefix, consensus_key_prefix, state_file, path of providers.softsign, validator addr, secret_key to match the relevant project
        
        ```
        # Tendermint KMS configuration file

        ## Chain Configuration

        ### Cosmos Hub Network

        [[chain]]
        id = "osmosis-1"  ==> Change this to match current project
        key_format = { type = "bech32", account_key_prefix = "osmopub", consensus_key_prefix = "osmovalconspub" } ==> Change this to match current project
        state_file = "/root/kms/osmosis/state/priv_validator_state.json" ==> Change this to match current project

        ## Signing Provider Configuration

        ### Software-based Signer Configuration

        [[providers.softsign]]
        chain_ids = ["osmosis-1"] ==> Change this to match current project
        key_type = "consensus"
        path = "/root/kms/osmosis/secrets/priv_validator_key" ==> Change this to match current project

        ## Validator Configuration

        [[validator]]
        chain_id = "osmosis-1" ==> Change this to match current project
        addr = "tcp://localhost:26659" ==> Change this to match current project
        secret_key = "/root/kms/osmosis/secrets/secret_connection_key" ==> Change this to match current project
        protocol_version = "v0.34"
        reconnect = true
        ```
        
11.  Modify the project config.toml to use the port selected in tmkms.toml
    
    `nano /blockchain/osmosis/osmsosis-1/config/config.tml`
    
    `priv_validator_laddr = "tcp://0.0.0.0:26659"`
        
    - Comment out the priv_validator_key_file line and the priv_validator_state_file line:
        
        ```
        # Path to the JSON file containing the private key to use as a validator in the consensus protocol
        # priv_validator_key_file = "config/priv_validator_key.json"

        # Path to the JSON file containing the last sign state of a validator
        # priv_validator_state_file = "data/priv_validator_state.json"
        ```
        
12.  In the docker startup script, add in the ports to expose ports for priv_validator_laddr and restart the docker container
    
    ```
    #Startup script
    .
    .
    sudo docker run \
    -w /opt/blockchain \
    --mount type=bind,source=/blockchain/osmosis/osmosis-1,target=/opt/blockchain \
    --name osmosis-testnet-val \
    -p 26657:26657 -p 26657:26656 -p 26659:26659 -p 1317:1317 \
    -td osmosis:$VERSION start \
    .
    .
    ```
    
13.  Create a systemd file to start up the tmkms service
    
    ```
    sudo tee <<EOF >/dev/null /etc/systemd/system/tmkms-osmosis.service

    [Unit]
    Description=TMKMS Service for Device osmosis testnet
    [Service]
    User=root
    Type=simple
    SyslogIdentifier=tmkms-osmosis-testnet
    TimeoutSec=0
    ExecStart=/root/.cargo/bin/tmkms start -c /root/kms/osmosis/tmkms.toml
    Restart=no
    WorkingDirectory=/root/kms/osmosis
    EOF
    ```
    
14.  Start up the systemd file
    
    ```
    sudo systemctl daemon-reload
    sudo systemctl start tmkms-osmosis.service
    sudo journalctl -u tmkms-osmosis.service -f
    ```
    
    - You will see error logs like the following:
        
        ```
        2022-03-08T23:42:37.926816Z  INFO tmkms::commands::start: tmkms 0.11.0 starting up...
        2022-03-08T23:42:37.926968Z  INFO tmkms::keyring: [keyring:softsign] added consensus Ed25519 key: xxx
        2022-03-08T23:42:37.927216Z  INFO tmkms::connection::tcp: KMS node ID: xxx
        2022-03-08T23:42:37.929454Z ERROR tmkms::client: [osmosis-1@tcp://xxx:26659] I/O error: Connection refused (os error 111)
        2022-03-08T23:42:38.929746Z  INFO tmkms::connection::tcp: KMS node ID: xxx
        2022-03-08T23:42:38.931428Z ERROR tmkms::client: [osmosis-1@tcp://xxx:26659] I/O error: Connection refused (os error 111)
        2022-03-08T23:42:39.931729Z  INFO tmkms::connection::tcp: KMS node ID: xxx
        2022-03-08T23:42:39.932417Z ERROR tmkms::client: [osmosis-1@tcp://xxx:26659] I/O error: Connection refused (os error 111)
        2022-03-08T23:42:40.932732Z  INFO tmkms::connection::tcp: KMS node ID: xxx
        2022-03-08T23:42:40.933425Z ERROR tmkms::client: [osmosis-1@tcp://xxx:26659] I/O error: Connection refused (os error 111)
        ```
        
15.  Start up the docker container for the project
    
    - Start osmosis docker container 
    `./start.sh v1.0.0`
    
16.  Your TMKMS service file should see logs like the following *signed PreCommit*, indicating its signing blocks:
    
    ```
    2022-03-08T23:46:06.208451Z  INFO tmkms::connection::tcp: KMS node ID: xxx
    2022-03-08T23:46:06.210568Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] connected to validator successfully
    2022-03-08T23:46:06.210604Z  WARN tmkms::session: [osmosis-1@tcp://xxx:26659]: unverified validator peer ID! (xxx)
    2022-03-08T23:46:15.929787Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] signed PreCommit:<nil> at h/r/s 3399910/0/2 (0 ms)
    2022-03-08T23:46:17.344579Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] signed PreCommit:<nil> at h/r/s 3399911/0/2 (0 ms)
    2022-03-08T23:46:22.367627Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] signed PreCommit:<nil> at h/r/s 3399912/0/2 (0 ms)
    2022-03-08T23:46:27.409777Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] signed PreCommit:<nil> at h/r/s 3399913/0/2 (0 ms)
    2022-03-08T23:46:32.442300Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] signed PreCommit:<nil> at h/r/s 3399914/0/2 (0 ms)
    2022-03-08T23:46:37.452162Z  INFO tmkms::session: [osmosis-1@tcp://xxx:26659] signed PreCommit:<nil> at h/r/s 3399915/0/2 (0 ms)
    ```
    

References links:

[TMKMS Repo](https://github.com/iqlusioninc/tmkms.git)

[https://docs.osmosis.zone/developing/keys/tmkms.html#prepare-tmkms-dependencies](https://docs.osmosis.zone/developing/keys/tmkms.html#prepare-tmkms-dependencies)

[GitHub - iqlusioninc/tmkms: Tendermint KMS: Key Management System for Tendermint Validators](https://github.com/iqlusioninc/tmkms)

[BitCanna validator security: Introducing TMKS in BitCanna](https://stakely.io/en/blog/bitcanna-advanced-validator-security-tendermint-key-management-system-tmkms)

