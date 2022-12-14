# Neutron Testnet node setup guide

1. Update the system:
```
sudo apt update && sudo apt upgrade --yes
```
2. Install core tools:
```
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop net-tools lsof --yes
```
3. Install Go:
```
cd $HOME
wget -c -O go1.19.3.linux-amd64.tar.gz https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz && sudo rm go1.19.3.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
sudo cp $(which go) /usr/local/bin
go version
```
4. Clone the Neutron software and build the binary:
```
cd $HOME
git clone -b v0.1.1 https://github.com/neutron-org/neutron.git
cd neutron
make install
neutrond version
```
5. Initialize ``neutrond``:
```
neutrond init <your_moniker> --chain-id quark-1
```
6. Get the genesis file:
```
curl -s https://raw.githubusercontent.com/neutron-org/testnets/main/quark/genesis.json > ~/.neutrond/config/genesis.json
```
7. Add seeds and peers to ``$HOME/.neutrond/config/config.toml``:
```
seeds="e2c07e8e6e808fb36cca0fc580e31216772841df@seed-1.quark.ntrn.info:26656,c89b8316f006075ad6ae37349220dd56796b92fa@tenderseed.ccvalidators.com:29001"
peers="fcde59cbba742b86de260730d54daa60467c91a5@23.109.158.180:26656,5bdc67a5d5219aeda3c743e04fdcd72dcb150ba3@65.109.31.114:2480,3e9656706c94ae8b11596e53656c80cf092abe5d@65.21.250.197:46656,9cb73281f6774e42176905e548c134fc45bbe579@162.55.134.54:26656,27b07238cf2ea76acabd5d84d396d447d72aa01b@65.109.54.15:51656,f10c2cb08f82225a7ef2367709e8ac427d61d1b5@57.128.144.247:26656,20b4f9207cdc9d0310399f848f057621f7251846@222.106.187.13:40006,5019864f233cee00f3a6974d9ccaac65caa83807@162.19.31.150:55256,2144ce0e9e08b2a30c132fbde52101b753df788d@194.163.168.99:26656,b37326e3acd60d4e0ea2e3223d00633605fb4f79@nebula.p2p.org:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.neutrond/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.neutrond/config/config.toml
```
8. Set ``timeout_commit`` in ``config.toml`` and ``minimum-gas-prices`` in ``app.toml``:
```
sed -i.bak -e "s/^timeout_commit *=.*/timeout_commit = \"2s\"/;" ~/.neutrond/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001untrn\"/;" ~/.neutrond/config/app.toml
```
9. Configure the background service:
```
sudo tee /etc/systemd/system/neutrond.service > /dev/null <<EOF
[Unit]
Description=Neutrond Daemon
After=network.target

[Service]
User=$USER
ExecStart=$(which neutrond) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
10  Enable service and start your node:
```
sudo systemctl daemon-reload
sudo systemctl enable neutrond
sudo systemctl restart neutrond
journalctl -u neutrond -f -o cat
```
11. If you want to sync your full node quickly you can use state sync guide here: https://github.com/mediumwe11/neutron-testnet/blob/main/statesync.md
