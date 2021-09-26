## BSV Identity Output Protocol (bitIdentity) 0.2

ChangeLog:

version 0.2: 
  + Change the storage of Public Key and Signature from hex string to binary, to save onchain space.
  + new protocol ID:"14kxqYv3emHGwf8m6YgSYLQkGCvn9Qrgr91" to identify the new format
  

### Background

In BSV eco-system, more and more apps are moving away from centrolized system to on-chain system. 
The on-chain protocols usually include a way to identify the user. This protocol is designed to provide a
universal way to attach identity in the BSV transaction. 

### Specification

The Identity data is encoded in a separate OP_RETURN output of the transaction. The data can be verified by any client.
The output structure is:

|     S0 |    S1   |      S2      |     S3     |      S4    |
|--------|---------|--------------|------------|------------|
|OP_FALSE|OP_RETURN|bitIdentity id| public key |  signature |

**bitIdentity ID:** `14kxqYv3emHGwf8m6YgSYLQkGCvn9Qrgr91`  
**old bitIdentity ID:** `14kxqYv3emHGwf8m6YgSYLQkGCvn9Qrgr9` , it will be deprecated.

The data used to sign is transaction dependent. It prevents attackers from reusing the signature.

### JS library
[bitIdentity](https://github.com/bloodchen/bitIdentity)

#### APIs

`const bitID = require('bitidentity')`

**bitID.config(options)**
Setup some options

```
options = {
debug:true, //enable debug, more log
filepayKey:"" //set mattercloud key, used by filepay to generate rawtx
}
```

**bitID.gentx(config)**
Generate rawtx according to the config template

```
//config is the save format as in filepay, with a new protocol property added
const config = {
  to:[
      {protocol:'bitid', value:{
          privateKey:"" //privateKey used to generate signature.
        }
      }
  ]
}
const rawtx = await bitID.gentx(config);
```

**bitID.verifyID(rawtx)**
Verify all identities in the transaction, return false if verify faid or no identity found

```
const verified = bitID.verifyID(rawtx);

```


**bitID.getBitID(rawtx)**
Return an array of bitID objects from rawtx.
```
const BIDs = bitID.getBitID(rawtx);
const pubKey = BIDs[0].publicKey;
const sig = BIDs[0].sig;
```

**bitID.verifyIDFromBitbus(tx)**
Verify identity from bitbus object


**bitID.getBitIDFromBitbus(tx)**
Return an array of bitID objects from bitbus


	
