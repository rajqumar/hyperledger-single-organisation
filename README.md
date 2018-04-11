
Hyperledger Composer business network to Hyperledger Fabric blockchain  for a single organization
==
==***Step 1: Starting a Hyperledger Fabric network***==

	/**
	* Fresh Hyperledger fabric installation
	*/
	 cd ~/fabric-tools 
	./stopFabric.sh 
	./teardownFabric.sh 
	./downloadFabric.sh 
	./startFabric.sh
	
	/**
	* Delete any business network cards for safety
	*/
	composer card delete -c PeerAdmin@fabric-network
	composer card delete -c admin@tutorial-network
	
	/**
	* If fails delete the file system card store
	*/
	rm -fr ~/.composer
	

==***Step 2: Exploring the Hyperledger Fabric network***==	

Configuration files - *cryptogen* and *configtxgen*

	/** Cryptogen file*/ 
	~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config.yaml
	
	/** Configtxgen file*/
	~/fabric-tools/fabric-scripts/hlfv11/composer/configtx.yaml


Only one organisation can interact in this network
 
-  Single organisation => Org1
- Domain name => org1.example.com
- Membership Services Provider (MSP) => Org1MSP


**Network components**
The Hyperledger Fabric network is made up of several components:

A single peer node for Org1, named peer0.org1.example.com.
The request port is 7051.
The event hub port is 7053.

A single Certificate Authority (CA) for Org1, named ca.org1.example.com.
The CA port is 7054.

A single orderer node, named orderer.example.com.
The orderer port is 7050 

This runs in a Docker container. Hyperledger composer commands on the Docker host machine.

**Users**
- configured with a user (i.e. admin) named Admin@org1.example.com
- permission to install the code for a blockchain business network onto their organization's peers

> Admin@org1.example.com has a set of certificates and private key files stored in 

>~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

Certificate Authority for Org1 has been configured with admin user
**enrollment ID : admin
enrollment secret : adminpw**

	
**Channel**

composerchannel has been created with peer node peer0.org1.example.com

>You can only deploy Hyperledger Composer blockchain business networks into existing channels, but you can create additional channels by following the Hyperledger Fabric documentation.
>

==***Step 3: Building a connection profile***==


	{
    "name": "fabric-network",
    "x-type": "hlfv1",
    "version": "1.0.0",
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpc://localhost:7051",
            "eventUrl": "grpc://localhost:7053"
        }
    },
    "certificateAuthorities": {
        "ca.org1.example.com": {
            "url": "http://localhost:7054",
            "caName": "ca.org1.example.com"
        }
    },
    "orderers": {
        "orderer.example.com": {
            "url": "grpc://localhost:7050"
        }
    },
    "organizations": {
        "Org1": {
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.org1.example.com"
            ]
        }
    },
    "channels": {
        "composerchannel": {
            "orderers": [
                "orderer.example.com"
            ],
            "peers": {
                "peer0.org1.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                }
            }
        }
    },
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300",
                    "eventHub": "300",
                    "eventReg": "300"
                },
                "orderer": "300"
            }
        }
    }
    }
    
    
==***Step 4: Locating the certificate and private key for the Hyperledger Fabric administrator***==


	~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

 
 public certificate :  signcerts subdirectory and is named Admin@org1.example.com-cert.pem
 private key to sign transactions : keystore subdirectory and named like this 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk
 
 ==***Step 5: Creating a business network card for the Hyperledger Fabric administrator***==	

>Business network card = connection profile + certificate + private key
> run composer card create command to do it

	composer card create -p connection.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk -r PeerAdmin -r ChannelAdmin
	
	/** card generation result would be PeerAdmin@fabric-network.card 
		-p connection.json (path to the connection profile)
		-u PeerAdmin (referenced to Admin@org1.example.com)
		-c Admin@org1.example.com-cert.pem (path to the certificate file)
		-k 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk (path to private key)
		-r PeerAdmin -r ChannelAdmin (roles PeerAdmin (ability to install chaincode) and ChannelAdmin (ability to instantiate chaincode))		
	*/
	
	  
==***Step 6: Locating the certificate and private key for the Hyperledger Fabric administrator***==

Hyperledger Composer can only use business network cards that are placed into a wallet. The wallet is a directory on the file system that contains business network cards.

	/**
	command to import the business network card into the wallet
	-f PeerAdmin@fabric-network.card (path to bnc)
	*/
	composer card import -f PeerAdmin@fabric-network.card 
	
then follow hyperledger composer developer tutorials to generate **business network archive (.bna)**

  
==***Step 7: Installing the Hyperledger Composer business network onto the Hyperledger Fabric peer nodes***==

Hyperledger Fabric terms, the Hyperledger Composer runtime is a standard chaincode.

Install the Hyperledger Composer runtime onto all of the Hyperledger Fabric peer nodes. In Hyperledger Fabric terms, this is a chaincode install operation.

	composer network install -c PeerAdmin@fabric-network -a tutorial-network@0.0.1.bna

==***Step 8: Starting the blockchain business network***==
Hyperledger Fabric terms, this is a chaincode instantiate operation.

	composer network start --networkName tutorial-network --networkVersion 0.0.1 -A admin -S adminpw -c PeerAdmin@fabric-network
	
	
We can interact with this business network using the business network card file **admin@tutorial-network.card** that was created

==***Step 9: Importing the business network card for the business network administrator***==

Import the business network card into the wallet and start interacting with the running blockchain business network

	composer card import -f admin@tutorial-network.card

==***Step 10: Testing the connection to the blockchain business network***==
Test the connection to the blockchain business network

	composer network ping -c admin@tutorial-network
