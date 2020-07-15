# Setting up an Orbs Public Blockchain Node using Polygon CLI

This step-by-step guide will walk you through creating a new node and connecting it to an existing Orbs network.

![](../diagram.png)

## Prerequisites

To complete this guide you will need the following set up:

- Mac or Linux machine
- An SSH public key (by default we use `~/.ssh/id_rsa.pub`). We go into details on how to generate it below
- **A clean, new AWS account with admin programmatic access.**
- AWS CLI
  
  Use `brew install awscli` to get it installed
- An AWS credentials profile set correctly:
  
  See more [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
  
  We require the `aws_access_key_id` and `aws_secret_access_key` of an admin account for our Terraform script to execute correctly 
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

- [Orbs Key Generator](https://www.github.com/orbs-network/orbs-key-generator)

  Use `brew install orbs-network/devtools/orbs-key-generator` to get it installed (requires a Mac)

### Generating SSH public and private keys

We require a valid public/private keys to run our deployment scripts and set up the EC2 resources. The key file should remain secret with the exception of feeding it to the configuration during setup. (providing the path for the pub file in the `orbs-node.json` setup file as described below)

The generated key should __not__ have a passphrase.
It is okay to generate a key by any means, such as based on the following tutorial by [GitHub](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

The gist of creating such a key is running:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

### Allocating an IP on Amazon

The Orbs node that you provision must have a static IPs in order to communicate with the network.

- Go to your AWS Console
- Pick a region (for example `ca-central-1`)
- Allocate 1 Elastic IPs

That IP address and region will later be used in the node configuration file.


### Generating Orbs addresses

An Orbs node is identified by a public key and any action of the node should be signed with the corresponding private key. 
These keys should be generated in a secure fashion and the private key should be securely stored. 

We require an Orbs private key and an Orbs address. These can be generated using the [Orbs key generator](https://github.com/orbs-network/orbs-key-generator) by running `orbs-key-generator node`

The output of the key generator should be securely stored and used in the `orbs-node.json` configuration file and node deployment command as explained below. You will need the `NodeAddress` and `NodePrivateKey` later on __without the leading 0x__.

### Install Polygon via NPM

To install Polygon run

    npm install -g @orbs-network/orbs-polygon

If you have previously installed Polygon and you are performing a new deploy, we recommend updating it by running `npm update -g @orbs-network/orbs-polygon`

### Create a dedicated folder

Create a dedicated folder "orbs-v2-beta"
This folder will store the logs and the artifacts for future needs (such as 'destroy'). 
Change your path into the folder and proceed with the instructions there.
Backup this folder upon completion of the instructions and do not delete it.
--- DO NOT DELETE THIS FOLDER ---

### Configure the boilerplate JSON file

The thing to do next is to create the `orbs-beta-node.json` file and configure it as required for the new node.

The content of the `orbs-beta-node.json` should be:

    {
        "name": "$VALIDATOR_NAME-orbs-beta",
        "awsProfile": "default",
        "sshPublicKey": "$LOCATION_TO_PUB_FILE",
        "orbsAddress": "$ORBS_PUBLIC_NODE_ADDRESS",
        "publicIp": "$NODE_AWS_IP",
        "region": "$NODE_AWS_REGION",
        "nodeSize": "r5.large",
        "nodeCount": 0,
        "bootstrapUrl": "TBD_FIX_ME_KIRILL",
        "ethereumChain": "mainnet",
        "ethereumTopologyContractAddress": "TBD_FIX_ME_KIRILL",
        "cachePath": "./_terraform_beta",
        "ethereumEndpoint": "$ETHEREUM_NODE_ADDRESS",
        "incomingSshCidrBlocks": ["$YOUR_OFFICE_IP/32"]
    }

You will need:
* $VALIDATOR_NAME-orbs-beta - Name for your Validator name, such as a company name or brand name.
* $LOCATION_TO_PUB_FILE - The SSH public and private key file path (the generated pub file)
* $ORBS_PUBLIC_NODE_ADDRESS - The Orbs node address (from the Orbs key generator - __without the leading 0x__)
* $NODE_AWS_IP - The IP address (from AWS)
* $NODE_AWS_REGION - The AWS region (from AWS)
* $ETHEREUM_NODE_ADDRESS - used to configure an external Ethereum node. If you have your own synced Ethereum node, you can use it as a value for ethereumEndpoint. Alternatively, you can use "http://eth.orbs.com" , which we provide for your convenience (configure it by writing "ethereumEndpoint": "http://eth.orbs.com"). Our long term goal is to use the Ethereum node that is internal to the Orbs node. 
* $YOUR_OFFICE_IP - This is the IP address/range that we will grant access to for ssh connections to the node. You will still need the public key to connect - it is required only in cases of troubleshooting. The format is standard CIDR so a range may be provided by changing the mask. Any IP not in the range will not be able to SSH to the node, even if it has the SSH key file.

Other parameters (no need to change them):

The `cachePath` configuration tells Polygon where to store the terraform installation meta-data created during the deploy stage. It is required in cases where you wish to remove the node from AWS. You should store these files and back them up so you can run maintenance if required.

The `awsProfile` configuration can be changed if you are using multiple AWS configurations and want a specific one to be applied.

## Some warning as to updating your node configuration JSON file
While the configuration is quite easily changeable, please do remember that any modification to your configuration file MUST be done while the node is DOWN.
For example: if you decide you want to set a backend syncing using the Terraform state syncing to S3 (Just as an example). 
You should do the following to perform the change:
* run polygon destroy
* update your configuration JSON file to reflect your changes (for example add `"backend": true`)
* run polygon create

### Run Polygon CLI to deploy the node

To avoid having the orbs node private key as part of your command history, we recommend creating a file called `orbs-private-key.txt` and put the orbs node private key inside it, __without the leading 0x__.
That key was generated by the key generator and should be in a hex string of size 64 characters, like `f5f83Ee70a85fFF2exxxxxxxxxxxxxxxxxxxxxxxxxxx334932F34C8D629165Ed`.

To provision the resources required for the node:

    polygon create -f orbs-node-beta.json --orbs-private-key $(cat path/to/orbs-private-key.txt)

Terraform files corresponding to nodes can be found in the folder defined in `cachePath` and should be backed up.

If needed, the command to remove all resources provisioned for the node is:
           
    polygon destroy -f orbs-node-beta.json
    
### IMPORTANT! ###
After deployment **make sure to backup and securely store** - 
1. __Orbs keys__ (and any other credentials you used and configured, such as SSH keys)
2. __`_terraform` folder contents__ - these are required to destroy or redeploy the node
3. The __`orbs-node.json`__ file

### Registering to the Orbs public beta

In order to register on the network, pleaseuse the tool at [Guardian Registration](https://guardians.orbs.network/registration)

Contact Orbs after registration is done.

### What happens after deployment

Once the deployment finishes, the node will start the various Orbs node services and, once it is part of the topology, it will start syncing with other nodes.

### How to inspect the network health
[TBD_FIX_ME_KIRILL]

__Congratulations!__

## Troubleshooting
[TBD_FIX_ME_KIRILL]

1. If you get an Terraform error that your IP does not exist, check whether the combination of ip and region is correct in the node configuration file (`orbs-node-beta.json`)

2. If the metrics page does not respond, it could be that the Ethereum node did not finish syncing - this takes several hours.

3. If you are having trouble with Ethereum node, add `"ethereumEndpoint": "http://eth.orbs.com"` to your `orbs-node-beta.json` and redeploy the node (`polygon destroy` and then `polygon create` as usual). If you have your own synced Ethereum node, you can use it as a value for `ethereumEndpoint`. We only provide `eth.orbs.com` for your convenience. Our long term goal is to use the Ethereum node that belongs to the Orbs node.

4. If you're having Terraform errors, check your clock [TBD_FIX_ME_KIRILL]

5. Contact Orbs for any other issues
