# REPUBLIC-AI
<img width="1500" height="500" alt="image" src="https://github.com/user-attachments/assets/1512180b-63b6-4c1e-a07f-34700a4ec5cf" />

# System Requirements


| Component       | Minimum              | Recommended |
|------------------|----------------------------|----------------------------|
| **CPU**          | 4+ | 8++ |
| **RAM**          | 16+  | 16++ GB                   |
| **Disk**      | 500GB NVME SSD| 500+ NVME GB SDD                   |
| **UBUNTU**      | UBUNTU 24.04 | UBUNTU 24.04 |

#  Update Server
```
sudo apt update -y && sudo apt upgrade -y
```
# Install Required Packages
```
sudo apt install htop ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev tmux iptables curl nvme-cli git wget make jq libleveldb-dev build-essential pkg-config ncdu tar clang bsdmainutils lsb-release libssl-dev libreadline-dev libffi-dev gcc screen file nano btop unzip lz4 -y
```
# Install Go & Cosmovisor
```
cd $HOME
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz


echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile


go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```
# Environment Variables
Choose a wallet name and a custom port prefix (example: 43)
```
echo "export REPUBLIC_WALLET=wallet" >> ~/.bash_profile
echo "export REPUBLIC_PORT=43" >> ~/.bash_profile
source ~/.bash_profile
```
# Download Republic Binary
```
VERSION=v0.1.0


mkdir -p ~/.republicd/cosmovisor/genesis/bin
curl -L https://media.githubusercontent.com/media/RepublicAI/networks/main/testnet/releases/${VERSION}/republicd-linux-amd64 -o republicd
chmod +x republicd
mv republicd ~/.republicd/cosmovisor/genesis/bin/


ln -s ~/.republicd/cosmovisor/genesis ~/.republicd/cosmovisor/current
sudo ln -s ~/.republicd/cosmovisor/genesis/bin/republicd /usr/local/bin/republicd


republicd version
```
# Initialize Node

Replace YOUR_NODE_NAME with your validator name (use only English characters)
```
republicd init YOUR_NODE_NAME --chain-id raitestnet_77701-1 --home ~/.republicd
```
# Download Genesis File
```
curl -s https://raw.githubusercontent.com/RepublicAI/networks/main/testnet/genesis.json > ~/.republicd/config/genesis.json
```
# Configure Ports
config.toml
```
sed -i.bak -e "s%:26658%:${REPUBLIC_PORT}658%g;
s%:26657%:${REPUBLIC_PORT}657%g;
s%:6060%:${REPUBLIC_PORT}060%g;
s%:26656%:${REPUBLIC_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${REPUBLIC_PORT}656\"%;
s%:26660%:${REPUBLIC_PORT}660%g" ~/.republicd/config/config.toml
```
app.toml
```
sed -i.bak -e "s%:1317%:${REPUBLIC_PORT}317%g;
s%:8080%:${REPUBLIC_PORT}080%g;
s%:9090%:${REPUBLIC_PORT}090%g;
s%:9091%:${REPUBLIC_PORT}091%g;
s%:8545%:${REPUBLIC_PORT}545%g;
s%:8546%:${REPUBLIC_PORT}546%g;
s%:6065%:${REPUBLIC_PORT}065%g" ~/.republicd/config/app.toml
```
# Configure Peers
```
SEEDS=""
PEERS="<USE_LATEST_PEER_LIST>"


sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.republicd/config/config.toml
```
# Pruning Configuration
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" ~/.republicd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" ~/.republicd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" ~/.republicd/config/app.toml
```
# Disable Indexer (Optional)
```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" ~/.republicd/config/config.toml
```
# Minimum Gas Price
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "250000000arai"|g' ~/.republicd/config/app.toml
```
# Create systemd Service
```
udo tee /etc/systemd/system/republicd.service > /dev/null <<EOF
[Unit]
Description=Republic AI Node
After=network-online.target


[Service]
User=$USER
Environment="DAEMON_NAME=republicd"
Environment="DAEMON_HOME=$HOME/.republicd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
ExecStart=$HOME/go/bin/cosmovisor run start --home $HOME/.republicd --chain-id raitestnet_77701-1
Restart=always
RestartSec=3
LimitNOFILE=65535


[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable republicd
sudo systemctl start republicd
```
# Logs 
```
journalctl -u republicd -f
```
# Wallet
```
republicd keys add $REPUBLIC_WALLET
```
# Validator Creation
```
PUBKEY=$(jq -r '.pub_key.value' ~/.republicd/config/priv_validator_key.json)
{
"pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"PUBKEY"},
"amount": "20000000000000000000arai",
"moniker": "your-validator-name",
"identity": "",
"website": "",
"security": "email",
"details": "validator description",
"commission-rate": "0.05",
"commission-max-rate": "0.15",
"commission-max-change-rate": "0.02",
"min-self-delegation": "1"
}
```
```
republicd tx staking create-validator validator.json \
--from $REPUBLIC_WALLET \
--chain-id raitestnet_77701-1 \
--gas auto \
--gas-adjustment 1.5 \
--gas-prices 1000000000arai \
--node tcp://localhost:43657 \
-y
```
# Useful Commands
### Delegate
```
republicd tx staking delegate <valoper_address> 1000000000000000000arai \
--from wallet \
--chain-id raitestnet_77701-1 \
--gas auto --gas-adjustment 1.5 \
--gas-prices 1000000000arai \
--node tcp://localhost:43657 -y
```
### Unjail
```
republicd tx slashing unjail \
--from wallet \
--chain-id raitestnet_77701-1 \
--gas auto --gas-adjustment 1.5 \
--gas-prices 1000000000arai \
--node tcp://localhost:43657 -y
```
