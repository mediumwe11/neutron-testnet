# Neutron Testnet state sync guide

1. Stop Neutron background service and reset database:
```
sudo systemctl stop neutrond
neutrond tendermint unsafe-reset-all --home $HOME/.neutrond
```
2. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
PEER="c32c97f026a9f707362a86503ed96925f69e2768@62.171.157.1:26656"
SNAP="http://62.171.157.1:26657"
LATEST_HEIGHT=$(curl -s $SNAP/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP,$SNAP\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.neutrond/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEER\"/" $HOME/.neutrond/config/config.toml
```
3. Run Neutron background service:
```
sudo systemctl start neutrond
sudo journalctl -u neutrond -f -o cat
```
