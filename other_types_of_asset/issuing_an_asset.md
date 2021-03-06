## Issuing an Asset {#issuing-an-asset}

### Objective {#objective}

For the purpose of this exercise, I will emit **BlockchainProgramming coins**.  

You get **one of these BlockchainProgramming coins** for every **0.004 stratis** you send me.  
**One more**  if you add some kind words.  
Furthermore this is a great opportunity to make it in to the [Hall of The Makers](http://n.stratis.ninja/). 

Let’s see how I would code such feature.

### Issuance Coin {#issuance-coin}

In Open Asset, the Asset ID is derived from the issuer's **ScriptPubKey**.  
If you want to issue a Colored Coin, you need to prove ownership of such **ScriptPubKey**. And the only way to do that on the Blockchain is by spending a coin belonging to such **ScriptPubKey**.

The coin that you will choose to spend for issuing colored coins is called “**Issuance Coin**” in **NStratis**.  
I want to emit an Asset from the book stratis address: [1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://www.smartbit.com.au/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB).

Take a look at my balance, I decided to use the following coin for issuing assets.  

```json
{
          "transactionId": "eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b",
          "index": 0,
          "value": 2000000,
          "scriptPubKey": "76a914c81e8e7b7ffca043b088a992795b15887c96159288ac",
          "redeemScript": null
} 
```  

Here is how to create my issuance coin:  

```cs
var coin = new Coin(
    fromTxHash: new uint256("eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b"),
    fromOutputIndex: 0,
    amount: Money.Satoshis(2000000),
    scriptPubKey: new Script(Encoders.Hex.DecodeData("76a914c81e8e7b7ffca043b088a992795b15887c96159288ac")));

var issuance = new IssuanceCoin(coin);
```  

Now I need to build transaction and sign the transaction with the help of the **TransactionBuilder**.  

```cs
var nico = StratisAddress.Create("15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe");
var bookKey = new StratisSecret("???????");
TransactionBuilder builder = new TransactionBuilder();
            
var tx = builder
    .AddKeys(bookKey)
    .AddCoins(issuance)
    .IssueAsset(nico, new AssetMoney(issuance.AssetId, quantity: 10))
    .SendFees(Money.Coins(0.0001m))
    .SetChange(bookKey.GetAddress())
    .BuildTransaction(true);

Console.WriteLine(tx);
```  

```json
{
  …
  "out": [
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 356facdac5f5bcae995d13e667bb5864fd1e7d59 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.01989400",
      "scriptPubKey": "OP_DUP OP_HASH160 c81e8e7b7ffca043b088a992795b15887c961592 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4f410100010a00"
    }
  ]
}
```  

You can see it includes an OP_RETURN output. In fact, this is the location where information about colored coins are stuffed.

Here is the format of the data in the OP_RETURN.  

![](../assets/ColorMaker.png)  

In our case, Quantities have only 10, which is the number of Asset I issued to ```nico```. Metadata is arbitrary data. We will see that we can put an url that points to an “Asset Definition”.  
An **Asset Definition** is a document that describes what the Asset is. It is optional, we are not using it in our case. (We’ll come back later on it in the Ricardian Contract part.)  

For more information check out the [Open Asset Specification](https://github.com/OpenAssets/open-assets-protocol/blob/master/specification.mediawiki).

After transaction verifications it is ready to be sent to the network.  

```cs
Console.WriteLine(builder.Verify(tx)); 
```  

### With QBitNinja
```cs
var client = new QBitNinjaClient(Network.Main);
BroadcastResponse broadcastResponse = client.Broadcast(tx).Result;

if (!broadcastResponse.Success)
{
    Console.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Success!");
}
```  

### Or with local Stratis core

```cs  
using (var node = Node.ConnectToLocal(Network.Main)) //Connect to the node
{
    node.VersionHandshake(); //Say hello
    //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, tx.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(tx));
    Thread.Sleep(500); //Wait a bit
}
```

My Stratis Wallet have both, the book address and the “Nico” address.  

![](../assets/NicoWallet.png)  

As you can see, Stratis Core only shows the 0.0001 Stratis of fees I paid, and ignore the 600 Stratis coin because of spam prevention feature.

This classical Stratis wallet knows nothing about Colored Coins.  
Worse: If a classical Stratis wallet spend a colored coin, it will destroy the underlying asset and transfer only the Stratis value of the **TxOut**. (600 satoshi)

For preventing a user from sending Colored Coin to a wallet that do not support it, Open Asset have its own address format, that only colored coin wallets understand.  

```cs
nico = StratisAddress.Create("15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe");
Console.WriteLine(nico.ToColoredAddress());
```

```
akFqRqfdmAaXfPDmvQZVpcAQnQZmqrx4gcZ
```  

Now, you can take a look on an Open Asset compatible wallet like Coinprism, and see my asset correctly detected:  

![](../assets/Coinprism.png)  

As I have told you before, the Asset ID is derived from the issuer’s **ScriptPubKey**, here is how to get it in code:  

```cs
var book = StratisAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var assetId = new AssetId(book).GetWif(Network.Main);
Console.WriteLine(assetId); // AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e
```  
