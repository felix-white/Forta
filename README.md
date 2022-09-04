# Manual node setup
If you want to setup fullnode manually follow the steps below

Official documentation:

>[Run a Scan Node](https://docs.forta.network/en/latest/scanner-quickstart/)

Explorer:

>https://explorer.forta.network/network

# Scan Node Requirements
The following are the requirements for running a Forta scan node.

- 64-bit Linux distribution
- CPU with 4+ cores
- 16GB RAM
- Connection to Internet
- 100GB SSD (in addition to full node requirements)
- Recommended: Full node (any chain)

# Preparation
>Create new metamask wallet and fund it with 1 MATIC in Polygon network. This address will be used as forta owner address.

### Synchronize system time
```
sudo systemctl enable systemd-timesyncd
sudo systemctl start systemd-timesyncd
timedatectl status
```
### Update packages
```
sudo apt-get update
```
# Docker
### 1.Install docker
```
sudo apt-get install ca-certificates curl gnupg lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
docker --version
```

### 2.Configure Docker
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
# Install Forta
### 1.Initialize Forta Directory
```
forta init --passphrase <your_passphrase>
```
### 2.Configure systemd
```
tee /etc/systemd/system/fortad.service > /dev/null <<EOF
[Unit]
Description=Forta
After=network.target

[Service]
Environment=FORTA_DIR=/root/.forta
Environment=FORTA_PASSPHRASE=<your_passphrase>
User=root
Type=simple
ExecStart=forta run
Restart=on-failure
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
### 3.Configure Chain APIs
```
tee /root/.forta/config.yml > /dev/null <<EOF
chainId: 137

scan:
  jsonRpc:
    url: https://polygon-rpc.com/

trace:
  enabled: false
EOF
```
### 4.Send 1 Matic in Polygon network to your scan node address
    
### 5.Register Scan Node
forta register --owner-address <forta_owner_address> --passphrase <your_passphrase>
### 6.Stake FORT
You can do it according to this [guide](https://docs.forta.network/en/latest/stake-on-scan-node/).
To be a part of the Forta Network, a scan node should have at least 500 FORT (2500 since Sep 30th) of active stake staked on its behalf.
You need approve FORT firstly and than stake FORT.

### 7.Start Forta via systemd
```
sudo systemctl start fortad
sudo systemctl status fortad
sudo systemctl enable fortad
```
### 8.Check Forta status in 5-10 minutes
```
forta status
```
