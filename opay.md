## OPay: Easy to use advanced BSV payment Library

![](https://images.unsplash.com/photo-1569284160951-aee94fa3c489?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60)

### Features:

1. Transactions are made from the user's wallet directly
2. Support advance BSV payment through [Open Payment Protocol](payment.md)
  * make OP_RETURN transactions
  * specify multiple inputs and outputs
  * specify locking and unlocking script
  * sign data using wallet
3. Support to use NBdomain as payment receiver
4. Support multipal wallets - [Maxthon VBox](https://maxthon.com), [Volt Wallet](https://volt.id), [Dot Wallet](https://dotwallet.com), more are coming...
5. Support web payment and mobile payment(QR code).
6. Easy to customize UI, can be embeded to other Apps.

### How to use?

Include `<script src="https://unpkg.com/opay@latest/opay.min.js"/>` in the `<HEAD>` element of your page.  

### API

**1. async opay.init(options)**:Initialize `opay` object. 

```
options = {
  containerID:"containerID", //the container element that hosts wallet UI. Only useful when no wallet is specified
  timeout:100, //default timeout, seconds
  wallet:null    //specify the wallet to use. Currently can be 'vbox' or 'dot' . If it's ommited or null, the wallet UI will appear 
}
opay.init(options);

```


**2. opay.request(req,callback)**:make a OPP call

**req**: Request object. Please refer to [Open Payment Protocol](payment.md) for details 
Please note, there are [some differences](opay.md?id=differents-between-opp-protocol-and-opay) 

**call_back**:Callback to receive result, result is in JSON format

**return value**:   

The `callback` can either return `false` or a json object.  
`false`: means the app will handle `ack_url` by itself.  
JSON object: means opay will return it to `ack_url` automatically.


```
{
  code:0, // (required) 0 for success, non 0 for error
  message:"" //optional error message,
  broadcasted:true //(optional)when only rawtx returned by wallet, it tells the wallet if the rawtx is broadcasted or not.  
}
```

**3. opay.hash256(data)**:return Hash256 of data


```
const data = "hello";
const strHash = opay.hash256(data);
```
**data**: the same format as in [Buffer.from(...)](https://nodejs.org/api/buffer.html)

**4. opay.verify(strHash,strSig,strPubKey)**:verify the signature of strHash against public key

### Differents between OPP protocol and OPay

To make it more easier for JS developers, there are some differences between OPP protocol and OPay.

1. Some properties of requestBody are made optional and will have a default value.
    
| property   |      default value      | 
|----------|-------------|
| v |  2 |
| id |    Date.now().toString()   |
| expire | Date.now() + (timeout ? timeoutx1000: 5x60x1000) |
| app | {name:"OPay APP"} |
|product | "Product"|

2. `data` property of OPP object is JSON object instead of string.

3. return object passed via `callback` is JSON object instead of string.

4. the APP can return a JSON object from `callback` function to let opay handle `ack_url` automatically, or return `false` and
handle `ack_url` by itself.

5. opay has extended `to` array to support `data` property for easy composing of an OP_RETURN output.

check below example for usage.

### Code Example:

1. Sign and Verify signature:


```
let reqBody = { signer:address, data_hash:hash1 }
console.log(reqBody);
let req = { sign_request:
            { data:reqBody}
}
opay.request(req,(e)=>{
          const retObj = e;
          const signed = retObj.body;
          const verified = opay.verify(hash1,signed,strPubkey);
          alert(verified?'login successful':'login failed');
          return {code:0}
    });
```
2. Save data onchain:

```
let reqBody = {
            v: 2,
            app: { name: "Demo"},
            id: (Date.now()).toString(),
            product: "P1",
            expire: Date.now()+120*1000,
            to:[
              {data:["Hello world",1234]}
            	],
            broadcast: true
        };
        let req = {
        	pay_request:{
        		data:reqBody
        	}
        }
        opay.request(req,(e)=>{
        	let result = e;
        	console.log(result);
          return {code:0}
        });
```    
3. get a raw transaction with an identity:

```
let reqBody = {
            to:[
              {protocol:"bitid"},
              {data:["Hello world",1234]}
            	],
            broadcast: false
        };
        let req = {
        	pay_request:{
        		data:reqBody
        	}
        }
        opay.request(req,(e)=>{
        	let result = e;
        	console.log(result);
          return {code:0}
        });
```    
4. Pay some Vpoint to someone

```
let reqBody = {
            to:[
              {address:'abcd.b',value:1000},
              {address:'kkkk.b',value:2000},
            	],
            broadcast: true
        };
        let req = {
        	pay_request:{
        		data:reqBody
        	}
        }
        opay.request(req,(e)=>{
        	let result = e;
        	console.log(result);
          return {code:0}
        });
```

### Others

**1. opay.bsv**

Exposed bsv library object

### Live examples

[vboxapp](https://vboxapp.glitch.me)  
[nbdomain](https://app.nbdomain.com)

### Discuss OPP and OPay Lib

[telegram](https://t.me/nbdomain)