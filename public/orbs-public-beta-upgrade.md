# Validator upgrade instructions

Instructions are valid for 8 November 2020.

## AWS

1) upgrade polygon to the latest version: `npm install -g @orbs-network/polygon@1.28.0`
2) add `"boyarAutoUpdate": true` to your `node.json` file
3) destroy the node (no additional actions necessary): `polygon destroy -f node.json`
4) provision a new node: `polygon create -f node.json --orbs-private-key $(cat path/to/orbs-private-key.txt)`

Please note that `boyarAutoUpdate` will allow automatic update of our orchestrator software, Boyar, that manages virtual chains and other node services.

## Own infrastructure

1) upgrade Boyar to the latest version: https://github.com/orbs-network/boyarin/releases
2) add `--auto-update --shutdown-after-update --bootstrap-reset-timeout 30m` to your boyar parameters
3) run boyar: `boyar --keys ./keys.json --management-config ./mgmt.json --log /var/efs/boyar-logs/current --status /var/efs/boyar-status/status.json --bootstrap-reset-timeout 30m --auto-update --shutdown-after-update`

Please note that `boyarAutoUpdate` will allow automatic update of our orchestrator software, Boyar, that manages virtual chains and other node services.

In this configuration, after the update it will shut itself down, so it is advisable to use an external process manager like Supervisord or turn the autoupdate off.
