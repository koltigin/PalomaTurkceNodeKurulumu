![image](https://user-images.githubusercontent.com/102043225/177013335-d766d4c9-0815-4ded-a99b-c0508e4ee1a0.png)


### Paloma Türkçe Node Kurulum Rehberi (Paloma Testnet 6'ya Güncellenmiştir)

### Go Kurulumu

```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
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
MONIKER="moniker-isminiz"
palomad init "$MONIKER"
```

### Yapılandırma Dosyalarını Kopyalama

```shell
wget -O ~/.paloma/config/genesis.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-6/genesis.json
wget -O ~/.paloma/config/addrbook.json https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-6/addrbook.json
```

### Indexer'i İnaktif Etme
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.paloma/config/config.toml
```

### Pruning Yapma 
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.paloma/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.paloma/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.paloma/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.paloma/config/app.toml
```

### Prometheus'u Aktif Etme
```shell
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.paloma/config/config.toml
```

### Servis Dosyası Oluşturma
Aşağıdaki kodu tek seferde giriniz.
```shell
sudo tee /etc/systemd/system/palomad.service > /dev/null <<EOF
[Unit]
Description=paloma
After=network-online.target

[Service]
User=$USER
ExecStart=$(which palomad) start --home $HOME/.paloma
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Node'u Başlatma
Aşağıdaki kodları da tek seferde girebilirsiniz.
```shell
sudo systemctl daemon-reload
sudo systemctl enable palomad
sudo systemctl restart palomad
```

### Cüzdan

## Yeni Cüzdan Oluşturma
<cüzdan-adınız> bu kısmı (<> dahil silerek cüzdan adınızı yazınız)
```shell
VALIDATOR=<cüzdan-adınız>
palomad keys add "$VALIDATOR"
```

## Var Olan Cüzdanı İçeri Aktarma

```
palomad keys add "CUZDAN_ADINIZ" --recover
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
palomad query bank balances CUZDAN_ADRESI
```

### ARA NOT
Bu aşamada validator oluşturmadan önce aşağıdaki kodlar ile blokları takip edebilir; 

```
journalctl -fu palomad -o cat
```

Aşağıdaki kod ile de senkronizasyonu takip edebilirsiniz. Eğer false çıktısı alıyorsanız validator oluşturma adımına geçebilirsiniz. 
```
palomad status 2>&1 | jq .SyncInfo

```

### PEER Bulma Sorunu Olursa Aşağıdaki Dosyayı Kullanabilirsiniz.



### Validator Oluşturma
1 GRAIN = 1.000.000 uGRAIN = 1000000ugrain

```
PUBKEY="$(palomad tendermint show-validator)"
palomad tx staking create-validator \
      --fees=1000000ugrain \
      --from="CUZDAN_ADINIZ" \
      --amount="10000000ugrain" \
      --pubkey="$PUBKEY" \
      --moniker="VALIDATOR_ADINIZ" \
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

## Peer Adresinizi Öğrenme

```
echo "$(palomad tendermint show-node-id)@$(curl ifconfig.me):26656"
```

## Cüzdanların Listesine Bakma

```
palomad keys list
```

## Cüzdanı İçeri Aktarma

```
palomad keys add CUZDAN_ADI --recover
```

## Cüzdanı Silme

```
palomad keys delete CUZDAN_ADI
```

## Cüzdan Bakiyesine Bakma

```
palomad query bank balances CUZDAN_ADRESI
```

## Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
palomad tx bank send CUZDAN_ADRESI GONDERILECEK_CUZDAN_ADRESI 100000000grain
```

## Proposal Oylamasına Katılma

```
palomad tx gov vote 1 yes --from CUZDAN_ADI --chain-id=paloma-testnet-6
```

## Validatore Stake Etme / Delegate Etme

```
palomad tx staking delegate $VALOPER_ADDRESS 100000000grain --from=CUZDAN_ADI --chain-id=paloma-testnet-6 --gas=auto
```

## Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme

```
palomad tx staking redelegate <MevcutValidatorAdresi> <StakeEdilecekYeniValidatorAdresi> 100000000grain --from=CUZDAN_ADI/CUZDAN_ADRESI --chain-id=paloma-testnet-6 --gas=auto
```

## Ödülleri Çekme

```
palomad tx distribution withdraw-all-rewards --from=CUZDAN_ADI --chain-id=paloma-testnet-6 --gas=auto
```

## Komisyon Ödüllerini Çekme

```
palomad tx distribution withdraw-rewards $VALOPER_ADDRESS --from=CUZDAN_ADI --commission --chain-id=paloma-testnet-6
```

## Validator İsmini Değiştirme

```
palomad tx staking edit-validator \
--moniker=YENI_NODE_ADI \
--chain-id=paloma-testnet-6 \
--from=CUZDAN_ADI
```

## Validatoru Jail Durumundan Kurtarma 

```
palomad tx slashing unjail \
  --broadcast-mode=block \
  --from=CUZDAN_ADI \
  --chain-id=paloma-testnet-6 \
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
