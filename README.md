# Hedge-install

Update && Upgrade:

    sudo apt update && sudo apt upgrade -y

Install package

    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make lz4 unzip ncdu -y

Install Go

    ver="1.21.5" 
    cd $HOME 
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 
    sudo rm -rf /usr/local/go 
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
    rm "go$ver.linux-amd64.tar.gz"
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile

Install Node

    sudo wget -O hedged https://github.com/hedgeblock/testnets/releases/download/v0.1.0/hedged_linux_amd64_v0.1.0
    chmod +x hedged
    mkdir -p $HOME/go/bin
    sudo mv hedged /go/bin
    set -eux; \wget -O /lib/libwasmvm.x86_64.so https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so

Config Node

    hedged config chain-id berberis-1
    hedged config keyring-backend test
    hedged init "Moniker" --chain-id berberis-1
    sudo wget -O $HOME/.hedge/config/genesis.json "https://raw.githubusercontent.com/NodeValidatorVN/GuideNode/main/Hedge/genesis.json"
    sudo wget -O $HOME/.hedge/config/addrbook.json "https://raw.githubusercontent.com/NodeValidatorVN/GuideNode/main/Hedge/addrbook.json"
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uhedge\"/;" ~/.hedge/config/app.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.hedge/config/config.toml
    peers=""
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.hedge/config/config.toml
    seeds="7879005ab63c009743f4d8d220abd05b64cfee3d@54.92.167.150:26656"
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.hedge/config/config.toml
    sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.hedge/config/config.toml
    sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.hedge/config/config.toml

Pruning and indexer

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.hedge/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.hedge/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.hedge/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.hedge/config/app.toml

Create Service

    sudo tee /etc/systemd/system/hedged.service > /dev/null <<EOF
    [Unit]
    Description=Hedged Node
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which hedged) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo systemctl enable hedged

Check Logs

    sudo systemctl start hedged && journalctl -u hedged -f -o cat

Check Syn

    hedged status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'

Add New Key

    hedged keys add wallet

Recover Existing Key

    hedged keys add wallet --recover

        

    
