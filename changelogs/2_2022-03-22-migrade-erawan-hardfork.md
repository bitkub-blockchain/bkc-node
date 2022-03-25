# Migration Erawan Hardfork

## Topic
- Migration Erawan Hardfork

## Detail
1. Upgrade `geth` support erawan hardfork
2. Migrade bash script to use `config.toml`
1. Update cache size `8Gb` to `18Gb`
2. Move `nodekey` and `passwod.sec` to `/bkc-node/mainnet`

## Timeline
- 28-29 Mar 2022

## Step
1. Switch Admin User
```sh
su -u
cd ~
```

2. Stop Geth Service
```sh
systemctl stop geth
```

3. Delete go-ethereum old version
```sh
rm -rf /use/bin/geth
```

4. Download and Install go-ethereum bkc version
```sh
curl -L -O https://github.com/bitkub-blockchain/bkc/releases/latest/download/geth-linux-x86_64.tar.gz && \
    tar -xf geth-linux-x86_64.tar.gz -C /usr/bin
```

5. Download config.toml and genesis.json
```sh
curl https://raw.githubusercontent.com/bitkub-blockchain/bkc-node/internal/mainnet/genesis.json -o /bkc-node/mainnet/genesis.json && \
    curl https://raw.githubusercontent.com/bitkub-blockchain/bkc-node/internal/mainnet/config.toml -o /bkc-node/mainnet/config.toml
``` 

6. Reinitail genesis.json
```
geth --datadir /bkc-node/mainnet/data init/bkc-node/mainnet/genesis.json
```

7. Redesign folder stucture
```
# Move password file
cp /root/bkc-node/bkc-mainnet/node0/password.sec /bkc-node/mainnet/password.sec
# Move nodekey file
cp /bkc-node/mainnet/data/geth/nodekey /bkc-node/mainnet/validator.key
```

8. Edit `/etc/systemd/system/geth.service`
```toml
[Unit]
Description=Geth Validator
After=network.target auditd.service
Wants=network.target

[Service]
Type=simple
Environment="NODE_ADDRESS=<<Your account address>>"
Environment="REWARD_ADDRESS=<<Reward pool address>>"
ExecStartPre=/usr/bin/bash -c "/usr/bin/systemctl set-environment NODE_NAME=$(hostname) && /usr/bin/systemctl set-environment IP=$(curl ifconfig.me)"
ExecStart=/usr/bin/geth --config /bkc-node/mainnet/config.toml --cache 18000 --identity "${NODE_NAME}" --ethstats "${NODE_NAME}:s3cr3t@49.0.194.80:4000" --nodekey /bkc-node/mainnet/validator.key --mine --miner.etherbase ${REWARD_ADDRESS} --unlock ${NODE_ADDRESS} --allow-insecure-unlock --password /bkc-node/mainnet/password.sec
KillMode=process
KillSignal=SIGINT
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=default.target 
RequiredBy=swarm.service
Alias=geth.service
```
> Replace `<<Your account address>>` and `<<Reward pool address>>` form [Sheet](https://docs.google.com/spreadsheets/d/1QPSP8GYxQJJSaSkdT6H1tXft-_CGFsz7RzFDWmJbX98/edit#gid=1196957597) 


9. SystemD reload service and Start `geth.service`
```
systemctl daemon-reload && \
    systemctl start geth && \
    journalctl -fu geth
```

## Reference
- https://www.flatcar.org/docs/latest/setup/systemd/environment-variables/
- https://docs.bitkubchain.org/nodes/setup-instructions-for-bitkubchain-node/validator-node
- https://docs.binance.org/smart-chain/validator/guideline.html