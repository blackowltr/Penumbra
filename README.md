# Penumbra Installation Guide

### 1. Update the System

First, ensure your system is up to date:

```shell
apt update && apt upgrade -y
```

### 2. Install Required Libraries

Install necessary libraries:

```
apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp wget jq llvm tmux htop screen unzip gcc lz4 -y < "/dev/null"
```

### 3. Install Go

Download and set up Go:

```shell
ver="1.22.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### 4. Penumbra Command Line Interface (CLI) Installation

Install the Penumbra CLI:

```shell
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/penumbra-zone/penumbra/releases/download/v0.75.0/pcli-installer.sh | sh
```

:::note Note
After completing the CLI installation, verify it was successful.

```shell
pcli --version
```
:::

### 5. Creating a Wallet

Generate a Penumbra wallet:

```shell
pcli init soft-kms generate
```

:::warning Warning
A private key (Private Seed) will be generated during the wallet creation process. Safeguard this key diligently as it's vital in case of losing access to your wallet.
:::

Locate your wallet address:

```shell
pcli view address
```

If you need to import a previously generated wallet:

```shell
pcli init soft-kms import-phrase
```

:::warning Warning
To obtain your wallet address, visit the Penumbra Discord and navigate to the "#-testnet-faucet" channel.
:::

Submit your wallet address to the Discord channel to receive faucet tokens.

Check your wallet balance:

```shell
pcli view sync
pcli view balance
```

### 6. Set up Penumbra and CometBFT

Download and configure Penumbra:

```shell
curl -sSfL -O https://github.com/penumbra-zone/penumbra/releases/download/v0.75.0/pd-x86_64-unknown-linux-gnu.tar.gz
tar -xf pd-x86_64-unknown-linux-gnu.tar.gz
sudo mv pd-x86_64-unknown-linux-gnu/pd /usr/local/bin/
```

Install CometBFT:

```shell
echo export GOPATH=\"\$HOME/go\" >> ~/.bash_profile
echo export PATH=\"\$PATH:\$GOPATH/bin\" >> ~/.bash_profile
source ~/.bash_profile
git clone --branch v0.37.5 https://github.com/cometbft/cometbft.git
cd cometbft
make install
```

### 7. Initializing Nodes

Initialize the Penumbra node:

```shell
pd testnet unsafe-reset-all
pd testnet join --external-address IPADDRESS:26656 --moniker NAME
```

### 8. Running Nodes

Start the Penumbra node:

```shell
sudo tee /etc/systemd/system/penumbra.service > /dev/null <<EOF
[Unit]
Description=Penumbra Node
After=network.target
[Service]
User=root
ExecStart=/usr/local/bin/pd start
Restart=always
RestartSec=3
LimitNOFILE=infinity
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable penumbra
systemctl start penumbra
```

Start the CometBFT node:

```shell
sudo tee /etc/systemd/system/cometbft.service > /dev/null <<EOF
[Unit]
Description=Cometbft Node
After=network.target
[Service]
User=root
ExecStart=/root/go/bin/cometbft start --home root/.penumbra/testnet_data/node0/cometbft
Restart=always
RestartSec=3
LimitNOFILE=infinity
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable cometbft
systemctl start cometbft
```

Check the status of the nodes:

```shell
journalctl -fu penumbra -n 50
journalctl -fu cometbft -n 50
```

Verify validator activation and balance:

```shell
pcli view balance
pcli query validator list --detailed
```

This condensed guide provides step-by-step instructions for installing and configuring Penumbra on your system, setting up a validator node, and delegating tokens.
