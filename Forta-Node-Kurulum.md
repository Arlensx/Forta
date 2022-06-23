# 1 - Forta Network Node Manuel Kurulum

apt update && apt upgrade <br><br>
apt install ca-certificates curl gnupg lsb-release git htop<br><br>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg<br><br>
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null <br><br>
apt-get update<br><br>
apt-get install docker-ce docker-ce-cli containerd.io <br> <br>


# 2 - Docker sürümünü kontrol et
docker version


# 3 
sudo tee /etc/docker/daemon.json > /dev/null <<EOF<br>
{<br>
   "default-address-pools": [<br>
        {<br>
            "base":"172.17.0.0/12",<br>
            "size":16<br>
        },<br>
        {<br>
            "base":"192.168.0.0/16",<br>
            "size":20<br>
        },<br>
        {<br>
            "base":"10.99.0.0/16",<br>
            "size":24<br>
        }<br>
    ]<br>
}<br>
EOF<br>
systemctl restart docker


# 4

Aşağıda ki işlemlerden sonra, bir adres oluştuğunu göreceksiniz. Bu adresi kaydedin! Sizin scanner (node) adresiniz budur. Bu adrese, 0.1 MATIC gönderin. (Testnet tokeni değil, Mainnet tokeni) cüzdanınızda yoksa, bir borsadan alıp aktarmayı deneyin. {ŞİFRENİZİ_GİRİN} kısmına, şifrenizi girmeyi unutmayın!!!



sudo curl https://dist.forta.network/pgp.public -o /usr/share/keyrings/forta-keyring.asc -s <br>
echo 'deb [signed-by=/usr/share/keyrings/forta-keyring.asc] https://dist.forta.network/repositories/apt stable main' | sudo tee -a /etc/apt/sources.list.d/forta.list<br>
apt-get update<br>
apt-get install forta<br>
forta init --passphrase {ŞİFRENİZİ_GİRİN}

# 5

Aşağıda ki {ALCHEMY_ENDPOİNT} yazılı kısma, kaydettiğiniz endpointi girip komutu terminale girin.

rm /root/.forta/config.yml
sudo tee /root/.forta/config.yml > /dev/null <<EOF
chainId: 137

scan:
  jsonRpc:
    url: {ALCHEMY_ENDPOİNT}

trace:
  enabled: false
EOF
                                                   
# 6                                               
         
Bu adımda, metamask adresinize ihtiyacınız olacak. Metamask adresinizi bir kenara kaydedin. Şimdi kullanacağız. {ŞİFRENİZİ_GİRİN} kısmına, belirlediğiniz şifreyi girin<br>
                                                   
forta register --owner-address {METAMASK_ADDRESS} --passphrase {ŞİFRENİZİ_GİRİN}

# 7

servis dosyası oluşturacağız. {ŞİFRENİZİ_GİRİN} yazılı kısma, üstte olduğu gibi yeniden şifrenizi girmeyi unutmayın. <br><br>

sudo tee /lib/systemd/system/forta.service > /dev/null <<EOF
[Unit]
Description=Forta
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
Environment="FORTA_DIR=/root/.forta/"
Environment="FORTA_PASSPHRASE={ŞİFRENİZİ_GİRİN}"
Restart=on-failure
RestartSec=15s

ExecStart=/usr/bin/forta run

[Install]
WantedBy=multi-user.target
EOF

systemctl enable forta
systemctl daemon-reload
systemctl start forta
