![Logo!](assets/paloma.png)



#### Paloma Türkçe Node Kurulum Rehberi

### Go Kurulumu

```
cd $HOME
wget -O go1.17.1.linux-amd64.tar.gz https://golang.org/dl/go1.17.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.1.linux-amd64.tar.gz && rm go1.17.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

### Gerekli Kütüphanelerin Kurulması
```
cd $HOME
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
```

### Paloma Kurulumu

```shell
wget -O - https://github.com/palomachain/paloma/releases/download/v0.2.3-prealpha/paloma_0.2.3-prealpha_Linux_x86_64v3.tar.gz | \
sudo tar -C /usr/local/bin -xvzf - palomad
sudo chmod +x /usr/local/bin/palomad
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/raw/main/api/libwasmvm.x86_64.so
```

### Node Adı Oluşturma

```shell
MONIKER="<moniker-isminiz>"
palomad init "$MONIKER"
```

### Yapılandırma Dosyalarını Kopyalama

```shell
wget -O ~/.paloma/config/genesis.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-4/genesis.json
wget -O ~/.paloma/config/addrbook.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-4/addrbook.json
```

### Cüzdan

## Yeni Cüzdan Oluşturma
```shell
VALIDATOR=<cüzdan-adını>
palomad keys add "$VALIDATOR"
```

## Var Olan Cüzdanı İçeri Aktarma
```
palomad keys add "$VALIDATOR" --recover
```

### Test Tokeni alma

```shell
ADDRESS="$(palomad keys show "$VALIDATOR" -a)"
JSON=$(jq -n --arg addr "$ADDRESS" '{"denom":"ugrain","address":$addr}') && curl -X POST --header "Content-Type: application/json" --data "$JSON" https://backend.faucet.palomaswap.com/claim
```

### Cüzdan Bakyesini Kontrol Etme
We can verify the new funds have been deposited.
```shell
palomad query bank balances --node tcp://testnet.palomaswap.com:26657 "$ADDRESS"
```

### Validator Oluşturma

Stake your funds and create our validator.
```shell
MAIN_VALIDATOR_NODE="tcp://testnet.palomaswap.com:26657"
CHAIN_ID="paloma-testnet-4"
DEFAULT_GAS_AMOUNT="10000000"
VALIDATOR_STAKE_AMOUNT=100000grain
PUBKEY="$(palomad tendermint show-validator)"
palomad tx staking create-validator \
      --fees 10000grain \
      --from="$VALIDATOR" \
      --amount="$VALIDATOR_STAKE_AMOUNT" \
      --pubkey="$PUBKEY" \
      --moniker="$MONIKER" \
      --chain-id="$CHAIN_ID" \
      --commission-rate="0.1" \
      --commission-max-rate="0.2" \
      --commission-max-change-rate="0.05" \
      --min-self-delegation="100" \
      --gas=$DEFAULT_GAS_AMOUNT \
      --node "$MAIN_VALIDATOR_NODE" \
      --yes \
      -b block
```


Start it!

```shell
MAIN_PEER_DESIGNATION=175ccd9b448390664ea121427aab20138cc8fcec@testnet.palomaswap.com:26656
palomad start --p2p.persistent_peers "$MAIN_PEER_DESIGNATION"
```

#### Running with `systemd`

First configure the service:

```shell
cat << EOT > /etc/systemd/system/palomad.service
[Unit]
Description=Paloma Blockchain
After=network.target
ConditionPathExists=/usr/local/bin/palomad

[Service]
Type=simple
Restart=always
RestartSec=5
WorkingDirectory=~
ExecStartPre=
ExecStart=/usr/local/bin/palomad start --p2p.persistent_peers 175ccd9b448390664ea121427aab20138cc8fcec@testnet.palomaswap.com:26656
ExecReload=

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=65535
EOT
```

Then reload systemd configurations and start the service!

```shell
systemctl daemon-reload
service palomad start

# Check that it started successfully.
service palomad status
```

### Uploading a local contract

```shell
CONTRACT=<contract.wasm>
VALIDATOR="$(palomad keys list --list-names | head -n1)"
palomad tx wasm store "$CONTRACT" --from "$VALIDATOR" --broadcast-mode block -y --gas auto --fees 3000grain
```

## Setting up a new private testnet

Download and install the latest release of palomad.

Install `jq`, used by the setup script.

```shell
apt install jq
```

Set up the chain validator.

```shell
CHAIN_ID=paloma-iona \
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/palomachain/paloma/master/scripts/setup-volume-testnet.sh)"
```

We should now see an error free execution steadily increasing chain depth.

```shell
palomad start
```

Others wishing to connect to the new testnet will need your `.paloma/config/genesis.json` file,
as well as the main peer designation. We can get the main peer designation with `jq`:

```shell
jq -r '.body.memo' ~/.paloma/config/gentx/gentx-*.json
```
