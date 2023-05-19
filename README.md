<h1 align="center"> Cascadia - DevNet </h1>

<h1 align="center"> Neden Cascadi node'u kuruyorum? </h1>

> Uzun bir süredir Cascadia DevNet'te, [Burada](https://twitter.com/Ruesandora0/status/1592480840512311299?s=20) paylaşmıştım, kontrol edin.

> Yakında Cascadia incentivize testnete geçiş yapacak, ben şahsen katılacağım.

> Yakında geçiş yapacağı ve kısa süreceği için, validatör rolünü almak içini kurdum, başka nedeni yok.

> Topluluk kanalları: [Duyuru kanalım](https://t.me/RuesAnnouncement) - [Sohbet kanalı](https://t.me/RuesChat) - [Cascadia Discord](https://discord.gg/cascadia)

> İhtiyaç olabilecek yardımcı komutlar [Linki](https://github.com/ruesandora/Cascadia/blob/main/yard%C4%B1mc%C4%B1_komutlar.md) ve [Cascadia Platformu](https://align.cascadia.foundation/)

## <h1 align="center"> Donanım </h1>
> Sunucu temin edemiyorsanız ve para vermek istemiyorsanız [Hetzner $20](https://hetzner.cloud/?ref=gIFAhUnYYjD3) veriyor.
```
# Hetzner'den 3 GPU 4 RAM sunucuya kurdum, sıkıntısız çalışıyor. 
# 3 dolarlık sunucu dahi kaldırır tahminim, ben yinede 7$ olanı kullandım.
# Sunucuyu beğenmezsen kapatıp yenisini açabilirsin, hetznerde saatlik ücret var, aylık yok. Sanırım bir kaç saat kullanmaya da bir şey almıyor.
```
![image](https://github.com/ruesandora/Cascadia/assets/101149671/a15c7404-3bab-4b79-8bfa-cc0aad56be1c)

```sh
# Sunucumuzu güncelleyelim
sudo apt update && sudo apt upgrade -y

apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
```sh
# Go'yu yüklüyoruz
if [ "$(go version)" != "go version go1.20.2 linux/amd64" ]; then \
ver="1.20.2" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile ; \
fi

go version

# go version sonrası çıktı: go version go1.20.2 linux/amd64
```

<h1 align="center"> Değişkenleri Ayarlayalım </h1>

```sh
# Validator_İsmi kısmına validatör (moniker) belirleyin.
CASCADIA_MONIKER=VALİDATOR_İSMİ

echo 'export CASCADIA_MONIKER='$CASCADIA_MONIKER >> $HOME/.bash_profile
echo "export CASCADIA_CHAIN_ID=cascadia_6102-1" >> $HOME/.bash_profile
echo "export CASCADIA_PORT=18" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```sh
# Binary dosyalarını yüklüyoruz
cd $HOME
git clone https://github.com/cascadiafoundation/cascadia && cd cascadia
git checkout v0.1.2
make install

cascadiad version --long | grep -e version -e commit

# Version kontrolde şöyle bir çıktı: 0.1.2 - commit: bde803072f5f52884a372c02d2249e743de9538d
```

```sh
# İnitalize işlemi yapalım
cascadiad init $CASCADIA_MONIKER --chain-id $CASCADIA_CHAIN_ID
# Uzun bir çıktı alacaksınız
```

```sh
# Genesisi yüklüyoruz
wget -O $HOME/.cascadiad/config/genesis.json "https://anode.team/Cascadia/test/genesis.json"

sha256sum $HOME/.cascadiad/config/genesis.json

# çıktı: 74ea3c84182028300d0c101c5cf017a055782c595ed91e4be3638380f0169582
```

<h1 align="center"> Portları, Peerleri ve Config Dosyaları </h1>

```sh
# config.toml
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CASCADIA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CASCADIA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CASCADIA_PORT}061\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CASCADIA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CASCADIA_PORT}660\"%" $HOME/.cascadiad/config/config.toml

# app.toml
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CASCADIA_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CASCADIA_PORT}91\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1${CASCADIA_PORT}7\"%" $HOME/.cascadiad/config/app.toml

# client.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${CASCADIA_PORT}657\"%" $HOME/.cascadiad/config/client.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${CASCADIA_PORT}656\"/" $HOME/.cascadiad/config/config.toml
```

<h1 align="center"> Config Yapılandırmaları </h1>

```sh
cascadiad config chain-id $CASCADIA_CHAIN_ID

cascadiad config keyring-backend test

cascadiad config node tcp://localhost:${CASCADIA_PORT}657

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025aCC\"/" ~/.cascadiad/config/app.toml

# seedleri ve peerleri ekliyoruz
peers="001933f36a6ec7c45b3c4cef073d0372daa5344d@194.163.155.84:49656,f78611ffa950efd9ddb4ed8f7bd8327c289ba377@65.109.108.150:46656,783a3f911d98ad2eee043721a2cf47a253f58ea1@65.108.108.52:33656,6c25f7075eddb697cb55a53a73e2f686d58b3f76@161.97.128.243:27656,8757ec250851234487f04466adacd3b1d37375f2@65.108.206.118:61556,df3cd1c84b2caa56f044ac19cf0267a44f2e87da@51.79.27.11:26656,d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:55656,f075e82ca89acfbbd8ef845c95bd3d50574904f5@159.69.110.238:36656,63cf1e7583eabf365856027815bc1491f2bc7939@65.108.2.41:60556,d5ba7a2288ed176ae2e73d9ae3c0edffec3caed5@65.21.134.202:16756"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.cascadiad/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.cascadiad/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cascadiad/config/config.toml

# İşe yaramaz peerleri kaldıracak filtreyi ekliyoruz
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.cascadiad/config/config.toml

# Pruning'i kuralım. (Pruning: diskten tasaruf eder, CPU ve RAM'den yer, opsiyoneldir.)
pruning="custom"
pruning_keep_recent="1000"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cascadiad/config/app.toml

# İndexeri kaldıralım, opsiyoneldir
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cascadiad/config/config.toml
```

```sh
# Servis dosyasını oluşturalım (tekte girebilirsiniz)
sudo tee /etc/systemd/system/cascadiad.service > /dev/null <<EOF
[Unit]
Description=Cascadia Node
After=network.target
 
[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME/go/bin
ExecStart=/root/go/bin/cascadiad start --trace --log_level info --json-rpc.api eth,txpool,personal,net,debug,web3 --api.enable
Restart=on-failure
StartLimitInterval=0
RestartSec=3
LimitNOFILE=65535
LimitMEMLOCK=209715200
 
[Install]
WantedBy=multi-user.target
EOF
```

```sh
# node'u  başlatalım
systemctl daemon-reload
systemctl enable cascadiad
systemctl restart cascadiad && journalctl -u cascadiad -f -o cat
```

```sh
# Node'u ctrl c ile durdurabilir tekrar bu komutla izleyebilirsiniz
journalctl -u cascadiad -f -o cat
# [Explorer Linki](https://validator.cascadia.foundation/validators/cascadiavaloper1s03cy478zv9w4sf9hkwl8dlvx82ncsxayrwmgj) Güncel blok 622k, 1 saate eşleşir tahminim.
```

<h1 align="center"> Eşleşirken hata alırsanız </h1>

```sh
sudo systemctl stop cascadiad
cp $HOME/.cascadiad/data/priv_validator_state.json $HOME/.cascadiad/priv_validator_state.json.backup
rm -rf $HOME/.cascadiad/data

curl -L https://snap.hexnodes.co/cascadia/cascadia.latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.cascadiad
mv $HOME/.cascadiad/priv_validator_state.json.backup $HOME/.cascadiad/data/priv_validator_state.json

sudo systemctl start cascadiad && sudo journalctl -fu cascadiad -o cat
```

<h1 align="center"> Eşleşirken Geri Kalan İşlemler </h1>

```shsh
# Statusu kontrol etme
curl localhost:${CASCADIA_PORT}657/status

# False çıktısında validatör oluşturabiliriz, sync olurken true çıktısı verir.
```

```sh
# Cüzdan oluşturma
cascadiad keys add rues
# rues kısmına wallet isminizi girin ve kaydedin bilgilerinizi.

# Recover yapmak isterseniz
cascadiad keys add rues --recover
# Cüzdan bilgileriniz ile faucetten token alın, discorda mevcut.
```
```sh 
# Token geldiğini kontrol edelim
cascadiad query bank balances cüzdan_adresiniz
# Tokenleri hangi blokta talep ederseniz, node'unuz o bloğa gelene kadar tokenlerinizi göstermez
# Güncel blokta değilseniz explorer'dan bakınız.
```

<h1 align="center"> Validatör Oluşturma </h1>

```sh
# sync olduktan yani false çıktısı verdikten sonra yapıyoruz burayı.
cascadiad tx staking create-validator \
--amount 1000000000000000000aCC \
--from=wallet \
--commission-rate "0.05" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(cascadiad tendermint show-validator) \
--moniker=$CASCADIA_MONIKER \
--chain-id=$CASCADIA_CHAIN_ID \
--identity="şart_değil" \
--details="şart_değil" \
--website="şart_değil" \
--gas auto \
--gas-adjustment=1.2 \
--gas-prices=7aCC \
-y

# Şart değil dediğim yerleri doldurmayacaksanız eğer silip "" tırnakları boş bırakın
# Validatör oluşturduktan sonra discord-validatör kanallarına explorer'dan validatorünüzün linkini atın ve rolü alın.
```

> İhtiyaç olabilecek yardımcı komutlar [linki](https://github.com/ruesandora/Cascadia/blob/main/yard%C4%B1mc%C4%B1_komutlar.md)

> Bu repoları forklayın - yıldızlayın github profiliniz boş durmasın.
