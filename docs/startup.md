# Starting up a project

## Initalise a project
- Ensure that the relevant project docker image is in the machine

```
sudo docker run --mount type=bind,src=<PATH TO STORE BLOCKCHAIN FILES>,dst=/opt/blockchain --rm -it kava:v1.0.0 init MONIKER --chain-id <CHAIN_ID> --home /opt/blockchain
```
This will create the inital setup for the relevant project

## Startup
- Ensure that you have the correct genesis file
- Persistent peers, seeds

A start up script is used to simpliy this process:

> startup.sh
```
OSMOSIS_VERSION=$1;

sudo docker stop osmosis-sentry-node
sudo docker rm osmosis-sentry-node

sudo docker run \
  --mount type=bind,source=<PATH TO STORE BLOCKCHAIN FILES>,target=/opt/blockchain \
  -w /opt/blockchain \
  --name osmosis-sentry-node \
  -p 26657:26657 -p 26656:26656 \
  -td osmosis:$OSMOSIS_VERSION start \
  --p2p.seeds="" \
  --p2p.persistent_peers="" \
  --home /opt/blockchain \
  --rpc.laddr=tcp://0.0.0.0:26657 \
  --p2p.laddr=tcp://0.0.0.0:26656
```

To run the startup script with the version number from the docker image
```
./startup.sh v1.0.0
```