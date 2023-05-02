A Step-by-Step Guide to Running a Humans.ai Testnet Node

Introduction

Humans.ai is an innovative platform that leverages the power of AI and blockchain technology to create a decentralized ecosystem. Testnets are essential to the development of blockchain platforms, as they allow developers and users to experiment with new features and test the network's robustness.

Let's dive in!

Step 1: Download the binary

The first step is to download the Humans.ai actual release from Github repository.

https://github.com/humansdotai/humans/releases

Step 2:
```tar -xvf humans_*.tar.gz```

Step 3: Initialize the Node

To initialize your testnet node, run the following command:

```./humansd init [your-node-name] --chain-id [testnet-chain-id]```

Replace [your-node-name] with a custom name for your node and [testnet-chain-id] with the specific testnet's chain ID provided in the testnet repository.
https://github.com/humansdotai/testnets


Step 4: Add the Genesis File

After initializing your node, download the testnet's genesis file by running:

```wget [genesis-file-url]```
Replace [genesis-file-url] with the URL of the genesis file provided in the testnet repository. Next, move the downloaded genesis file to the appropriate directory:

```mv [downloaded-genesis-file] ~/.humansd/config/genesis.json ```

Replace [downloaded-genesis-file] with the name of the downloaded genesis file.

Step 5: Configure Your Node

To configure your node, update the necessary parameters in the ~/.humansd/config/config.toml file:

Add the seed nodes addresses and fill in persistent_peers field.

Step 6: Start the Node

Now that your node is configured, start it by running:

```./humansd start```

Your node will now connect to the Humans.ai testnet and begin syncing with the blockchain. Since you submitted the gentx, your node is the validator by default.

Congratulations! You have successfully set up a Humans.ai testnet node. Your node will now contribute to the stability and security of the network, enabling you to test new features, participate in governance decisions, and take part in decentralization.
