MaxthonMaxthon# Open Payment Protocol Docs <small>0.7</small>

Changelog

|  vesion | changes     |
|-------|---------------------|
| 0.7   | + add ack_url in wallet response |
| 0.6   | - removed base64 encode of parameters<br/>protocol version upgraded to 2|
| 0.5   | + add `inputs` to support customized UTXO and unlocking script |
| 0.4   | + add  `protocol`->`refund` in `to` attribute<br/>- removed `refund_to` attribute |
| 0.3   | + add `protocol`->`bitIdentity` in `to` attribute<br/>- removed `payer` attribute |
| 0.2   | + add `script` in `to` attribute<br>- removed `data` attribute|
| 0.1   | Initial version |

## General

**Open Payment Protocol - OPP** is an extension of [BIP270](https://github.com/bitcoin-sv-specs/bips/blob/master/bip-0270.mediawiki), with some issues fixed and some enhancements, which are:

1. **Attach the Identity to a transaction:** 
  With the genesis update, a transaction can have non-standard locking script and unlocking script, which makes the old way to
  determind the identity (through publick key in inputs) vulnerable to malicious attacks. OPP supports attaching a separate identity
  to the transaction. [bitIdentity Protocol](identity.md "bitIdentity Protocol").

2. **Customized refund address:**
  The wallet can define a customized refund address in the transaction for the calling app to fetch. Since the refund address is
  in the transaction itself, it will be recorded in the blockchain, thus provide a good tracking record.

3. **Specified UTXO:**
  The app can use **OPP** to specify the UTXOs it wants to use.
  
4. **Customized locking and unlocking script**:
  The app can use **OPP** to generate transactions with customized locking and unlocking script.
  
## Scenario

**OPP** is used for **the application** to talk to **the wallet** to initiate bitcoin transactions. It assumes the wallet holds users'
private key and the application does not have access to the private key.

## Implementations

### JS Object

maxthon6 has created a build-in JS object - **VBox** , to expose OPP API.

Currently, VBox object only has one function, 

`VBox.Request(req,callback)` 

**req**: JSON object of **OPP**.

**callback**: callback function to handle result   

**Note**: Request function can only be called from **HTTPS** pages or local pages( mainly for testing ).

### QR code

A QR code can be generated to embed the JSON object of **OPP**.

**Wallets that support OPP API embeded in QR code:**

* [Maxthon6 mobile browser](https://www.maxthon.com)
* [Volt Wallet](https://volt.id)

### HTTP endpoint

A HTTP endpoint can handle a post call with `body` set to the JSON object of **OPP**.

**Wallets that support OPP API through HTTP API:**

* [Dot Wallet](https://dotwallet.com)
 

## Specifications

### JSON Object

**OPP** is defined by the JSON object.


### Payment API

**req**
```
{
   "pay_request":{                 //command
      "data":reqBody, // string of reqBody object JSON.stringify(reqBody)
      "url":"https://example.com/request/123" //the url will return string of requestBody
                }
}
```
Note: caller shall choose to use either `data` or `url`, but not the both.  `data` is used to handle small amount of data and
`url` is used to handle large amount of data. If the caller wants to support multiple wallet, url is the recommanded way.

If the caller wants to support multiple wallet, it can generate request and encoded it into a QR image for users to scan.

#### RequestBody Format

|  name | required | type  | example  | comments  |
|-------|----------|-------|----------|-----------|
|  v | yes  | int  | 2     | protocol version  |
| app| yes  | JSON |  {<br>"name":"NBdomain","logo":"http://nbdomain.com/logo.png"<br>} | logo(96*96 png)|
| id | yes  |string| 20200402095801666666 | the unique order id inside the app. It will be returned by callback |
| signer|no   |string|19YawJS5tLtQk4XUPyyXzM98H5ceWx4KPJ047d32a8544a1960d37d0634|Signer address. Used to specify the payer|
| to | no   |array |[<br/>{"protocol":"bitIdentity","value":{"public_key":"..."}}"<br/> {"address": "...","value": 500},<br/> {"script": "...","value": 500}<br/>]|A list of payment receiver. It shall be address or script. The amount shall be bigger than 600 satoshi.|
| input|no  |array |see notes| set customized UTXO and unlocking script|
|notify_url| no | url |https://www.maxthon.com/notify| URL to accept result |
|broadcast| no | bool| false | broadcast or just sign. default: false|
|product|yes|string|For rent | name of Payment Item |
|detail| no | Bob's 2019 rental fee | comments of payment item |
|expire | yes | int | Data.now() | the exact time that this order will expire. format: unix time, milliseconds|
|app_data|no | string | anything | the data that the app wants to attach to the order. It will be returned by callback or notify |

**notes**:

1. `expire` is required when `broadcast` is set to false. 
2. [filepay](https://github.com/MatterCloud/filepay) can be used to generate OP_RETURN script .
3. for `bitIdentity` protocol, please check [bitIdentity Protocol](identity.md "bitIdentity Protocol")
4. `bitIdentity` can have an `address` in value which means the identity with `address` shall be put in the output. 
    However, if there is no `address` specified, which means any identity can be put in the output.
5. **ACK return**: callback or notify_url shall return an ACK message or call `ack_url` to acknowledge the acceptance of result.

Example:
```
{
  code:0, //0 for success, non 0 for error. 
  id:1573665155,
  message:"" //errror message
}
```


#### Input property
App can specify an `input` property to specify which UTXOs are must. The VBox will use these UTXOs first, if they are not enough,
the VBox will use other UTXOs in user's Identity. The format of  `input` looks like:

```
...
reqBody.input = [
  {
    "txid":"6d3ca9cadcd0ae32d0628404e9dd73a7eecfe7f54ad137fd41d0d37c38e7c258",
    "index": 1,
    "value": 700,
    "lockingScript":"006a223139664c7054354c70614d474b754c665556716d4e64586b566365713272626a796e09736578796c61647931",
    "unlockingScript":["47304402207645a9167ebb5d24268ea037d","{OP_SIG_HASHALL}","2207645a9167ebb5d24268ea037d","{OP_SIG_HASHANY}"],
    "privateKeyAddress":"" //specify the address of the private key used to sign this input. If not specified, the payment privateKey will be used
  }
]
...

```



There are several template opcodes which are supposed to be replaced with coresponding data by wallet.

"{OP_PUBKEY}": public key of the signing private key
"{OP_PUBKEYHASH}": public key hash of the signing pivate key
"{OP_SIG_HASHALL}": signature which is signed by the signing private key, signHashType=0x01
"{OP_SIG_HASHNONE}": signature which is signed by the signing private key,signHashType=0x02
"{OP_SIG_HASHSINGLE}": signature which is signed by the signing private key,signHashType=0x03
"{OP_SIG_HASHANY}": signature which is signed by the signing private key,signHashType=0x80


#### Output protocol

There is a `protocol` type output supported in `to` property. It defines something shall be done by the wallet.
Currently below protocol are supported:

1. `bitIdentity`: If the app wants to attach an identity to the transaction, it can define an `bitIdentity` output and the wallet
    will put an identity public key into the output.
    
    **Format:** 
    ```
    {"protocol":"bitIdentity","value":{"public_key":"publicKey of Identity"}}  
    ```
    **Output:** For more details about how to handled it, please check [bitIdentity](identity.md "bitIdentity Protocol")  

  
2. `refund`: If the app supports refund money to users, it can define a output with `refund` protocol. The wallet will put 
    an address that the refund shall go to.
    **Format:** 
    ```
    {"protocol":"refund"}
    ```
    There shall be only one `refund` protocol output, if multiple refund protocols are set, only the 1st is considered valid. 
      
    **Output:** The wallet shall put two strings in this output. The resulting output shall look like
    |OP_FALSE|OP_RETURN|refund|%address%|
    |--------|---------|---------|-------------|

#### Payment Response

The Response of a request will be retuned both by `callback` and `notify_url` (if notify_url is set) . The response data will be a JSON string of the response object.
The JSON string shall be `POST`ed to the `notify_url` as `body`, with header includes `Content-Type: application/json` .

A simple payment requeset flow :
```
let req = {
  payment_request:{
    data: JSON.stringify(requestBody)
  }
}
VBox.Request(req, function(res){
  let obj = JSON.parse(res);
  if(obj.code==0){ //success
    
  }
  const act = {
  code:0,
  message:""
  }
  return act;
});
```


#### Response format


|  name | required | type  | example  | comments  |
|-------|----------|-------|----------|-----------|
|  id | yes  | string  | 20200402095801666666     | order id  |
| app_data | yes | anything | Addtional data attached by the App |
| message | yes | string | SUCCESS | Error message |
| message_info|no|string |missing to attribute|descriptive error message|
| code | yes | int | error code. 0 for success |
| rawtx | yes | string | | rawTx |
| ack_url | yes | string | https://url.com/ack | ?ack=JSON.stringify(ackmsg) |
| txhash | yes | string | hash of the transaction |

**Error Message** 


|  name | description | solution  |
|-------|----------|----------|
| SYSTEM ERROR|system error | retry |
|PARAM_ERROR | order parameter error | check parameter|
|ORDERTIMEOUT| user doesn't pay before timeout| none |
|ORDERCLOSED| user cancelled the order | none |
|SIGN_ERROR | Signature Error | check parameter |
|SEND_ERROR|error sending tx | try again|
|USER_NONE|no such user |when a payer or signer is specified and it's not in vbox|
|FREQUENCY_LIMITED|too much calls | make less calls|
|RULELIMIT| The transaction has risks| contact us |
|SYSTEM_MAINTENANCE| System maintence | wait |

#### App ACK response
After the APP get notifications from wallet, it shall reply with an ACK response and send to `ack_url`: 
` http://ack_url?ack=ack_msg `.

The response is a JSON string.
```
{
  code:0, //0 for success, non 0 for error
  id:135545454，
  message:"success" //detailed error message
}
```

### Signing API
The Signing API uses [ECDSA](https://docs.moneybutton.com/docs/bsv-ecdsa.html) to sign non-transactioinal data.
The data shall be hashed(sha256) first and pass the data-hash to API. The hash will be hashed again and use it to generate signature.
The app shall hash the data-hash to veify against signature returned.

**req**
```
{
   "sign_request":{                 //protocol identifier
      "data":reqBody, // string of reqBody object JSON.stringify(reqBody)
      "url":"https://example.com/request/123" //the url will return string of requestBody
                }
}
```

#### RequestBody Format

|  name | required | type  | example  | comments  |
|-------|----------|-------|----------|-----------|
|  v | yes  | int  | 1     | protocol version  |
| app| yes  | JSON |  {<br>"name":"NBdomain","logo":"http://nbdomain.com/logo.png"<br>} | logo(96*96 png)|
| id | yes  |string| 20200402095801666666 | the unique order id inside the app. It will be returned by callback |
|detail| no | Sign for xxx | comments of payment item |
|signer|no   |string|19YawJS5tLtQk4XUPyyXzM98H5ceWx4KPJ047d32a8544a1960d37d0634|Signer address. Used to specify the payer|
|notify_url| no | url |https://www.maxthon.com/notify| URL to accept result |
|data_hash|yes|hashstring|hash256 of the some data|
|app_data|no | string | anything | The data that the app wants to attach to the order. It will be returned by callback or notify |

#### Signing Response

The format is the same as payment response.

A simple signing requeset flow :
```
let req = {
  sign_request:{
    data: JSON.stringify(reqBody)
  }
}
VBox.Request(req, function(res){
  let obj = JSON.parse(res);
  if(obj.code==0){ //success
    
  }
  const act = {
  code:0,
  message:""
  }
  return act;
});
```
#### Response format


|  name | required | type  | example  | comments  |
|-------|----------|-------|----------|-----------|
|  id | yes  | string  | 20200402095801666666     | order id  |
| app_data | no | anything | Addtional data attached by the App |
| message | yes | string | SUCCESS | Error message |
| ack_url | yes | string | https://url.com/ack | ?ack=JSON.stringify(ackmsg) |
| code | yes | int | error code. 0 for success |
| body | yes | string | | signed string |

### Misc Tool API

**req**
```
{
   "tool_request":{                 //protocol identifier
      "data":reqBody, // string of toolBody object JSON.stringify(toolBody)
      "url":"https://example.com/request/123" //the url will return string of toolBody
                }
}
```
`toolBody` is in JSON format:

```
const toolBody = {
  "cmd":"...",
}
```

#### **balance:** Get balance of an address
```  
  {
    "id":"id_of_order", //order id
    "cmd":"balance", //command name
    "address":"",    //the address of which balance will be get. "optional". If it's missing, the wallet can ask use to select an address.
  }
```
return:
```
  {
    "id":"id_or_order",
    "code":0, //0 for success
    "message":"" // Error message
    "value":10000 // Number of SATs
  }
```
#### **gen_hash:** Generate a hash using privateKey and seed string. 
```
const result = hash256(hash256(privateKey)+seed);

```
```  
  {
    "id":"id_of_order", //order id
    "cmd":"gen_hash", //command name
    "seed":"abcd",    //seed of the key. The same seed will result in the same output
  }
```
return:
```
  {
    "id":"id_or_order",
    "code":0, //0 for success
    "message":"", // Error message
    "value":"..." // hash result
  }
```


#### Typical Process

![](https://cdn.glitch.com/88b5ee04-8b7d-4187-88c8-5cac946553ba%2FUML%E6%97%B6%E5%BA%8F%E5%9B%BE%20(1).jpg?v=1604632089340)

**Demo Code**

```
var data = 'this is my data that i want to sign'
let hash = bsv.crypto.Hash.sha256(Buffer.from(data));
let signBody = {
  app:"NBdomain",
  id:"Ruc2Rmaml1ZHlmdWhkZm5rZGZqa3V",
  "data_hash": hash,
};

let config = {
sign_request:{}
}
config.sign_request.data = JSON.stringify(signBody);

VBox.Request(config, function(msg) {
  let obj = JSON.parse(msg);
  if(obj.code==0){ //success
    const hash1 = bsv.crypto.Hash.sha256(Buffer.from(hash.toString(),'hex'));
    var verified = bsv.crypto.ECDSA.verify(hash1, obj.body, publicKey)
    console.log(verified)
  }
  const act = {
  code:0,
  message:""
  }
  return act;
);
```

## Adopte OPP API in the wallet and the App

Since VBox API is open and has implemented almost all required features, it will be cost effective to just use it in wallets and apps.

### The benefits of implementing this API in wallets:

* Leverage a set of tested APIs instead of designing new ones.
* Be compatible with APPs that already support the API. *All bitcoin apps in Maxthon6 browser will use this API.*
* Create a developer friendly eco-system by providing a similar set of APIs.
* Keep up with the latest updates to all protocol changes. 
* Keep up with all security update with OPP.

### The benefits of implementing this API in apps:

* Leverage the users of supporting wallets, like maxthon6 and the volt wallet.
* Save time in designing a payment protocol which is not easy.
* Being supported by other wallets, instead of creating an internal wallet which is complex and insecure.
* Be more trustworthy by not having access to user's private key.

### Related Libraries

[OPay JS Library](opay.md "OPay Lib")
