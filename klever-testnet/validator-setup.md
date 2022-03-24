Hello, here is the way to setup your klever node - first, you need to install docker, in a usual way;
then you can generate the key and address to fund 

``` cd && mkdir wallet && docker run -it --rm --user "$(id -u):$(id -g)" -v $(pwd)/wallet:/opt/klever-blockchain --entrypoint=/usr/local/bin/operator kleverapp/klever-go-testnet:latest account create ```

to check the successfull key creation run

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest account address
```
    
and after account funded you may start the node according to official Klever docs:
    
- Create statics directory
Create a folder where all the data will be stored:

``` mkdir -p $(pwd)/node/config $(pwd)/node/db $(pwd)/node/logs ```

- Network genesis config

Download latest config file (http://) and extract into your config directory

```
curl -k https://backup.testnet.klever.finance/config.testnet.100008.tar.gz \
    | tar -xz -C ./node
```
    
    
- Create your validator BLS Key

Execute a command inside the docker container to create wallets for validators. The data is then forwarded to the directory created previously. 

``` 
docker run -it --rm -v $(pwd)/node/config:/opt/klever-blockchain \
    --user "$(id -u):$(id -g)" \
    --entrypoint='' kleverapp/klever-go-testnet:latest keygenerator
```
    
- Download and extract data backup for fast spin
Download a backup in case it exists. Every time the network is reset, files change as well. So, this backup is necessary. 

``` 
curl -k https://backup.testnet.klever.finance/kleverchain.latest.tar.gz \
    | tar -xz -C ./node
```
    
- Run your node

The command for running a node comes with a few settings: it includes mappings for cryptographic keys, data directories and logs, as well as network ports, the application that will run when the docker image is executed and the file of node validators signature.

```
docker run -it --rm \
    --user "$(id -u):$(id -g)" \
    --name klever-node \
    -v $(pwd)/node/config:/opt/klever-blockchain/config/node \
    -v $(pwd)/node/db:/opt/klever-blockchain/db \
    -v $(pwd)/node/logs:/opt/klever-blockchain/logs \
    --network=host \
    --entrypoint=/usr/local/bin/validator \
    kleverapp/klever-go-testnet:latest \
    '--log-save' '--rest-api-interface=0.0.0.0:8080'
```

This way is to run node not in background, it's supposed that you use ```tmux``` or ```screen``` to observe the node status.

After that you should register the validator using:

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \
    validator create \
    <your klv1 address> \
    --bls=<bls address from ~/node/validatorKey.pem> \
    --rewards=<your klv1 address> \
    --name=<your validator name> \
    --commission=5 \
    --maxDelegation=12000000 \
    --uris="github=github.com/<your github>"
```   

The funds from the account need to be frozen first

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance account freeze 1500000
```

And delegated after

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \
    account delegate \
    <klv1....> \
    --bucketID=<frozen txid>
```
The monitoring guide follows...
