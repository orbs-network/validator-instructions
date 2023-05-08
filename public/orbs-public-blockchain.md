# Setting up an Orbs Public Blockchain Node using Polygon CLI

This step-by-step guide will walk you through creating a new node and connecting it to an existing Orbs network.

![](../diagram.png)

## Prerequisites

To complete this guide you will need the following set up:

- A Mac or Linux machine
- An Ethereum Endpoint URL. You may use an [Infura free tier account](infura-setup-free.md).
- **A clean, new AWS account with admin programmatic access.**
- The AWS CLI with an AWS credentials profile configured. See [AWS's getting started guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
- If you are already participating in Orbs v1 and have deployed a Validator node, you may use the same AWS account
- An Orbs Node (Wallet) Address. For details see [below](#allocate-orbs-node-address-and-private-key)
- A Guardian (Wallet) Address
- Metamask installed
- [Node.js](https://nodejs.org/en/) version 18 or above

  Use `brew install node` to install Node.js

- [Terraform](https://www.terraform.io/downloads.html)

  Polygon currently supports the Terraform v0.12.23

  Use

  `brew install tfenv`

  `tfenv install 0.12.23`

  `tfenv use 0.12.23`

  to install Terraform

  Expected version `Terraform v0.12.23`, verify yours with `terraform -v`

### Generate SSH public and private keys

If you already have an ssh public key to use with your Orbs Node instaces, you may skip this step.

A valid public/private keypair is needed to run the deployment scripts and set up the EC2 resources. The key file should remain secret with the exception of feeding it to the configuration during setup (providing the path for the pub file in the `orbs-node.json` setup file as described below)

The generated key should **not** have a passphrase.
It is okay to generate a key by any means, such as based on the following tutorial by [GitHub](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

The gist of creating such a key is running:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

- -C "your_email@example.com" is an optional comment

### Allocate a static IP on Amazon

The Orbs node that you provision must have a static IPs (Elastic IP) in order to communicate with the network.

- Login to your AWS console
- Pick a region for your node deployment (for example `ca-central-1`)
- Navigate to "EC2" > "Elastic IPs" (you can also use this link - https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Addresses: - replacing "us-east-1" with your desired region)
- Allocate 1 Elastic IP by clicking the "Allocate Elastic IP address" and following the directions.

You can also do this via the AWS CLI with the command `aws ec2 allocate-address --region YOUR-DESIRED-REGION`.

The Node IP address and region will later be used in the node configuration file.
The IP will also be required during the [Guardian Registration phase](#register-your-guardian).

### Allocate Orbs Node address and private key

The Orbs Node address is a standard Ethereum Address. This address is used for (1) signing blocks on the Orbs network, and for (2) sending transactions to Orbs PoS smart contracts on Ethereum. Therefore, the Orbs node stores and uses the node address's private key. As such, the private key should be different from the Guardian private key.

During normal operation, Orbs node automatically sends transactions to the PoS smart contracts on Ethereum (e.g. to execute reward distribution or to signal that it is in sync with the network and ready to enter a committee). The Orbs Node address should hold enough ETH to fund the gas for transactions sent to the PoS contracts. It is the Guardian's responsibility to periodically verify the Orbs node address has a balance of at least 0.5 ETH.

Note - To complete registration, the Orbs node address is required to start with at least 1 ETH.

Orbs Node address and private key should be generated in a secure fashion and the private key is required during node deployment (see below).

The Orbs Node address is also required during the [Guardian Registration phase](#register-your-guardian).

The Orbs address may be modified, by updating the registration. Prior to a node address change, the node should be teared-down and redeployed.

The Orbs Node private key should be securely stored.

If needed, see the following [key generation instructions](https://github.com/orbs-network/validator-instructions/blob/master/public/key_generation.md) to generate keys using MyCrypto. You can also create it through Metamask.

### Install Polygon via NPM

To install Polygon run

    npm install -g @orbs-network/polygon

If you have previously installed Polygon and you are performing a new deployment, we recommend updating it by running `npm update -g @orbs-network/polygon`

### Create a dedicated deployment folder

Create a dedicated folder for your deployment. For example, "orbs-v2".
This folder will hold config files, logs and the artifacts for future needs (such as if you ever want to remove your AWS resources)

**Backup this folder upon completion of the instructions and DO NOT delete it.**

Change your working path to the deployment folder and proceed with the instructions below.

### Configure the boilerplate JSON file

Create a config file `orbs-node.json` for your deployment in .

The content of the `orbs-node.json` should be:

    {
        "name": "<orbs node name>",
        "awsProfile": "<aws profile>",
        "sshPublicKey": "<ssh access public key file>",
        "orbsAddress": "<orbs node ethereum address>",
        "region": "<aws region>",
        "publicIp": "<node ip>",
        "nodeSize": "r5.large",
        "nodeCount": 0,
        "cachePath": "./_terraform",
        "incomingSshCidrBlocks": ["<ssh source cird block>",...],
        "boyarAutoUpdate": true,
        "managementConfig": {
            "orchestrator": {
                "DynamicManagementConfig": {
                    "Url": "http://localhost:7666/node/management",
                    "ReadInterval": "1m",
                    "ResetTimeout": "30m"
                },
                "storage-driver": "local",
                "storage-mount-type": "bind"
            },
            "services": {
                "management-service": {
                    "InternalPort": 8080,
                    "ExternalPort": 7666,
                    "DockerConfig": {
                        "Image": "orbsnetwork/management-service",
                        "Tag": "bootstrap",
                        "Pull": true
                    },
                    "Config": {
                        "BootstrapMode": true,
                        "EthereumEndpoint": "<ethereum endpoint url>"
                    }
                }
            }
        }
    }

Where:

- `<orbs node name>` - Will be used with some AWS resource names as prefix or suffix to uniquely identify node resources. Must be S3 bucket name complient (e.g. no \_...)
- `<aws profile>` - use "default", or, if you are using AWS profiles with more than one AWS account, an AWS profile name with pre-configured AWS credentials and account for your account.
- `<ssh access public key file>` - The SSH public key filename (if omitted, defaults to `~/.ssh/id_rsa.pub`)
- `<orbs node ethereum address>` - The Orbs Node Ethereum address, EIP-55 compliant with checksum capitalization **but without the leading '0x'**
- `<aws region>` - An AWS region name. The new node will be provisioned in this region. e.g. `ca-central-1`. Ensure it is the same region as your previously created Elastic IP address.
- `<node ip>` - A static IP address where this Orbs Node will be reachable. Must be a valid Elastic IP allocated and unattached in the selected region.
- `<ethereum endpoint url>` - a URL to a working and synced Ethereum node (such as an Infura endpoint).
- `<ssh source cird block>` - One or more CIDR blocks which will be granted access to ssh from into the node. You will still need the public key to connect - it is required only in cases of troubleshooting. The format is standard CIDR. Any IP not in the range will not be able to SSH to the node, even if it has the SSH key file. To allow ssh access from any ip use `"incomingSshCidrBlocks": ["0.0.0.0/0"],`

### Deploy the Node using Polygon CLI

Temporarily store the Orbs Node private key in an environment variable **without the leading 0x**. This is to prevent your private key being inadvertently commited to source control.

Note, the private key should be a string of 64 characters, e.g. `f5f83Ee70a85fFF2exxxxxxxxxxxxxxxxxxxxxxxxxxx334932F34C8D629165Ed`.

Example command:

```
export ORBS_NODE_PRIVATE_KEY=f5f83Ee70a85fFF2exxxxxxxxxxxxxxxxxxxxxxxxxxx334932F34C8D629165Ed
```

To deploy the node run:

    polygon create -f orbs-node.json --orbs-private-key $(echo $ORBS_NODE_PRIVATE_KEY)

During node creation, Terraform cache files will be created in the folder defined in `cachePath` - these should be backed up to allow proper cleanup at a later stage.

If needed, the command to remove all resources provisioned for the node is:

    polygon destroy -f orbs-node.json

### IMPORTANT!

After deployment make sure to backup and securely store the deployment folder data, including:

- **Orbs keys** (`<orbs private key file>` and any other credentials you used and configured, such as SSH keys)
- \_terraform cache folder - is required to destroy or redeploy the node
- The `orbs-node.json` file

Do not leave sensitive data such as the `<orbs private key file>` in an unsecure disk location.

### Update Orbs Node configuration

**Any modification to your configuration file MUST be done while the node is DOWN.**

To update your node configuration

1. run polygon destroy (e.g. `polygon destroy -f orbs-node.json`)
1. update your configuration JSON file as required
1. run [polygon create](#deploy-the-node-using-polygon-cli)

### Move funds to your node address

Your node will need funds to occasionally send transactions over ethereum and polygon networks. For example when the node is ready for committee a tx will be sent over the network.
You will need to send 1 ETH and 1 MATIC to your ORBS node address.

### Register your Guardian

To register on the network, go to [Guardian Registration](https://guardians.orbs.network/registration)
(Requires Metamask)

### Verify your Node is deployed correctly

Check that all the services started without errors

```
http://<node ip>/services/boyar/status
```

Check your node's status in the PoS network (readings are 10 minutes delayed for finality)

```
http://<node ip>/services/management-service/status
```

### Polygon network support

**New Guardians** Please Register on both Ethereum and Polygon networks, using the following link
[Guardian Registration](https://guardians.orbs.network/registration)

Ethereum regisration is mandatory in order to get certified.

Changing network is available on top right near the language selector.

**Existing guardians** are already registred on both networks automatically.

**Congratulations!**

## Troubleshooting

1. If you get an Terraform error that your IP does not exist, check whether the combination of ip and region is correct in the node configuration file (`orbs-node.json`)

2. If the metrics page does not respond, it could be that the Ethereum node did not finish syncing - this takes several hours.

3. If you're having Terraform errors, check whether your clock is configured correctly

4. Contact the Orbs core team for any other issues
