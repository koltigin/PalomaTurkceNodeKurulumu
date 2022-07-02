![Logo!](assets/paloma.png)



### Paloma Türkçe Node Kurulum Rehberi (Paloma Testnet 6'ya Güncellenmiştir)

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
wget -O - https://github.com/palomachain/paloma/releases/download/v0.2.5-prealpha/paloma_0.2.5-prealpha_Linux_x86_64.tar.gz | \
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
wget -O ~/.paloma/config/genesis.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-6/genesis.json
wget -O ~/.paloma/config/addrbook.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-6/addrbook.json
```

### Cüzdan

## Yeni Cüzdan Oluşturma
<cüzdan-adınız> bu kısmı (<> dahil silerek cüzdan adınızı yazınız)
```shell
VALIDATOR=<cüzdan-adınız>
palomad keys add "$VALIDATOR"
```

## Var Olan Cüzdanı İçeri Aktarma
$VALIDATOR bömünü silip cüzdan adınızı yazınız.
```
palomad keys add "$VALIDATOR" --recover
```

### Test Tokeni alma

https://faucet.palomaswap.com/ adresinden ya da aşağıdaki kodu terminalde girerek token talep edebilirsiniz.

Aşağıdaki kodda yer alan $ADDRESS bu bölümü silerek paloma adresinizi yazınız.

```shell
ADDRESS="$(palomad keys show "$VALIDATOR" -a)"
JSON=$(jq -n --arg addr "$ADDRESS" '{"denom":"ugrain","address":$addr}') && curl -X POST --header "Content-Type: application/json" --data "$JSON" https://backend.faucet.palomaswap.com/claim
```

### Cüzdan Bakyesini Kontrol Etme
```shell
palomad query bank balances --node tcp://testnet.palomaswap.com:26657 "$ADDRESS"
```

### Validator Oluşturma

```
MONIKER="$(hostname)"
VALIDATOR="$(palomad keys list --list-names | head -n1)"
STAKE_AMOUNT=1000000ugrain
PUBKEY="$(palomad tendermint show-validator)"
palomad tx staking create-validator \
      --fees=1000000ugrain \
      --from="$VALIDATOR" \
      --amount="$STAKE_AMOUNT" \
      --pubkey="$PUBKEY" \
      --moniker="$MONIKER" \
      --identity=kaybase.io'dan aldığınız id'nizi buraya giriniz \
      --website="https://www.example.com" \
      --details="Bu bölüme istediğiniz bir cümya ya da her ne ise onu yazabilirsiniz" \
      --chain-id=paloma-testnet-6 \
      --commission-rate="0.1" \
      --commission-max-rate="0.2" \
      --commission-max-change-rate="0.05" \
      --min-self-delegation="100" \
      --yes \
      --broadcast-mode=block
```

Aşağıdaki Testnet 5'te kullandığımız kod yapısı. (Yukarıdaki kod test edildikten sonra tekrar güncelleyeceğim)
```shell
palomad tx staking create-validator \
--amount=2950000ugrain \
--pubkey=$(palomad tendermint show-validator) \
--moniker=validator-ismi \
--chain-id=paloma-testnet-5 \
--commission-rate="0.05" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--fees=10000ugrain \
--gas=10000000 \
--from=cuzdan-ismi \
--node "tcp://testnet.palomaswap.com:26657" \
-y
```


### FAYDALI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu palomad -o cat
```

## Sistemi Başlatma

```
systemctl start palomad
```

## Sistemi Durdurma
```
systemctl stop palomad
```

## Sistemi Yeniden Başlatma
```
systemctl restart palomad
```

## Node Senkronizasyon Durumu
```
palomad status 2>&1 | jq .SyncInfo

```

## Validator Bilgileri
```
palomad status 2>&1 | jq .ValidatorInfo
```

## Node Bilgileri

```
palomad status 2>&1 | jq .NodeInfo
```

## Node ID Öğrenme

```
palomad tendermint show-node-id
```


## Node IP Adresini Öğrenme

```
curl icanhazip.com
```

## Cüzdanların Listesine Bakma

```
palomad keys list
```

## Cüzdanı İçeri Aktarma

```
palomad keys add $WALLET --recover
```

## Cüzdanı Silme

```
palomad keys delete $WALLET
```

## Cüzdan Bakiyesine Bakma

```
palomad query bank balances $WALLET_ADDRESS
```

## Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
palomad tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 100000000grain
```

## Proposal Oylamasına Katılma

```
palomad tx gov vote 1 yes --from $WALLET --chain-id=$CHAIN_ID
```

## Validatore Stake Etme / Delegate Etme

```
palomad tx staking delegate $VALOPER_ADDRESS 100000000grain --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

## Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme

```
palomad tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000grain --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

## Ödülleri Çekme

```
palomad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

## Komisyon Ödüllerini Çekme

```
palomad tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CHAIN_ID
```

## Validator İsmini Değiştirme

```
seid tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$CHAIN_ID \
--from=$WALLET
```

## Validatoru Jail Durumundan Kurtarma 

```
palomad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID \
  --gas=auto
```

## Node'u Tamamen Silme 

```
sudo systemctl stop palomad && \
sudo systemctl disable palomad && \
rm /etc/systemd/system/palomad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .paloma paloma && \
rm -rf $(which palomad)
```

## Bir Lokal Kontrat Yükleme

```shell
CONTRACT=<contract.wasm>
VALIDATOR="$(palomad keys list --list-names | head -n1)"
palomad tx wasm store "$CONTRACT" --from "$VALIDATOR" --broadcast-mode block -y --gas auto --fees 3000grain
```


