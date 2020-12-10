# Deploying owncloud on Akash

This is a guide to deploying on Akash an owncloud docker app in a non-custodial way. Akash is a permissionless and censorship-resistant cloud network that guarantees sovereignty over your data and your applications. With Akash, you’re in complete control of all aspects of the life cycle of an application with no middleman.

Readme is adapted from Deploying ThorChain BEPSwap UI on Akash, which is an excellent guide to getting started with DeFi deployments on Akash DeCloud.

## Before We Start

This is a technical guide, best suited to a reader with basic Linux command line knowledge. Anyone who is able to use Linux shell will be able to do the necessary steps. Application developers with little or no systems administration experience, wanting to deploy applications on the decentralized cloud or system administrators with little or no experience with infrastructure automation, wanting to learn more. Anyone who wants to get a feel for the current state of the decentralized cloud ecosystem.

You will need the below setup before we being:

1. Install Akash: Make sure to have Akash client installed on your workstation, check [install guide](https://docs.akash.network/guides/install) for instructions.
2. Choose Your Akash Network: You'll need to know information about the network you're connecting your node to. See [Choosing a Network](https://docs.akash.network/guides/version) for how to obtain any network-related information.
3. Fund Your Account: You'll need a AKT wallet with funds to pay for your deployment. See the [funding guide](https://docs.akash.network/guides/funding)
   creating a key and funding your account.
4. You don't even need to install docker since you can choose any public available image on dockerhub.

### Set up your Environment

We will be using shell variables throughout this guide for convenience and clarity. Ensure you have the below set of variables defined on your shell, you can use `export VARNAME=...`:

| Name              | Description                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- |
| `AKASH_NODE`      | Akash network configuration base URL. See [here](/guides/version.md).                                               |
| `AKASH_CHAIN_ID`  | Chain ID of the Akash network connecting to. See [here](/guides/version.md).                                        |
| `ACCOUNT_ADDRESS` | The address of your account. See [here](/guides/wallet/README.md#account-address).                                  |
| `KEY_NAME`        | The name of the key you will be deploying from. See [here](/guides/wallet/README.md) if you haven't yet setup a key |

Verify you have correct `$AKASH_NODE`, that you have populated while [configuring the connection](/guides/version) using `export AKASH_NODE=$(curl -s "$AKASH_NET/rpc-nodes.txt" | shuf -n 1)`.

```sh
echo $AKASH_NODE $AKASH_CHAIN_ID
```

You should see a response similar to:

```
tcp://rpc-edgenet.akashdev.net:2665 akash-edgenet-1
```

Your values may differ depending on the network you're connecting to, `tcp://rpc-edgenet.akashdev.net:2665` and `akash-edgenet-1` are details for [edgenet](https://github.com/ovrclk/net/tree/master/edgenet).

Verify you have the key set up and your account has sufficient balances, see the [funding guide](/guides/wallet/funding.md) otherwise:

My local key is named `bob`, the below command should return the name you've used:

```sh
echo $KEY_NAME
```

The above should return a response similar to:

```
bob
```

Populate `ACCOUNT_ADDRESS` from `KEY_NAME` and verify:

```sh
export ACCOUNT_ADDRESS="$(akash keys show $KEY_NAME -a)"

echo $ACCOUNT_ADDRESS

akash1tdg2j786lgvdzh3flcu29qhq5rv093zuajxgag
```

Fund your account on [faucet]https://akash.vitwit.com/faucet


Check your account has sufficient balance by running:

```sh
akash query bank balances --node $AKASH_NODE $ACCOUNT_ADDRESS
```

You should see a response similar to:

```
balances:
- amount: "93000637"
  denom: uakt
pagination:
  next_key: null
  total: "0"
```

Please note the balance indicated is is denominated in uAKT (AKT * 10^-6), in the above example, the account has a balance of *93 AKT\*. We're now setup to deploy.


## Create the Deployment

Create a deployment configuration [owncloud.yml](deploy.yml) to deploy the `owncloud` for [owncloud docker image](https://github.com/docker-library/owncloud) container using [SDL](https://docs.akash.network/documentation/sdl):

```sh
cat > owncloud.yml <<EOF
---
version: "2.0"

services:
  owncloud:
    image: owncloud 
    expose:
      - port: 80
        to:
          - global: true

profiles:
  compute:
    owncloud:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          size: 2Gi
  placement:
    westcoast:
      signedBy:
        anyOf:
          - "akash1vz375dkt0c60annyp6mkzeejfq0qpyevhseu05"
      pricing:
        owncloud: 
          denom: uakt
          amount: 5000

deployment:
  owncloud:
    westcoast:
      profile: owncloud
      count: 1
      
EOF>>
```

You may use the sample deployment file as-is or modify it for your own needs as desscribed in our [SDL (Stack Definition Language](https://docs.akash.network/documentation/sdl) documentation.

Please note if you are running on the testnet, you are limited in the amount of testnet resources you may request.

To deploy on Akash, run:

```sh
akash tx deployment create owncloud.yml --from $KEY_NAME --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID --fees 5000uakt -y
```

In this step, you post your deployment, the Akash marketplace matches you with a provider via auction. To create a deployment use akash deployment. The syntax for the deployment is `akash tx deployment create <config-path> --from <key-name>`.

You can check the status of your lease by running:

```
akash query market lease list --owner $ACCOUNT_ADDRESS --node $AKASH_NODE --state active

```

```yaml
leases:
- lease_id:
    dseq: "185396"
    gseq: 1
    oseq: 1
    owner: akash1tdg2j786lgvdzh3flcu29qhq5rv093zuajxgag
    provider: akash15ql9ycjkkxhpc2nxtnf78qqjguwzz8gc4ue7wl
  price:
    amount: "732"
    denom: uakt
  state: active
pagination:
  next_key: null
  total: "0"
```

In the above example, we can see that a lease is created using for _732 uakt_ or _0.0000732 AKT_ per block to execute the container.

For convenience and clarity for future referencing, we can extract the below set of values to shell variables that we will be using to reference the deployment:

| Attribute  | Value                                          |
| ---------- | ---------------------------------------------- |
| `PROVIDER` | `akash15ql9ycjkkxhpc2nxtnf78qqjguwzz8gc4ue7wl` |
| `DSEQ`     | `185396`                                       |
| `OSEQ`     | `1`                                            |
| `GSEQ`     | `1`                                            |

Verify we have the right values populated by running:

```sh
echo $PROVIDER $DSEQ $OSEQ $GSEQ
```

You should see a response similar to:

```
akash15ql9ycjkkxhpc2nxtnf78qqjguwzz8gc4ue7wl 185396 1 1
```

Upload the manifest using the values from above step:

```sh
akash provider send-manifest owncloud.yml --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --owner $ACCOUNT_ADDRESS --provider $PROVIDER
```

Your image is now deployed, once you uploaded the manifest. You can retrieve the access details by running the below:

```sh
akash provider lease-status --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --provider $PROVIDER --owner $ACCOUNT_ADDRESS
```

You should see a response similar to:

```json
{
  "services": {
    "owncloud": {
      "name": "owncloud",
      "available": 1,
      "total": 1,
      "uris": [
        "eug3adjnllaop39fj720goabfg.provider4.akashdev.net"
      ],
      "observed-generation": 0,
      "replicas": 0,
      "updated-replicas": 0,
      "ready-replicas": 0,
      "available-replicas": 0
    }
  },
  "forwarded-ports": {}
}
```

You can access the application by visiting the hostnames mapped to your deployment. In above example, its http://eug3adjnllaop39fj720goabfg.provider4.akashdev.net


## Close your deployment

When you are done with your application, close the deployment. This will deprovision your container and stop the token transfer. Close deployment using deployment by creating a `deployment-close` transaction:

```sh
akash tx deployment close --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID --dseq $DSEQ --owner $ACCOUNT_ADDRESS --from $KEY_NAME -y --fees 5000uakt
```

Additionally, you can also query the market to check if your lease is closed:

```sh
akash query market lease list --owner $ACCOUNT_ADDRESS --node $AKASH_NODE
```

You should see a response similar to:

```yaml
leases:
  - lease_id:
      dseq: "185396"
      gseq: 1
      oseq: 1
      owner: akash1tdg2j786lgvdzh3flcu29qhq5rv093zuajxgag
      provider: akash15ql9ycjkkxhpc2nxtnf78qqjguwzz8gc4ue7wl
    price:
      amount: "343"
      denom: uakt
    state: closed
pagination:
  next_key: null
  total: "0"
```
