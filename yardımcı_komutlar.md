## İhtiyaç olabilecek komutlar:

```sh
# Validatör düzenleme
cascadiad tx staking edit-validator \
  --new-moniker=$CASCADIA_MONIKER \
  --website="" \
  --identity=<"" \
  --details="" \
  --chain-id=$CASCADIA_CHAIN_ID \
  --gas auto \
  --gas-adjustment=1.2 \
  --gas-prices=7aCC \
  --from=wallet

# Olduğun bloğu kontrol etme
cascadiad status 2>&1 | jq ."SyncInfo"."latest_block_height"

# Node durumu
cascadiad status 2>&1 | jq .SyncInfo

# Validatör pubkeyi görme
cascadiad tendermint show-validator

# txhash bilgisi kontrol etme
cascadiad q tx tx_hash

# Validatörün kaç blok kaçırdı, jail  bilgisi ve bir kaç yardımcı bilgi.
cascadiad q slashing signing-info $(cascadiad tendermint show-validator)

# Cüzdanları listeleme
cascadiad keys list

# Cüzdan silmek isterseniz
cascadiad keys delete cüzdan_ismi
```
## Validatör yönetimi hakkında komutlar (kendinize göre düzenlemeyi unutmayın)
```sh
# Tüm blok ödüllerini toplamak için
cascadiad tx distribution withdraw-all-rewards --from rues --gas auto --gas-adjustment=1.2 --gas-prices=7aCC -y

# Kendine delege etmek için. 1000000000000000000aCC = 1 Cascadia token
cascadiad tx staking delegate $CASCADIA_VALOPER 1000000000000000000aCC --from rues --gas auto --gas-adjustment=1.2 --gas-prices=7aCC -y

# Redelegasyon
cascadiad tx staking redelegate delegasyon_adresi redelege_edilecek_adres 1000000000000000000aCC --from rues --gas auto --gas-adjustment=1.2 --gas-prices=7aCC -y

# Undelege
cascadiad tx staking unbond undelege_edilecek_adres 1000000000000000000aCC --from rues --gas auto --gas-adjustment=1.2 --gas-prices=7aCC -y

# Transfer işlemi
cascadiad tx bank send rues gönderilecek_adres 1000000000000000000aCC --gas auto --gas-adjustment=1.2 --gas-prices=7aCC -y

# Jailden kurtulma
cascadiad tx slashing unjail --from rues --gas auto --gas-adjustment=1.2 --gas-prices=7aCC
```

## Oylamalar önemli oluyor bazı zamanlar puan alıyoruz
```sh
# Oylamaları listeleme
cascadiad q gov proposals

# Oy verme, 1 yazan yerde oylama numarası, bu 18'de olabilir. Yes yazan yerede yes veya no olarak belirtin oyunuza göre.
cascadiad tx gov vote 1 yes --from cascadia_adresiniz
```

## Node'u silmek
```sh
# Bu komutu sona bıraktım, çok zorda kalmadığınız sürece silmeyin, hemen pes etmeyin.
sudo systemctl stop cascadiad && \
sudo systemctl disable cascadiad && \
rm /etc/systemd/system/cascadiad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf cascadia && \
rm -rf .cascadiad && \
rm -rf $(which cascadiad)
```
