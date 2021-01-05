# Updating Docker

Currently, the nodes provisioned before December 15th 2020 are running Docker version 19.03, which has memory consumption issues.

Docker 20.10 addresses these issues and memory consumption goes down significantly, sometimes by 1Gb RAM and more.

There are two available ways to upgrade.

## Upgrade via SSH

SSH into the node and run the following command:

```bash
sudo apt-get install -y docker-ce=5:20.10.1~3-0~ubuntu-bionic
sudo service docker restart
```

Check Docker version:

```bash
docker info
```

## Upgrade by recreating the node

Destroy the node first (the data will not be affected).

```bash
polygon destroy -f node.json
```

Create the node again:

```bash
polygon create -f node.json --orbs-private-key $(cat <orbs private key file>)
```

The update will happen automatically because the latest version of Docker will be installed by default.
