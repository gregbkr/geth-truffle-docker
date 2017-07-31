## Description
This compose will give you:
- ethereum go-client (geth) running on port 8540 with
  - A default account, already unlocked
  - Connected to devchain (network 2017042099)
  - Mining the blocks
  - Data are saved out of the container on your host in `/root/.ethereum`, so no problem if you delete the container
- A testrpc eth-node running a test network on port 8545
- Truffle: where you can test and deploy a simple smart contract
- Netstats: which will collect and send your node perf to our devchain dashboard on http://factory.shinit.net:15000

## Prerequisit
- A linux VM, preferable ubuntu 14.x or 16.x
- [Docker](#docker) v17 and [docker-compose](#docker-compose) v1.13 
- This code: `git clone https://github.com/gregbkr/geth-truffle-docker.git devchain && cd devchain`

## Run containers

- Change "yourNodeName" in `docker-compose.yml` 
`echo "Give a node name :"; read n ; sed -i -- "s/yourNodeName/$n/g" docker-compose.yml ; echo "Your node name is: $n"`
- Run the stack: `sudo docker-compose up -d`
- Check geth is up and answering locally: `curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' localhost:8540`
- Check testrpc node is running: `curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' localhost:8545`

## Geth
- Check container logs: `sudo docker logs -f geth`
- Start shell in the geth container: `sudo docker exec -it geth sh` 
- Type folowing commands
  - List account: `geth --datadir=/root/.ethereum/devchain account list`
  - Run this command: `cat /root/.ethereum/devchain/keystore/*`
  - Backup the account somewhere safe. For example, I saved this block for my wallet (carefull, you can steal my coins with these info):
`{"address":"6e068b2fcf3ed73d5166d0b322fa10e784b7b4fe","crypto":{"cipher":"aes-128-ctr","ciphertext":"0d392da6deb66b13c95d1b723ea51a53ab58e1f7555c3a1263a5b203885b9e51","cipherparams":{"iv":"7a919e171cda132f375afd5f9e7c2ba1"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"1f3f814262b9a4ce3c2f3e1cabb5788f0520101f00598aa0b84bbda08ceaaf31"},"mac":"8e8393e86fe2278666ec26e9956b49adc25bc2e7492d5a25ee30e8118dd17441"},"id":"71aa2bfd-ee91-4206-ab5e-82c38ccd071f","version":3}/`
- The account is on your vm too, exit docker `exit` then type `sudo ls /var/lib/docker/volumes/devchain_geth/_data/devchain`. From here you can save or import another account

If needed, create new account with:
- Create password: `echo "Geneva2017" > /root/.ethereum/devchain/pw2`
- create account: `geth --datadir=/root/.ethereum/devchain account new --password /root/.ethereum/devchain/pw2`

- Don't want to mine? In `docker-compose.yml` section `geth/command` remove `--fast --mine` and run again `docker-compose up -d`
- Check that you are well connected to our netstat dashboard: http://factory.shinit.net:15000
- Use python to check your node, and later send ether: `python3 checkWeb3.py`

## Truffle
- Go in truffle container:  `sudo docker exec -it truffle sh`
- try truffle:
```
cd /dapp
truffle init
vi truffle.js
  host:"testrpc" <-- edit this line
  port: 8545     <-- edit this line

truffle test
```
- And same test with `host:geth port 8540` --> if warning: `authentication needed: password or unlock` --> you need to unlock your wallet!

Now test our helloWord dapp
- Navigate to the folder: `cd /dapp/helloWorld`
- Test the contract against the testrpc mode: `truffle test --testrpc`
- Test against our devchain node: `truffle test --devchain`

### Usefull Truffle commands
- Access to truffle console: `truffle console --network devchain`
  - `web3.fromWei(web3.eth.getBalance(web3.personal.listAccounts[0]))`
  - `web3.personal.unlockAccount(web3.personal.listAccounts[0], "password", 150000);`
- Send contract to devchain `truffle migrate --network devchain`
  - See your contract adress: `Greeter.deployed()`
  - Launch a new contract: `var greeter = Greeter.new("Hello gva")`
  - `Greeter.at('0x5a2676_____YOUR_CONTRACT_ADRESS_____9e7fb42d').greet()`
  - View balance: `web3.eth.getBalance(web3.personal.listAccounts[0]).toNumber()`
  - Send transaction: `web3.eth.sendTransaction({from:web3.personal.listAccounts[0], to:'0x41df2990b4efd225f2bc12dd8b6455bf1c07ff6d', value: web3.toWei(10, "ether")})`

## Install guide
### Docker
```
wget https://get.docker.com/ -O script.sh
chmod +x script.sh
sudo ./script.sh
```
check docker version `sudo docker version`

[Back to Prerequisit](#prerequisit)

### docker-compose
Replace 1.15.0 with latest version available on https://github.com/docker/compose/releases 
```
sudo -i
curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
exit
```
check docker-compose version `docker-compose version`
[Back to Prerequisit](#prerequisit)
