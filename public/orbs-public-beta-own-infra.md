# Running Orbs v2 on your own infrastructure

## Prerequisites

* A Linux machine with Docker Swarm
* Boyar v1.1.1 [binary download](https://s3.amazonaws.com/orbs-network-releases/infrastructure/boyar/boyar-v1.1.1.bin)
* Connection to Ethereum node (`$ETHEREUM_ENDPOINT` url)
* Directories `/var/efs`, `/var/efs/boyar-status` and `/var/efs/boyar-logs` should be created on the machine and have enough space (separate filesystem is recommended, you can use NFS if you like)

## JSON configuration files

### `mgmt.json`

You need to replace `$ETHEREUM_ENDPOINT` with a URL of a real Ethereum node (you can use Infura free tier or your own node).

```json
{
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
        "Tag": "v1.1.1",
        "Pull": true
      },
      "Config": {
        "BootstrapMode": true,
        "EthereumEndpoint": "$ETHEREUM_ENDPOINT",
        "DockerNamespace": "orbsnetwork"
      }
    }
  }
}
```

### `keys.json`

You can use [Orbs key generator](https://github.com/orbs-network/orbs-key-generator) with `orbs-key-generator node` command.

```json
{
  "node-address": "$NODE_ADDRESS_WITH_NO_LEADING_0x",
  "node-private-key": "$NODE_PRIVATE_KEY_WITN_NO_LEADING_0x"
}
```

## Running Boyar

[Boyar](https://github.com/orbs-network/boyarin) is a bootstrapping software that we use to provision virtual chains. First, it starts a management service container that pulls info from Ethereum, including the list of the chains, consensus committees, and so on, and then uses this information to run other containers.

```bash
mkdir -p /var/efs/ /var/efs/boyar-status /var/efs/boyar-logs
boyar --keys ./keys.json --management-config ./mgmt.json --log /var/efs/boyar-logs/current --status /var/efs/boyar-status/status.json
```

## Verifying your node's health

From your Linux machine you can access these URLs:
* http://localhost/services/boyar/status
* http://localhost/services/boyar/logs
* http://localhost/services/management-service/status 
* http://localhost/services/management-service/logs

You can also replace `localhost` with your node IP address.
