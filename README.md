# Nibiru-Testnet-2


Explorer:
>-  https://explorer.secardnode.com/nibiru


## Sistem Gereksinimleri
### Minimum Sistem Gereksinimler
 - 4x CPUs
 - 16GB RAM
 - 500GB of disk space (SSD)


### Manuel Kurulum

// İlk Öncelikle Gerekli Updateleri Yüklüyoruz.

~~~bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make gcc -y
~~~


// Go Kurulumu Yapıyoruz

~~~bash
cd $HOME
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
~~~

// Binnary Dosyasını İndirip Build Ediyoruz

~~~bash
curl -s https://get.nibiru.fi/@v0.21.9! | bash
~~~

/// Yükledikten Sonra v0.21.9 olduğunu aşağıdaki kodu yazarak onaylayalım
~~~
nibid version
~~~
### init işlemi 
```
nibid init <moniker-name> --chain-id=nibiru-itn-2 --home $HOME/.nibid
```

### Genesisi İndiriyoruz

~~~bash
NETWORK=nibiru-itn-2
curl -s https://networks.itn2.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
~~~

### Seed ve Peerleri Giriyoruz

~~~bash
SEEDS="142142567b8a8ec79075ff3729e8e5b9eb2debb7@35.195.230.189:26656,766ca434a82fe30158845571130ee7106d52d0c2@34.140.226.56:26656"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|' $HOME/.nibid/config/config.toml
~~~



### Pruning Ayarlarını Yapıyoruz

~~~bash
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.nibid/config/app.toml
~~~

### Gerekli Chain Ayarlarını Yapıyoruz

~~~bash
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nibid/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.nibid/config/config.toml
~~~

### Ayarlamalar

```
NETWORK=nibiru-itn-2
config_file="$HOME/.nibid/config/config.toml"

sed -i "s|enable =.*|enable = true|g" "$config_file"
sed -i "s|rpc_servers =.*|rpc_servers = \"$(curl -s https://networks.itn2.nibiru.fi/$NETWORK/rpc_servers)\"|g" "$config_file"
sed -i "s|trust_height =.*|trust_height = \"$(curl -s https://networks.itn2.nibiru.fi/$NETWORK/trust_height)\"|g" "$config_file"
sed -i "s|trust_hash =.*|trust_hash = \"$(curl -s https://networks.itn2.nibiru.fi/$NETWORK/trust_hash)\"|g" "$config_file"
```

### Ve Servis Dosyası Oluşturuyoruz

~~~bash
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start --home $HOME/.nibid
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
~~~

### Servis Dosyasını Aktif Edip Çalıştırıyoruz

~~~bash
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f
~~~

### Faucet alma
NOT: adres-girin kısımına cüzdan adresinizi yaıznız
```
FAUCET_URL="https://faucet.itn-2.nibiru.fi/"
ADDR="adres-girin" # your address
curl -X POST -d '{"address": "'"$ADDR"'", "coins": ["11000000unibi","100000000unusd","100000000uusdt"]}' $FAUCET_URL
```

## Wallet Oluşturma
Mnemonic Kelimelerinizi Güvenli Bir Yere Kaydedin Kaybetmeyin

~~~bash
nibid keys add walletisminiyazıyorsunuz
~~~

### Eskiden Wallet Oluşturduysanız Recovery Edebilirsiniz

~~~bash
nibid keys add walletisminiyazıyorsunuz --recover
~~~

- Sync Olduktan Sonra Discord Faucet Sayfanızdan Cüzdan Adresinize Faucet Talep Edip Validator Oluşturabilirsiniz


### Validator Oluşturma

~~~bash
nibid tx staking create-validator \
  --amount 1000000unibi \
  --from cüzdanisminiziyazın \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(nibid tendermint show-validator) \
  --moniker SizinMonikerİsminizYazın \
  --chain-id nibiru-testnet-2 \
  --fees 10000unibi
~~~
  


### Servis Komutları
Log Kontrol

~~~bash
sudo journalctl -u nibid -f
~~~

Servis Durdurma

~~~bash
sudo systemctl stop nibid
~~~

Servis Başlatma

~~~bash
sudo systemctl start nibid
~~~

Servis Restart

~~~bash
sudo systemctl restart nibid
~~~

### Cüzdan

Cüzdan Miktar Kontrol

~~~bash
nibid query bank balances cüzdanadresiniz
~~~


### Node Silme

~~~bash
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm -rf /etc/systemd/system/nibid*
sudo rm $(which nibid)
sudo rm -rf $HOME/.nibid
sudo rm -fr $HOME/nibiru
sed -i "/NIBIRU_/d" $HOME/.bash_profile
~~~
