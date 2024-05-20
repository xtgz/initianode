[<img src='[assets\Initia-Banner.png](https://github.com/xtgz/initianode/assets/141474535/16c226ff-a2bd-45cb-8a4b-7a9661e3c5ac)' alt='banner' width= '99.9%'>]()

## Tutorial running node initia

### Hardware requirements
```py
- Memory: 16 GB RAM
- CPU: 4 cores
- Disk: 1 TB SSD
- Bandwidth: 1 Gbps
- Linux amd64 arm64 (Ubuntu LTS release)
```

### Setup Node
```bash
wget https://github.com/xtgz/initianode/raw/main/initia.sh && chmod +x *.sh && ./initia.sh && source $HOME/.bash_profile
```

## While waiting for the node setup to complete, create a new wallet/import 
### CREATE WALLET
```bash
initiad keys add $WALLET_NAME
```
# DON'T FORGET TO SAVE THE SEED PHRASE
### IMPORT WALLET
```bash
initiad keys add $WALLET_NAME --recover
```
### Request tokens from the faucet
-> <a href="https://faucet.testnet.initia.xyz/"><font size="4"><b><u>FAUCET</u></b></font></a> <-
### Check wallet balance
Make sure your node is fully synced unless it won't work.
```bash
initiad status | jq -r .sync_info
```
```bash
initiad q bank balances $(initiad keys show $WALLET_NAME -a) 
```

### Find out if the node is synchronized, if the result is false – it means the node is synchronized then you can create a validator
```bash
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```
### Create a validator
```bash
initiad tx mstaking create-validator \
  --amount=1000000uinit \
  --pubkey=$(initiad tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --from=$WALLET_NAME \
  --identity="" \
  --website="" \
  --details="Initia to the moon!" \
  --gas=2000000 --fees=300000uinit \
  -y
```
Don't forget to save `priv_validator_key.json` file located in $HOME/.initia/config/

## Useful commands
### Check node status 
```bash
initiad status | jq
```
### Query your validator
```bash
initiad q mstaking validator $(initiad keys show $WALLET_NAME --bech val -a) 
```
### Query missed blocks counter & jail details of your validator
```bash
initiad q slashing signing-info $(initiad tendermint show-validator)
```
### Unjail your validator 
```bash
initiad tx slashing unjail --from $WALLET_NAME --gas=2000000 --fees=300000uinit -y
```
### Delegate tokens to your validator 
edit <AMOUNT> with 1/10/100
```py 
initiad tx mstaking delegate $(initiad keys show $WALLET_NAME --bech val -a)  <AMOUNT>uinit --from $WALLET_NAME --gas=2000000 --fees=300000uinit -y
```
### Get your p2p peer address
```bash
initiad status | jq -r '"\(.NodeInfo.id)@\(.NodeInfo.listen_addr)"'
```
### Edit your validator
```bash 
initiad tx mstaking edit-validator --website="<WEBSITE>" --details="<DESCRIPTION>" --moniker="<NEW_MONIKER>" --from=$WALLET_NAME --gas=2000000 --fees=300000uinit -y
```
### Send tokens between wallets 
```bash
initiad tx bank send $WALLET_NAME <TO_WALLET> <AMOUNT>uinit --gas=2000000 --fees=300000uinit -y
```
### Query your wallet balance 
```bash
initiad q bank balances $WALLET_NAME
```
### Monitor server load
```bash 
sudo apt update
sudo apt install htop -y
htop
```
### Query active validators
```bash
initiad q mstaking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.voting_power + " - " + .description.moniker' \
| sort -gr | nl
```
### Query inactive validators
```bash
initiad q mstaking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.voting_power + " - " + .description.moniker' \
| sort -gr | nl
```
### Check logs of the node
```bash
sudo journalctl -u initiad -f -o cat
```
### Restart the node
```bash
sudo systemctl restart initiad
```
### Stop the node
```bash
sudo systemctl stop initiad
```
### Delete the node from the server
```bash
# !!! IF YOU HAVE CREATED A VALIDATOR, MAKE SURE TO BACKUP `priv_validator_key.json` file located in $HOME/.initia/config/ 
sudo systemctl stop initiad
sudo systemctl disable initiad
sudo rm /etc/systemd/system/initiad.service
rm -rf $HOME/.initia
sudo rm /usr/local/bin/initiad
```
### Example gRPC usage
```bash
wget https://github.com/fullstorydev/grpcurl/releases/download/v1.7.0/grpcurl_1.7.0_linux_x86_64.tar.gz
tar -xvf grpcurl_1.7.0_linux_x86_64.tar.gz
chmod +x grpcurl
./grpcurl  -plaintext  localhost:$GRPC_PORT list
### MAKE SURE gRPC is enabled in app.toml
# grep -A 3 "\[grpc\]" $HOME/.initia/config/app.toml
```
### Example REST API query
```bash
curl localhost:$API_PORT/cosmos/mstaking/v1beta1/validators
### MAKE SURE API is enabled in app.toml
# grep -A 3 "\[api\]" $HOME/.initia/config/app.toml
```
