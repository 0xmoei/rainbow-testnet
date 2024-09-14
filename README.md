<h1 align="center">🌈 Rainbow Protocol Indexer Testnet</h1>

![1500x500](https://github.com/user-attachments/assets/f90e5ca5-43bb-4864-b12d-893c31858928)


![Screenshot_260](https://github.com/user-attachments/assets/5c0b7d53-31c7-4823-ae5e-9b03405ce75a)


## System Requirements (Minimum-Recommended)
| Ram | cpu     | disk                      |
| :-------- | :------- | :-------------------------------- |
| `2 GB`      | `4 Core` | `40 GB SSD` |

## 1- Install Dependecies
```console
sudo apt update && sudo apt upgrade -y 
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```

## 2- Install docker
```console
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker version check
docker --version
```

## 3- Install Bitcoin Core
```console
mkdir -p /root/project/run_btc_testnet4/data
git clone https://github.com/rainbowprotocol-xyz/btc_testnet4
cd btc_testnet4
```

### Optional: Change ports of your Bitcoin Core node
You can continue without changing your default config in `docker-compose.yml` but if you need to change the ports to avoid conflicts, you can change the ports like following example: `5001:5000`

## 4- Run Bitcoin Core node
**1- Start Bitcoin Core:**
```
docker compose up -d
```

**2- Enter container:**
```
docker exec -it bitcoind /bin/bash
```

**3- Create new wallet:**
```
bitcoin-cli -testnet4 -rpcuser=demo -rpcpassword=demo -rpcport=5000 createwallet test
bitcoin-cli -testnet4 -rpcuser=demo -rpcpassword=demo -rpcport=5000 loadwallet test
bitcoin-cli -testnet4 -rpcuser=demo -rpcpassword=demo -rpcport=5000 getnewaddress
```

**4- Exit the container:**
```
exit
```

## 5- Connect Bitcoin Core and Run Indexer
1. **Download and set up the indexer:**
```console
cd $HOME
git clone https://github.com/rainbowprotocol-xyz/rbo_indexer_testnet.git && cd rbo_indexer_testnet
wget https://github.com/rainbowprotocol-xyz/rbo_indexer_testnet/releases/download/v0.0.1-alpha/rbo_worker
chmod +x rbo_worker
```

2. **Start a screen:**
```
screen -S rbo
```

3. **Start Indexer:**
```
./rbo_worker worker --rpc http://127.0.0.1:5000 --password demo --username demo --start_height 42000
```
![image](https://github.com/user-attachments/assets/83f5ba74-a90f-43ab-be07-41c0bb9288d3)

* `Ctrl+A+D` to minimze the screen

## 6- Verify Your Setup

1. **Copy your pricipal ID**:
```
cat ./identity/principal.json
```

2. **Log in to the [Testnet Explorer](https://testnet.rainbowprotocol.xyz/explorer)**:

   - Input the Principal ID and submit the details.

![image](https://github.com/user-attachments/assets/6594bbe4-9dcb-4019-b21d-578e0510267b)


## 7- Save your private key
```console
cat ./identity/private_key.pem
```
* Or just save the `./identity/private_key.pem` file in your local system

## Usefull Commands:
Return to screen
```
screen -r rbo
```

Kill screen
```
screen -XS rbo quit
```

## 8- Upgrade Indexer
* 1- Return to your rbo screen
* 2- stop  it with `CTRL+C`
* 3- Run the follwoing commands
```console
# Change 5050 to the port you customized beore or don't touch it
cd $HOME && /bin/netstat -ntpl | grep 5050 | awk '{print $NF}' | awk -F"/" '{print $1}' | xargs -r kill -9  

# Replace with the new rbo_worker
wget https://storage.googleapis.com/rbo/rbo_worker/rbo_worker-linux-amd64-0.0.2-20240914-4ec80a8.tar.gz && tar -xzvf rbo_worker-linux-amd64-0.0.2-20240914-4ec80a8.tar.gz

cp rbo_worker-linux-amd64-0.0.2-20240914-4ec80a8/rbo_worker rbo_indexer_testnet/rbo_worker

# Start the new one
cd rbo_indexer_testnet && chmod +x rbo_worker

./rbo_worker worker --rpc http://127.0.0.1:5000 --password demo --username demo --start_height 42000
```

## ERROR:
if you get into you just need to rerun your indexer using step 5.3 and with the latest height of your bitcoin core node ( just check its logs using `docker ps -a` & `docker logs -f CONTAINER_ID` )

Or check this faq
https://github.com/rainbowprotocol-xyz/rbo_indexer_testnet/blob/main/FAQ.md
