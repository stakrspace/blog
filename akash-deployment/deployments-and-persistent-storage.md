Akash testnet 3 deployment and persistent storage guide
After proper setting up the testnet environment you might ask what is the authorized spend for another account on Akash?
The answer is - it could be pretty useful to delegate the deployment rights to another teammate or devops helping with the deployment independently.
</br>Here are the cli commands to do that:</br>
```akash keys add <name for the second key to spend>```</br>
```akash tx deployment authz grant <akash1...(second key address)> 1000000uakt --from <akash1...(authorizing key address)> --fees 5000uakt -y```
</br>check the successful authorization:</br>

```akash query authz grants <akash1...(authorizing key address)> <akash1...(second key address)>```</br>
example output:

```
grants:
- authorization:
    '@type': /akash.deployment.v1beta2.DepositDeploymentAuthorization
    spend_limit:
      amount: "1000000"
      denom: uakt
  expiration: "2023-03-08T13:37:44Z"
pagination:
  next_key: null
  total: "0" 
  ```
  </br>
  
Don’t forget to fund the <akash1…(second key address)> from the faucet or via the command:

```akash tx bank send <akash1...(authorizing key address)> <akash1...(second key address)> 1000000uakt --fees 5000uakt -y```

not to get the obvious error:

```
Error: rpc error: code = NotFound desc = rpc error: code = NotFound desc = account <akash1...(second key address)> not found: key not found
```

You could create the deployment then:


```akash tx deployment create --depositor-account <akash1...(authorizing key address)> --from <akash1...(second key address)> deploy.yaml -y --fees 5000uakt```

You can authorize the <akash1…(authorizing key address)> to spend the funds for deployment from <akash1…(second key address)>, just swapping the addresses in ‘akash tx deployment authz grant …’ command.

If you wish to cancel the deployment:

```akash tx deployment close --dseq <deployment sequence number from the create command> --fees 5000uakt --from <akash1...(second key address)> -y```</br></br>
And now, for something completely different! The persistent storage explanation — an example to add postgres persistent storage for your Grafana:</br></br>
Create pers.yaml

```version: "2.0"
services:
  postgres:
    image: postgres
    params:
      storage:
        data:
          mount: /var/lib/postgres
  grafana:
    image: grafana/grafana
    expose:
      - port: 3000
        as: 80
        to:
          - global: true
        accept:
          - webperstest.localhost
    params:
      storage:
        data:
          mount: /var/lib/grafana
profiles:
  compute:
    grafana-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 1Gi
            attributes:
              persistent: true
              class: beta2
    postgres-profile:
      resources:
        cpu:
          units: 1
        memory:
          size: 1Gi
        storage:
          - size: 512Mi
          - name: data
            size: 10Gi
            attributes:
              persistent: true
              class: beta2
  placement:
    westcoast:
      attributes:
        region: us-west
      pricing:
        grafana-profile:
          denom: uakt
          amount: 1000
        postgres-profile:
          denom: uakt
          amount: 7000
deployment:
  grafana:
    westcoast:
      profile: grafana-profile
      count: 1
  postgres:
    westcoast:
      profile: postgres-profile
      count: 1
```

Create deployment, assuming you have the environment set as in Akash testnet docs:

```akash tx deployment create pers.yaml --from $AKASH_KEY_NAME --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID --gas-prices="0.025uakt" --gas="auto" --gas-adjustment=1.5 -y```

set AKASH_DSEQ=<dseq no from deployment create output> and query the market bid

```akash query market bid list — owner=$AKASH_ACCOUNT_ADDRESS — node $AKASH_NODE — dseq $AKASH_DSEQ```

choose the provider and set the var AKASH_PROVIDER=<provider addr from output> and create the lease

```akash tx market lease create --chain-id $AKASH_CHAIN_ID --node $AKASH_NODE --owner $AKASH_ACCOUNT_ADDRESS --dseq $AKASH_DSEQ --provider $AKASH_PROVIDER --from $AKASH_KEY_NAME --gas-prices="0.025uakt" --gas="auto" --gas-adjustment=1.5 -y```

query the leases list
  
```akash query market lease list --owner $AKASH_ACCOUNT_ADDRESS --node $AKASH_NODE --dseq $AKASH_DSEQ```
  
send manifest and check lease status

```akash provider send-manifest pers.yaml --node $AKASH_NODE --dseq $AKASH_DSEQ --provider $AKASH_PROVIDER --home ~/.akash --from $AKASH_KEY_NAME && akash provider lease-status --node $AKASH_NODE --home ~/.akash --dseq $AKASH_DSEQ --from $AKASH_KEY_NAME --provider $AKASH_PROVIDER```

If it worked, you can close the lease when necessary.
