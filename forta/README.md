

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/166480394-78f4659d-f4d8-4194-80de-a4080b207978.png">
</p>

# Manual node  setup
If you want to setup fullnode manually follow the steps below

Official documentation:
> [Run a Scan Node](https://docs.forta.network/en/latest/scanner-quickstart/)

Explorer:
> https://explorer.forta.network/network

## Hardware requirements:
#### For running a Forta scan node we recommend the following:
- CPU: 4 cores
- Memory: 16GB RAM
- SSD: 100GB drive space

## Preparation
### 1. Create new metamask wallet and fund it with 1 MATIC in Polygon network. This address will be used as forta owner
### 2. Register at https://moralis.io/ to generate proxy rpc url


## 0. Update packages
```
sudo apt update && sudo apt upgrade -y
```

### 1. Download & Install Docker
```
apt install ca-certificates curl gnupg lsb-release git htop
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
apt-get update
```
```
apt-get install docker-ce docker-ce-cli containerd.io
```
Cek versi docker
```
docker version
```

### 2. Konfigurasikan Docker Address Pools
```
tee /etc/docker/daemon.json > /dev/null <<EOF
{
   "default-address-pools": [
        {
            "base":"172.17.0.0/12",
            "size":16
        },
        {
            "base":"192.168.0.0/16",
            "size":20
        },
        {
            "base":"10.99.0.0/16",
            "size":24
        }
    ]
}
EOF
systemctl restart docker
```

## 3 Install forta
### 3.1 Download and install forta binaries
```
sudo curl https://dist.forta.network/pgp.public -o /usr/share/keyrings/forta-keyring.asc -s
echo 'deb [signed-by=/usr/share/keyrings/forta-keyring.asc] https://dist.forta.network/repositories/apt stable main' | sudo tee -a /etc/apt/sources.list.d/forta.list
sudo apt-get update
sudo apt-get install forta
```

## 3.2 Configure systemd service

```
nano /lib/systemd/system/forta.service
```

add this config inside forta.service file
```
[Unit]
Description=Forta
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
Environment="FORTA_DIR=/root/.forta/"
Environment="FORTA_PASSPHRASE=<password>"
Restart=on-failure
RestartSec=15s

ExecStart=/usr/bin/forta run

[Install]
WantedBy=multi-user.target
```
- password jgn lupa diisi

## 4. Initial forta scan node setup (Membuat Node scan baru)
### 4.1 Initialize Forta using the forta init command
```
forta init
```
> This is the value that will be registered in the scan node registry smart contract. If you need to find out your address later again, you can run `forta account address`

### 5.1 Send 0.1 Matic in Polygon network to your scan node address

### 6 Mengkonfigurasikan Forta (config.yml)
```
nano /root/.forta/config.yml
```
> hapus semua konfigurasi sebelumnya dengan menahan Ctrl + K sampai field kosong, kemudian masukkan salah satu chain yang akan anda gunakan:

```
Binance Smart Chain
chainId: 56

scan:
  jsonRpc:
    url: https://bsc-dataseed.binance.org/

trace:
  enabled: false

Polygon
chainId: 137

scan:
  jsonRpc:
    url: https://polygon-rpc.com/

trace:
  enabled: false

Avalanche
chainId: 43114

scan:
  jsonRpc:
    url: https://api.avax.network/ext/bc/C/rpc

trace:
  enabled: false

Arbitrum
chainId: 42161

scan:
  jsonRpc:
    url: https://arb1.arbitrum.io/rpc

trace:
  enabled: false

Optimism
chainId: 10

scan:
  jsonRpc:
    url: https://mainnet.optimism.io

trace:
  enabled: false
  ```
>( simpan dengan cara tekan Ctrl + X , lalu tekan Y pada keyboard, dan tekan Enter )



### 5.4 Register forta
```
forta register --owner-address $FORTA_OWNER_ADDRESS
```

### 5.5 Start service
```
sudo systemctl enable forta
sudo systemctl start forta
```

### 5.6 Check status. Status should be updated 5-10 minutes after starting node
```
forta status
```

## Usefull commands
Get forta scan node address
```
forta account address
```

Check scan node status
```
forta status
```

Check asociated bots
```
docker ps | grep docker-entrypoint | wc -l
```

Check scan node logs
```
journalctl -u forta -f -o cat
```

stop forta
```
systemctl stop forta
```

start forta
```
systemctl start forta
```

# Testnet rewards
Check your SLA
```
curl https://api.forta.network/stats/sla/scanner/<FORTA_SCANNER_ADDRESS>?startTime=2022-04-24T00:00:00Z | jq .statistics.avg
```

SLA Score will determine if and how much of the reward each scan node gets.
During 75% or more node's running time each week:
- 100% reward: SLA ≥ 0.9
- 80% reward: SLA ≥ 0.75
- No reward: SLA < 0.75

