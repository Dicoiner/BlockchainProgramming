## Stratis address {#stratis-address}

You know that your **Stratis Address** is what you share to the world to get paid.  
![](../assets/StratisAddress.png)  
You probably know that your wallet software uses a **private key** to spend the money you received on this address.  
![](../assets/PrivateKey.png)  

The keys are not stored on the network and they can be generated without access to the Internet.  

With code:  
```cs  
Key privateKey = new Key(); // generate a random private key
```  
From the private key, we use a one-way cryptographic function, to generate a **public key**.  

![](../assets/PrivKeyPubKey.png)  
```cs 
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```  

There are two Stratis **networks**: 
* **TestNet** is a Stratis network for development purposes. Stratis on this network worth nothing.  
* **MainNet** is the Stratis network everybody uses.  

> **Note:** You can acquire testnet coins quickly by using **faucets**, just google "get testnet stratis".  

You can easily get your **stratis address** from your public key and the **network** on which this address should be used. 

![](../assets/PubKeyToAddr.png)  

```cs 
Console.WriteLine(publicKey.GetAddress(Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

**Precisely a stratis address is made up by a version byte (different on both networks) and your public key’s hash bytes concatenated then encoded into Base58Check:**  

![](../assets/PubKeyHashToBitcoinAddress.png)  

```cs 
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
```  

> **Fact:** The hash of the public key is generated by performing a SHA256 hash on the public key, and then performing a RIPEMD160 hash on the result, with Big Endian notation. The function could look like this: RIPEMD160(SHA256(pubkey))  

The Base58Check encoding has some neat features, such as checksums to prevent typos and a lack of ambiguous characters such as '0' and 'O'.  
The Base58Check encoding of an address also make sure that a user of a stratis wallet don't send money to an address that should be used in a different network.

```cs 
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

> **Tip:** Practicing Stratis Programming on MainNet makes mistakes more memorable.  
