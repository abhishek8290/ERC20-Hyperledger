ERC_20 on hyper ledger 

Enviroments needs to be installed first 

Node v 8.9.3 and v 8.11.4
Hyper ledger fabric version 1.2
Docker latest
Python 2.7.12
Go go1.9.3
Curl 7.47.0

Prerequisites needs to be downloaded are from this link :- https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html

Once you are ready, and in the directory into which you will install the Fabric Samples and binaries, go ahead and execute the following command:-  

curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2

Open terminal and run these command :

cd path/to/repository/folder
cd network
./buildERC20TokenNetwork.sh up

It created the required certificates for each entity of HyperLedger using the crypto-config.yaml file, in a folder named crypto-config within your networks directory. Check it out!
It also created channel.tx, genesis.block, Org1MSPanchors.tx and Org1MSPanchors.tx.

It also created docker containers and volumes for:
peer0 and peer1 or Org1
peer0 and peer1 of Org2
orderer
cli
chaincode
To enter into the CLI container run

docker exec -it cli bash
You’ll see something like this 

root@0e2b84a5cedc:/opt/gopath/src/github.com/hyperledger/fabric/peer#

This function will return the owner of the token contract.
peer chaincode query -C mychannel -n mycc -c ‘{"Args":["getOwner"]}'

Org1MSP

This function will return the name of our token contract.
peer chaincode query -C mychannel -n mycc -c ‘{"Args":["getName"]}'

Simple Token



This function will return the symbol for our token contract.

peer chaincode query -C mychannel -n mycc -c ‘{"Args":["getSymbol"]}'

SMT

This function will return the total supply for our token contract. It defaults to 0 until it is set once.
peer chaincode query -C mychannel -n mycc -c ‘{"Args":["getTotalSupply"]}'

0

This function returns the value of isMintingAllowed boolean stored on HyperLedger. It defaults to undefined until it is set once.

peer chaincode query -C mychannel -n mycc -c ‘{"Args":["isMintingAllowed"]}'

undefined

This getter returns the value of allowance set by a token owner for a spender MSPID. It takes as Input the MSPID token owner as first argument and MSPID of spender as second argument. It defaults to 0 until it is set once.

peer chaincode query -C mychannel -n mycc -c '{"Args":["getAllowance", "Org1MSP", “Org2MSP"]}'

0
As you can see getAllowance is now, 0. It will return float once set later. check for the other combination we have and see if it returns 0

peer chaincode query -C mychannel -n mycc -c '{"Args":["getAllowance", "Org2MSP", “Org1MSP”]}'

0
Our last getter is getBalanceOf function, it returns the token balance of every MSPID we enter. It also defaults to 0 if the MSPID don't have any token balance.

peer chaincode query -C mychannel -n mycc -c '{"Args":["getBalanceOf", “Org1MSP"]}'

0
peer chaincode query -C mychannel -n mycc -c '{"Args":["getBalanceOf", “Org2MSP"]}'

0

Assuming your config is set to peer0 of org1, otherwise set it using the following commands:

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/users/Admin@org1.techracers.com/msp

export CORE_PEER_ADDRESS=peer0.org1.techracers.com:7051

export CORE_PEER_LOCALMSPID="Org1MSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt



Now let's try to update our minting state to true so run:

peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c ‘{“Args":["updateMintingState","true"]}'

INFO 001 Chaincode invoke successful. result: status:200

Check if it’s true 

peer chaincode query -C mychannel -n mycc -c ‘{"Args":["isMintingAllowed"]}'

true

This function can be used to create/mint tokens by the token owner. But isMintingAllowed should be set to true so run:

peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["mint","Org1MSP", “100.2345"]}'
INFO 001 Chaincode invoke successful. result: status:200

You can check the balance using our getter:
peer chaincode query -C mychannel -n mycc -c '{"Args":["getBalanceOf", "Org1MSP"]}'
100.2345

transfer
Now we know that we have 100.2345 tokens registered under Org1MSP. Let's try to transfer 10 tokens to Org2MSP.

peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["transfer","Org2MSP", “10"]}'

INFO 001 Chaincode invoke successful. result: status:200

You can check Org2's balance using:
peer chaincode query -C mychannel -n mycc -c '{"Args":["getBalanceOf", "Org2MSP"]}'
10

updateTokenName
peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["updateTokenName","TECH COIN"]}'

INFO 001 Chaincode invoke successful. result: status:200

Check it using:
peer chaincode query -C mychannel -n mycc -c '{"Args":["getName"]}'
TECH COIN

updateTokenSymbol
peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["updateTokenSymbol","TEC"]}'

INFO 001 Chaincode invoke successful. result: status:200

Check it using:
peer chaincode query -C mychannel -n mycc -c '{"Args":["getSymbol"]}'
TEC

updateApproval
If you want some other MSPID to spend some tokens on your behalf you can use this setter.
peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["updateApproval","Org2MSP", "30"]}'

INFO 001 Chaincode invoke successful.
Check it using:
peer chaincode query -C mychannel -n mycc -c '{"Args":["getAllowance", "Org1MSP", "Org2MSP"]}'
30


transferFrom
Once you have approved Org2 to transfer on behalf of Org1. First set the config in cli for Org2, so you can call functions on its behalf.

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/users/Admin@org2.techracers.com/msp

export CORE_PEER_ADDRESS=peer0.org2.techracers.com:7051

export CORE_PEER_LOCALMSPID="Org2MSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt

Now to transfer a float value to a non existent, but valid MSPID.

peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["transferFrom","Org1MSP", "UndefinedOrgMSP", "29.8989"]}'

INFO 001 Chaincode invoke successful. result: status:200
Check it using:
peer chaincode query -C mychannel -n mycc -c '{"Args":["getBalanceOf", "Org1MSP"]}'

60.3356

peer chaincode query -C mychannel -n mycc -c '{"Args":["getBalanceOf", "Org2MSP"]}'

10

transferOwnership
Lastly set your config back to Owner of token and try transfering Token Ownership.
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/users/Admin@org1.techracers.com/msp

export CORE_PEER_ADDRESS=peer0.org1.techracers.com:7051

export CORE_PEER_LOCALMSPID="Org1MSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt

peer chaincode invoke -o orderer.techracers.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/techracers.com/orderers/orderer.techracers.com/msp/tlscacerts/tlsca.techracers.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.techracers.com/peers/peer0.org1.techracers.com/tls/ca.crt --peerAddresses peer0.org2.techracers.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.techracers.com/peers/peer0.org2.techracers.com/tls/ca.crt -c '{"Args":["transferOwnership","Org2MSP"]}'


 INFO 001 Chaincode invoke successful. result: status:200

Check it using:
peer chaincode query -C mychannel -n mycc -c ‘{"Args":["getOwner"]}'

Org2MSP







