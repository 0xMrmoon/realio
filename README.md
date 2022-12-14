# realio
#!/bin/bash
echo "skynodejs.net" 
echo "Skynodejs"
echo "github.com/0xMrmoon"
echo "Ty kjnodes & Nodeist for script"



sleep 2

# Variables Skynodes
RLO_WALLET=wallet
RLO=realio-networkd
RLO_ID=realionetwork_1110-2
RLO_PORT=17
RLO_FOLDER=.realio-network
RLO_FOLDER2=realio-network
RLO_VER=v0.6.2
RLO_REPO=https://github.com/realiotech/realio-network.git
RLO_GENESIS=https://raw.githubusercontent.com/realiotech/testnets/main/realionetwork_1110-2/genesis.json
RLO_ADDRBOOK=
RLO_MIN_GAS=0
RLO_DENOM=ario
RLO_SEEDS=aa194e9f9add331ee8ba15d2c3d8860c5a50713f@143.110.230.177:26656
RLO_PEERS=


sleep 1

echo "export RLO_WALLET=${RLO_WALLET}" >> $HOME/.bash_profile
echo "export RLO=${RLO}" >> $HOME/.bash_profile
echo "export RLO_ID=${RLO_ID}" >> $HOME/.bash_profile
echo "export RLO_PORT=${RLO_PORT}" >> $HOME/.bash_profile
echo "export RLO_FOLDER=${RLO_FOLDER}" >> $HOME/.bash_profile
echo "export RLO_FOLDER2=${RLO_FOLDER2}" >> $HOME/.bash_profile
echo "export RLO_VER=${RLO_VER}" >> $HOME/.bash_profile
echo "export RLO_REPO=${RLO_REPO}" >> $HOME/.bash_profile
echo "export RLO_GENESIS=${RLO_GENESIS}" >> $HOME/.bash_profile
echo "export RLO_PEERS=${RLO_PEERS}" >> $HOME/.bash_profile
echo "export RLO_SEED=${RLO_SEED}" >> $HOME/.bash_profile
echo "export RLO_MIN_GAS=${RLO_MIN_GAS}" >> $HOME/.bash_profile
echo "export RLO_DENOM=${RLO_DENOM}" >> $HOME/.bash_profile
source $HOME/.bash_profile

sleep 1

if [ ! $RLO_NODENAME ]; then
	read -p "Your Node Name: " RLO_NODENAME
	echo 'export RLO_NODENAME='$RLO_NODENAME >> $HOME/.bash_profile
fi

echo -e "NODE: \e[1m\e[32m$RLO_NODENAME\e[0m"
echo -e "WALLET: \e[1m\e[32m$RLO_WALLET\e[0m"
echo -e "CHAIN : \e[1m\e[32m$RLO_ID\e[0m"
echo -e "PORT : \e[1m\e[32m$RLO_PORT\e[0m"
echo '================================================='

sleep 2


# Upgrades
echo -e "\e[1m\e[32m1. ... \e[0m" && sleep 1
sudo apt update && sudo apt upgrade -y


# Install.Packages
echo -e "\e[1m\e[32m2. Packages... \e[0m" && sleep 1
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

# GO 
echo -e "\e[1m\e[32m1. GO ... \e[0m" && sleep 1
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version

sleep 1

# 
echo -e "\e[1m\e[32m1. REPO  \e[0m" && sleep 1
cd $HOME
git clone $RLO_REPO
cd $RLO_FOLDER2
git checkout $RLO_VER
make install

sleep 1

# 
echo -e "\e[1m\e[32m1.  \e[0m" && sleep 1
$RLO config chain-id $RLO_ID
$RLO config keyring-backend file
$RLO init $RLO_NODENAME --chain-id $RLO_ID

# ADDRBOOK 
wget $RLO_GENESIS -O $HOME/$RLO_FOLDER/config/genesis.json
wget $RLO_ADDRBOOK -O $HOME/$RLO_FOLDER/config/addrbook.json

# 
SEEDS="$RLO_SEEDS"
PEERS="$RLO_PEERS"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/$RLO_FOLDER/config/config.toml

sleep 1


# config pruning
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/$RLO_FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/$RLO_FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/$RLO_FOLDER/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/$RLO_FOLDER/config/app.toml


# ports
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${RLO_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${RLO_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${RLO_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${RLO_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${RLO_PORT}660\"%" $HOME/$RLO_FOLDER/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${RLO_PORT}317\"%; s%^address = \":8080\"%address = \":${RLO_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${RLO_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${RLO_PORT}091\"%" $HOME/$RLO_FOLDER/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${RLO_PORT}657\"%" $HOME/$RLO_FOLDER/config/client.toml

# 
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/$RLO_FOLDER/config/config.toml

# MINIMUM GAS 
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0$RLO_DENOM\"/" $HOME/$RLO_FOLDER/config/app.toml

# INDEXER 
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/$RLO_FOLDER/config/config.toml

# RESET 
$RLO tendermint unsafe-reset-all --home $HOME/$RLO_FOLDER

echo -e "\e[1m\e[32m4. ... \e[0m" && sleep 1
# create service
sudo tee /etc/systemd/system/$RLO.service > /dev/null <<EOF
[Unit]
Description=$RLO
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which $RLO) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF


# 
sudo systemctl daemon-reload
sudo systemctl enable $RLO
sudo systemctl restart $RLO

echo '555555555555555 Skynodes 5555555555555555555'
echo -e 'Check your logs: \e[1m\e[32mjournalctl -fu realio-networkd -o cat\e[0m'
echo -e "Check your sync : \e[1m\e[32mcurl -s localhost:${RLO_PORT}657/status | jq .result.sync_info\e[0m"

source $HOME/.bash_profile
