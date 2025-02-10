# Running Orbs v2 on your own infrastructure

## Prerequisites

* A Linux machine with Docker Swarm
* Boyar [binary download](https://github.com/orbs-network/boyarin/releases)
* Connection to Ethereum node (`<ETHEREUM_ENDPOINT>` url)
* Directories `/var/efs`, `/var/efs/boyar-status` and `/var/efs/boyar-logs` should be created on the machine and have enough space (separate filesystem is recommended, you can use NFS if you like)
* Orbs node address and orbs node private key. You can learn about key generation [here](https://github.com/orbs-network/validator-instructions/blob/master/public/orbs-public-blockchain.md#allocate-orbs-node-address-and-private-key).

## JSON configuration files

### `mgmt.json`

You need to replace `<ETHEREUM_ENDPOINT>` with a URL of a real Ethereum node (you can use [Infura free tier](https://github.com/orbs-network/validator-instructions/blob/master/public/infura-setup-free.md) or your own node).

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
        "Tag": "bootstrap",
        "Pull": true
      },
      "Config": {
        "BootstrapMode": true,
        "EthereumEndpoint": "<ETHEREUM_ENDPOINT>",
        "DockerNamespace": "orbsnetwork"
      }
    }
  }
}
```

### `keys.json`

```json
{
  "node-address": "<NODE_ADDRESS_WITH_NO_LEADING_0x>",
  "node-private-key": "<NODE_PRIVATE_KEY_WITN_NO_LEADING_0x>"
}
```

## Running Boyar

[Boyar](https://github.com/orbs-network/boyarin) is a bootstrapping software that we use to provision virtual chains. First, it starts a management service container that pulls info from Ethereum, including the list of the chains, consensus committees, and so on, and then uses this information to run other containers.

```bash
mkdir -p /var/efs/ /var/efs/boyar-status /var/efs/boyar-logs
boyar --keys ./keys.json --management-config ./mgmt.json --log /var/efs/boyar-logs/current --status /var/efs/boyar-status/status.json --bootstrap-reset-timeout 30m --auto-update --shutdown-after-update
```

It is recommended to use some external process manager, for exampls, [Supervisord](https://github.com/Supervisor/supervisor).

### Supervisord config example

Place in `/opt/orbs/boyar.sh`

```bash
#!/bin/bash

trap "kill -- -$$" EXIT

multilog_err=1
multilog_cmd="multilog s16777215 n32 /var/efs/boyar-logs/"

while [[ "$multilog_err" -ne "0" ]]; do
    sleep 1
    echo "boyar logging pre checks..." | $multilog_cmd
    multilog_err=$?
done

echo "Running boyar..."

# FIXME: please set correct /path/to/keys.json
# FIXME: please set correct /path/to/mgmt.json

exec /usr/bin/boyar --keys /path/to/keys.json --max-reload-time-delay 0m --bootstrap-reset-timeout 30m --status /var/efs/boyar-status/status.json  --management-config /path/to/management-config.json --auto-update --shutdown-after-update 2>&1 | $multilog_cmd
```

Place in `/etc/supervisor/conf.d/boyar.conf`

```ini
[program:boyar]
command=/opt/orbs/boyar.sh
autostart=true
autorestart=true
environment=HOME="/root"
stdout_logfile=/var/efs/boyar-logs/supervisor.stdout
redirect_stderr=true
stdout_logfile_maxbytes=10MB
```

### Systemd unit config example

```ini
[Unit]
Description=Boyar service
After=network.target

[Service]
# FIXME: please set correct /path/to/keys.json
# FIXME: please set correct /path/to/mgmt.json
ExecStart=/usr/bin/boyar --keys /path/to/keys.json --management-config /path/to/mgmt.json --log /var/efs/boyar-logs/current --status /var/efs/boyar-status/status.json --auto-update --shutdown-after-update --bootstrap-reset-timeout 30m
Restart=always
KillSignal=SIGHUP

[Install]
WantedBy=default.target
```

## Verifying your node's health

From your Linux machine you can access these URLs:
* http://localhost/services/boyar/status
* http://localhost/services/boyar/logs
* http://localhost/services/management-service/status 
* http://localhost/services/management-service/logs

You can also replace `localhost` with your node IP address.

## Register your Guardian

To register on the network, go to [Guardian Registration](https://guardians.orbs.network/registration)
(Requires Metamask)

## Polygon network support

**New Guardians** Please Register on both Ethereum and Polygon networks, using the following link
[Guardian Registration](https://guardians.orbs.network/registration)

Ethereum regisration is mandatory in order to get certified.

Changing network is available on top right near the language selector.

**Congratulations!**