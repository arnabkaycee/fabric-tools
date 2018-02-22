# General Guidelines for writing Hyperledger Fabric Chaincodes.

I believe smart-contracts (chaincodes) are the heart of any Blockchain Network. Written correctly can yeild many benefits and truly bring out the best in a secure blockchain network and have distratous effects if written inefficiently. Although I will not discuss about any specific goods and bads of Chaincode design, I will touch down a few guidelines which I experienced while doing some POC applications during my work. Without further ado, let's start.

Below are a few general guidelines / caveats that can be adhered to (although there are exceptions) while writing chaincodes. These I have particularly written for chaincodes written for Hyperledger fabric v.1.0 network in golang.  But, I believe they can be extrapolated to chaincodes written in any language for Hyperledger Fabric. 

# Use Chaincode DevMode

To kickoff your development process, use the DevMode. I cannot emphasise enough, how important it is to use dev mode and save time and effort in changing your code and restarting the network again and again.

Reference : https://github.com/hyperledger/fabric-samples/tree/release/chaincode-docker-devmode

*P.S. - Although the tutorial is for Golang, using it for other languages should not be different except for building the chaincode part.* 

# Use Chaincode Logging
Well, this is perhaps the first useful thing that you can do to debug your chaincode and find bugs quickly. Using logging is simple and easy. Use Fabric's inbuilt logger. Fabric provides logging mechanism as follows:

**For Golang:**
https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeLogger
**For NodeJS:**
https://fabric-shim.github.io/Shim.html#.newLogger__anchor
**For Java:**
You can use any standard logging framework like Log4J

# Avoid using Global Keys

While writing chaincode, we often find our hands tied when finding data. To keep track of keys registered in the Key Value Store, we try and use some sort of Global Collection.

For example, when keeping track of registered users in your application, you might want to make a global list and keep adding the list of all the user keys in a Set or Array like structure. But while doing so, you are introducing dependency on a single variable to write to when you add a new user.
This might not seem a problem at first, but underlying is a bug waiting to surprise you when you do concurrent transactions. How? Let me explain.

Hyperledger Fabric uses an Optimistic Locking Model while committing transactions. As you know that first the proposal responses are collected from the endorsing peers by the client and then sent to the Orderer for ordering and finally orderer delivers it to the Committing Peers. 
In this two stage process, if some versions of the keys that you had read in the Endorsement has changed till your transactions reach the  committing stage, you get an MVCC_READ_CONFLICT error. This often is a probability when one or more concurrent transactions is updating the same key.

Reference : https://medium.com/wearetheledger/hyperledger-fabric-concurrency-really-eccd901e4040

[P.S. This is also applicable even when you are not doing concurrent transactions but your block determination criteria is such that one or more transactions updating the same key is landing up in the same block]

# Use Mongo Queries wisely

Mongo Queries [For Couch DB only] is such a boon when searching for data in the Key Value store. But there are a few caveats one needs to take care.

## Mongo Queries DO NOT alter the READ SET of a transaction.

Mongo Queries are for querying the Key Value store aka StateDB only. It does not alter the read set of a transaction. This might lead to phantom reads in the transaction.

## Only the DATA that you have stored in the couchDB is searchable

Do not be tempted to search for a key by its name using the MangoQuery. Although you can access the Fauxton console of the CouchDB, you cannot access a key by querying a key by which it is stored in the database. Example : Querying by **channelName\0000KeyName** is not allowed. It is better to store your key as a property in your data itself.

# Write Deterministic Chaincode

Never write chaincode that is not deterministic. It means that if I execute the chaincode in 2 or more different environments at different times, result should always be the same, like setting the value as the current time or setting a random number.
This is because, that if the read write sets generated are not the same, the Validation System chaincode might reject it and throw an ENDORSEMENT_POLICY_FAILURE.

# Remember to Set Chaincode Execution Timeout

Often it might so happen that during high load your chaincode might not complete its execution under 30s. It is a good practice to custom set your timeout as per your needs. This is goverened by the parameter in the core.yaml of the peer. You can override it by setting the environment variable in your docker compose file :

*Example:* CORE_CHAINCODE_EXECUTETIMEOUT=60s

#  Refrain from Accessing External Resources

Accessing external resources (http) might expose vulnerability and security threats to your chaincode.


