# Althea

# Althea
Althea Node Installation Instructions </br>
### [Official documentation](https://www.althea.net)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd && rm -rf althea-L1
git clone https://github.com/althea-net/althea-L1
cd althea-L1
git checkout v0.5.5
```

# Build binary
```
make install
```

# Set node CLI configuration
```
althea config chain-id althea_417834-3
althea config keyring-backend test
althea config node tcp://localhost:12457
```

# Initialize the node
```
althea init "Your Node Name" --chain-id althea_417834-3
```

# Download genesis and addrbook files
```
curl -L https://snapshots-testnet.nodejumper.io/althea-testnet/genesis.json > $HOME/.althea/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/althea-testnet/addrbook.json > $HOME/.althea/config/addrbook.json
```

# Set seeds
```
sed -i -e 's|^seeds *=.*|seeds = "df949a46ae6529ae1e09b034b49716468d5cc7e9@testnet-seeds.stakerhouse.com:10056,d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.nodex.one:21110,bc47f3e8f9134a812462e793d8767ef7334c0119@chainripper-2.althea.net:23296"|' $HOME/.althea/config/config.toml
```

# Set minimum gas price
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01ualthea"|' $HOME/.althea/config/app.toml
```

# Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.althea/config/app.toml
```

# Change ports
```
sed -i -e "s%:1317%:12417%; s%:8080%:12480%; s%:9090%:12490%; s%:9091%:12491%; s%:8545%:12445%; s%:8546%:12446%; s%:6065%:12465%" $HOME/.althea/config/app.toml
sed -i -e "s%:26658%:12458%; s%:26657%:12457%; s%:6060%:12460%; s%:26656%:12456%; s%:26660%:12461%" $HOME/.althea/config/config.toml
```

# Download latest chain data snapshot
```
curl "https://snapshots-testnet.nodejumper.io/althea-testnet/althea-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.althea"
```

# Create a service
```
sudo tee /etc/systemd/system/althead.service > /dev/null << EOF
[Unit]
Description=Althea node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which althea) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable althead.service
```

# Start the service and check the logs
```
sudo systemctl start althead.service
sudo journalctl -u althead.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
althea keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
althea keys add wallet --recover
```

### We receive tokens from the tap in the discord(https://discord.gg/w2zuaRQsJH)

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
althea status 2>&1 | jq .SyncInfo.catching_up
```

### Check the balance before creating for the presence of tokens
```
althea q bank balances $(althea keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
althea tx staking create-validator \
--amount=1000000ualthea \
--pubkey=$(althea tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I'm sexy and I know it😉" \
--chain-id=althea_417834-3 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.1ualthea \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:althea_417834-3
Current version:v0.5.5
```

### Useful commands

Check balance
```
althea q bank balances $(althea keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u althead -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart althead
```

GET VALIDATOR INFO
```
althea status 2>&1 | jq .ValidatorInfo
```

DELEGATE TOKENS TO YOURSELF
```
althea tx staking delegate $(althea keys show wallet --bech val -a) 1000000ualthea --from wallet --chain-id althea_417834-3 --gas-prices 0.1ualthea --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop althead && sudo systemctl disable althead && sudo rm /etc/systemd/system/althead.service && sudo systemctl daemon-reload && rm -rf $HOME/.althea && rm -rf althea-L1 && sudo rm -rf $(which althea)
```

