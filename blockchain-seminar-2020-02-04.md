# Blockchain Seminar 2020/02/04

## Chaincode Development

### Contents

1. Add execute permission to *sh* script file command
2. Run *sh* file 
3. Create Query script
4. Create Invoke script
5. Create Installation & Upgrade script
6. Docker commands

---

### 1. Add execute permission to *sh* script file command

Use `chmod +x <file>` command to give execute permission to script file.

Example:
```
fabric@ubuntu:~$ vi query.sh
fabric@ubuntu:~$ chmod +x query.sh
```

---

### 2. Run *sh* file

Use `./<file>` to run script file.

Example:
```
fabric@ubuntu:~$ ./query.sh
```

---

### 3. Create Query script `vi query.sh`

```bash
docker exec -it cli peer chaincode query -C mychannel -n fabcar -c '{"args":["queryAllCars"]}'
```

---

### 4. Create Invoke script `vi invoke.sh`

```bash
docker exec -it cli peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar -c '{"Args":["createPerson","001","Chhaileng"]}'
```

---

### 5. Create Installation & Upgrade script `vi install-and-upgrade.sh`

```bash
#!/bin/bash

CC_SRC_PATH=/opt/gopath/src/github.com/chaincode/fabcar/javascript

CONFIG_ROOT=/opt/gopath/src/github.com/hyperledger/fabric/peer
ORG1_MSPCONFIGPATH=${CONFIG_ROOT}/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
ORG1_TLS_ROOTCERT_FILE=${CONFIG_ROOT}/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
ORG2_MSPCONFIGPATH=${CONFIG_ROOT}/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
ORG2_TLS_ROOTCERT_FILE=${CONFIG_ROOT}/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
ORDERER_TLS_ROOTCERT_FILE=${CONFIG_ROOT}/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

CC_VERSION=1.2

echo "Installing smart contract on peer0.org1.example.com"
docker exec \
  -e CORE_PEER_LOCALMSPID=Org1MSP \
  -e CORE_PEER_ADDRESS=peer0.org1.example.com:7051 \
  -e CORE_PEER_MSPCONFIGPATH=${ORG1_MSPCONFIGPATH} \
  -e CORE_PEER_TLS_ROOTCERT_FILE=${ORG1_TLS_ROOTCERT_FILE} \
  cli \
  peer chaincode install \
    -n fabcar \
    -v $CC_VERSION \
    -p "$CC_SRC_PATH" \
    -l "node"

echo "Installing smart contract on peer0.org2.example.com"
docker exec \
  -e CORE_PEER_LOCALMSPID=Org2MSP \
  -e CORE_PEER_ADDRESS=peer0.org2.example.com:9051 \
  -e CORE_PEER_MSPCONFIGPATH=${ORG2_MSPCONFIGPATH} \
  -e CORE_PEER_TLS_ROOTCERT_FILE=${ORG2_TLS_ROOTCERT_FILE} \
  cli \
  peer chaincode install \
    -n fabcar \
    -v $CC_VERSION \
    -p "$CC_SRC_PATH" \
    -l "node"

echo "Upgrade smart contract on mychannel"
docker exec \
  -e CORE_PEER_LOCALMSPID=Org1MSP \
  -e CORE_PEER_MSPCONFIGPATH=${ORG1_MSPCONFIGPATH} \
  cli \
  peer chaincode upgrade \
    -o orderer.example.com:7050 \
    -C mychannel \
    -n fabcar \
    -l "node" \
    -v $CC_VERSION \
    -c '{"Args":[]}' \
    -P "AND('Org1MSP.member','Org2MSP.member')" \
    --tls \
    --cafile ${ORDERER_TLS_ROOTCERT_FILE} \
    --peerAddresses peer0.org1.example.com:7051 \
    --tlsRootCertFiles ${ORG1_TLS_ROOTCERT_FILE}
```
---

### 6. Docker commands

Command for login to docker *cli* container
```
fabric@ubuntu:~$ docker exec -it cli bash
```

Command for list installed chaincode (login to *cli* container first to run this)
```
root@666644d9046d:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode list --installed
```

Command for list instantiated chaincode (login to *cli* container first to run this)
```
root@666644d9046d:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode list --instantiated -C mychannel
```