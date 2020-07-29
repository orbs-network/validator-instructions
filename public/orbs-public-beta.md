# Setting up an Orbs Public Blockchain Node using Polygon CLI

This step-by-step guide will walk you through creating a new node and connecting it to an existing Orbs network.

![](../diagram.png)

## Prerequisites

To complete this guide you will need the following set up:

- Mac or Linux machine
- An Ethereum Endpoint URL. For the beta program you may use an [Infura free tier account](infura-setup-free.md).
- **A clean, new AWS account with admin programmatic access.**
- AWS CLI - Install using `brew install awscli` 
- An Orbs Node Address. For details see [below](#allocate-orbs-node-address-and-private-key)
- A Guardian (Wallet) Address
- Metamask installed 
- An AWS credentials profile set correctly:
  
  For a simple setup of the "default" profile run `aws configure`. For additional options see [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
  
  Admin privileges for your AWS account are requried to be setup in the specified profile. 
- [Node.js](https://nodejs.org/en/) version 8 or above
  
  Use `brew install node` to get it installed

- [Terraform](https://www.terraform.io/downloads.html)
  
  Polygon currently supports the latest Terraform (v0.12.x) and have been tested on version v0.12.23
  for most scenarios. We recommend installating this specific version to have a fail-proof experience.

  For macOS:

  Download Terraform by running this command in your Terminal:
  `wget https://releases.hashicorp.com/terraform/0.12.23/terraform_0.12.23_darwin_amd64.zip`

  Then unzip it:
  `unzip terraform_0.12.23_darwin_amd64.zip`

  Grant it executable permissions:
  `chmod +x terraform && sudo mv terraform /usr/local/bin/`

  Make sure it's installed by typing in:
  `terraform --version` - you should see the version printed out

  For Linux:

  Download Terraform by running this command in your Terminal:
  `curl -O https://releases.hashicorp.com/terraform/0.12.23/terraform_0.12.23_darwin_amd64.zip`

  Then unzip it: (If unzip is not installed, install it by running `apt-get install unzip`)
  `unzip terraform_0.12.23_darwin_amd64.zip`

  Grant it executable permissions:
  `chmod +x terraform && sudo mv terraform /usr/local/bin/`

  Make sure it's installed by typing in:
  `terraform --version` - you should see the version printed out

### Generate SSH public and private keys

If you already have an ssh public key to use with your Orbs Node instaces, you may skip this step.

We require a valid public/private keys to run our deployment scripts and set up the EC2 resources. The key file should remain secret with the exception of feeding it to the configuration during setup. (providing the path for the pub file in the `orbs-node.json` setup file as described below)

The generated key should __not__ have a passphrase.
It is okay to generate a key by any means, such as based on the following tutorial by [GitHub](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

The gist of creating such a key is running:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    
* -C "your_email@example.com" is an optional comment

### Allocate a static IP on Amazon

The Orbs node that you provision must have a static IPs (Elastic IP) in order to communicate with the network.

- Go to your AWS Console
- Pick a region for your node deployment (for example `ca-central-1`)
- Allocate 1 Elastic IP

The Node IP address and region will later be used in the node configuration file.
The IP will also be required during the [Guardian Registration phase](#register-your-guardian).


### Allocate Orbs Node address and private key
The Orbs Node address is a standard Ethereum Address. This address is used for (1) signing blocks on Orbs blockchain, and for (2) sending transactions to Orbs PoS smart contracts on Ethereum. Therefore, the Orbs node stores and uses the node address's private key. As such, the private key should be different from the Guardian private key.

During normal operation, Orbs node automatically sends transactions to the PoS smart contracts on Ethereum (e.g. to execute reward distribution or to signal that it is in sync with the network and ready to enter a committee). The Orbs Node address should hold enough ETH to fund the gas for transactions sent to the PoS contracts. It is the Operators responsibility to periodically verify the Orbs Node Address has a balance of 0.5 - 1 ETH.

Orbs Node address and private key should be generated in a secure fashion and the private key is provided during node deployment, see below.

The Orbs Node address is also provided during the [Guardian Registration phase](#register-your-guardian).

The Orbs address may be modified, by updating the registration, prior to a node address change, the node should be teared-down and then redeployed.

The Orbs Node address, and its private key should be securely stored.

### Install Polygon via NPM

To install Polygon run

    npm install -g @orbs-network/polygon

If you have previously installed Polygon and you are performing a new deploy, we recommend updating it by running `npm update -g @orbs-network/polygon`

### Create a dedicated deployment folder

Create a dedicated folder for your deployment. For example, "orbs-v2-beta".
This folder will hold config files, logs and the artifacts for future needs (such as 'destroy'). 

__Backup this folder upon completion of the instructions and DO NOT delete it.__

Change your working path to the deployment folder and proceed with the instructions below.

### Configure the boilerplate JSON file

Create a config file `orbs-node-beta.json` for your deployment in .

The content of the `orbs-node-beta.json` should be:

    {
        "name": "<orbs node name>",
        "awsProfile": "<aws profile>",
        "sshPublicKey": "<ssh access public key file>",
        "orbsAddress": "<orbs node ethereum address>",
        "region": "<aws region>",
        "publicIp": "<node ip>",
        "nodeSize": "r5.large",
        "nodeCount": 0,
        "cachePath": "./_terraform_beta",
        "incomingSshCidrBlocks": ["<ssh source cird block>",...],
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
                        "EthereumEndpoint": "<ethereum endpoint url>",
                        "DockerNamespace":"orbsnetwork"
                    }
                }
            }
        }
    }

Where:
* `<orbs node name>` - Will be used with some AWS resource names as prefix or suffix to uniquly identify node resources. Must be S3 bucket name complient (e.g. no _...)
* `<aws profile>` - use "default", or, if you are using AWS profiles with more than one AWS account: an AWS profile name with pre-configured AWS credentials  and account for your account.
* `<ssh access public key file>` - The SSH public key filename (if omitted, defaults to `~/.ssh/id_rsa.pub`)
* `<orbs node ethereum address>` - The Orbs Node Ethereum address, EIP-55 compliant with checksum capitalization __but without the leading '0x'__
* `<aws region>` - An AWS region name. The new node will be provisioned in this region. e.g. `ca-central-1`
* `<node ip>` - A static IP address where this Orbs Node will be reachable. Must be a valid Elastic Ip allocated and unattached in the selected region.
* `<ethereum endpoint url>` - a url to a working and synced Ethereum node. 
* `<ssh source cird block>` - One or more CIDR blocks which will be granted access to ssh from into the node. You will still need the public key to connect - it is required only in cases of troubleshooting. The format is standard CIDR. Any IP not in the range will not be able to SSH to the node, even if it has the SSH key file. To allow ssh access from any ip use `"incomingSshCidrBlocks": ["0.0.0.0/0"],` 

### Deploy the Node using Polygon CLI 

Temporarily store the Orbs Node private key in a file `<orbs private key file>` __without the leading 0x__.
Note, the private key should be a string of 64 characters, e.g. `f5f83Ee70a85fFF2exxxxxxxxxxxxxxxxxxxxxxxxxxx334932F34C8D629165Ed`.

To provision the resources required for the node:

    polygon create -f orbs-node-beta.json --orbs-private-key $(cat <orbs private key file>)

After ensuring the private key is stored in a secure place, delete the temporary private key file `<orbs private key file>`

During creation of the node, Terraform cache files will be created in the folder defined in `cachePath` and should be backed up to allow proper cleanup at a later stage.

If needed, the command to remove all resources provisioned for the node is:
           
    polygon destroy -f orbs-node-beta.json
    
### IMPORTANT! ###

After deployment make sure to backup and securely store the deployment folder data, including:

* __Orbs keys__ (`<orbs private key file>` and any other credentials you used and configured, such as SSH keys)
* _terraform cache folder - is required to destroy or redeploy the node
* The `orbs-node-beta.json` file

Do not leave sensitive data such as the `<orbs private key file>` in a non secure disk location

### Update Orbs Node configuration

__Any modification to your configuration file MUST be done while the node is DOWN.__

To update your node configuration
1. run polygon destroy (e.g. `polygon destroy -f orbs-node-beta.json`)
1. update your configuration JSON file as required
1. run [polygon create](#deploy-the-node-using-polygon-cli)

### Register your Guardian

In order to register on the network, navigate to [Guardian Registration](https://guardians.orbs.network/registration)
(Requires Metamask)

### Verify your Node is deployed correctly

Check that all the services started without errors
```
http://<node ip>/services/boyar/status
```

Check your node's status in the PoS network (readings are 10 minutes dalayed for finality)
```
http://<node ip>/services/management-service/status
```


__Congratulations!__

## Troubleshooting

1. If you get an Terraform error that your IP does not exist, check whether the combination of ip and region is correct in the node configuration file (`orbs-node-beta.json`)

2. If the metrics page does not respond, it could be that the Ethereum node did not finish syncing - this takes several hours.

3. If you're having Terraform errors, check your clock

4. Contact Orbs for any other issues
