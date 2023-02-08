# Namada-Pre-Genesis-Guide

```bash
#Environment Setup

cd $HOME
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev libclang-dev -y
sudo apt install jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
sudo apt install -y uidmap dbus-user-session


cd $HOME
sudo apt update
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
. $HOME/.cargo/env
curl https://deb.nodesource.com/setup_16.x | sudo bash
sudo apt install cargo nodejs -y < "/dev/null"
cargo --version

  
if ! [ -x "$(command -v go)" ]; then
ver="1.19.4"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
fi


echo "export NAMADA_TAG=v0.13.3" >> ~/.bash_profile
echo "export TM_HASH=v0.1.4-abciplus" >> ~/.bash_profile
source ~/.bash_profile


cd $HOME && git clone https://github.com/anoma/namada && cd namada && git checkout $NAMADA_TAG
make build-release

cd $HOME && git clone https://github.com/heliaxdev/tendermint && cd tendermint && git checkout $TM_HASH
make build

cd $HOME && cp $HOME/tendermint/build/tendermint /usr/local/bin/tendermint && cp "$HOME/namada/target/release/namada" /usr/local/bin/namada && cp "$HOME/namada/target/release/namadac" /usr/local/bin/namadac && cp "$HOME/namada/target/release/namadan" /usr/local/bin/namadan && cp "$HOME/namada/target/release/namadaw" /usr/local/bin/namadaw


#Create a pre-genesis
cd namada
export ALIAS="CHOOSE_A_NAME_FOR_YOUR_VALIDATOR"
export PUBLIC_IP="YOUR_LAPTOP_OR_SERVER_IP"

namada client utils init-genesis-validator --alias $ALIAS \
--max-commission-rate-change 0.01 --commission-rate 0.05 \
--net-address $PUBLIC_IP:26656


#join the network
echo "export CHAIN_ID=public-testnet-3.0.81edd4d6eb6" >> ~/.bash_profile

namada client utils join-network \
--chain-id $CHAIN_ID --genesis-validator $ALIAS


#Start your node and sync
1) start node:
NAMADA_TM_STDOUT=true namada node ledger run

2) Optional: If you want more logs, you can instead run:
NAMADA_LOG=debug ANOMA_TM_STDOUT=true namada node ledger run

3) And if you want to save your logs to a file, you can instead run:
TIMESTAMP=$(date +%s) ANOMA_LOG=debug NAMADA_TM_STDOUT=true namada node ledger run &> logs-${TIMESTAMP}.txt tail -f -n 20 logs-${TIMESTAMP}.txt ## (in another shell)


Note: If started correctly you should see a the following log: [<timestamp>] This node is a validator ...
