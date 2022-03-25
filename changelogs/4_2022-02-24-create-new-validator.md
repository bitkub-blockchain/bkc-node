# 4_2022-02-24-create-new-validator

## Topic
- Create new validator

## Detail
- Create Validate No.12-21

## Timeline
- 31 Mar 2022

## Step
1. Overwrite SSH public key
```sh
echo "<<Validator SSH Public Key>>" > ~/.ssh/authorized_keys
```
> Replace `<<Validator SSH Public Key>>` 

2. Switch Admin User
```sh
su -u
cd ~
```

3. Generate keystore password
```sh
curl https://random.justyy.workers.dev/api/random/?cached&n=64 -o /bkc-node/mainnet/password.sec
```

4. Create a new Account
```sh
geth --datadir /bkc-node/mainnet/data account new --password /bkc-node/mainnet/password.sec
```
> Copy Account address to ***Step. 5***

5. Edit `/etc/systemd/system/geth.service`
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

6. SystemD reload service and Start `geth.service`
```
systemctl daemon-reload && \
    systemctl start geth && \
    journalctl -fu geth
```