# Celer-iOS-SDK
# Hackathon doc
https://docs.google.com/document/d/17xTCVwyPqIiSNYr5dR7n3k9wNzGhXzvzwmdSDpUVbtk/edit?usp=sharing

# Add Payment funtionality to your app 

## API Overview

### Start the app and connect to Celer
In this step, Celer does the following things for you: 
* Prepare ETH account
* Create Celer Client
* Join Celer with deposit

Implement the following code when you start your app.
* client = Mobile.createNewCelerClient()
* client.joinCeler()

### Display off-chain balance
Implement the following code when want to display users current balance in UI:
* client.getAvailableBalance()

### Send Payment
Implement the following code on your "send" button click event or UI swipe event.
* client.sendPay(destinationAddress)


## Get started

### Step 1. Prepare an Ethereum account (wallet)

We have a helper to quickly generate wallets for you. If you already have an account or a wallet, you don't need to use our helper. Alternatively, you can use whatever wallet generation tools you like. 

KeyStoreHelper will create a new ethereum account (wallet).

```swift
let keyStoreString = KeyStoreHelper.shared.getKeyStoreString()
let password = KeyStoreHelper.shared.getPassword()
```
Both the keyStoreString and the password will be used to create Celer client in step 3.

### Step 2. Get Celer profile

To connect to an off-chain service provider, you need a server profile which is provided by this off-chain service provider. 
For your quick-test convenience, we have prepared everything you need. You can simply use the hard-coded profile in the sample project. “StoreDir” tells the SDK where to cache the data locally on your device. 

```swift
let datadir = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true)[0]

let config = "{\"ETHInstance\": \"wss://ropsten.infura.io/ws\", \"SvrRPC\": \"osp1-hack-ropsten.celer.app:10000\", \"StoreDir\": \"\(datadir)\", \"SvrETHAddr\": \"f805979adde8d63d08490c7c965ee5c1df0aaae2\", \"ChanAddr\": \"011b1fa33797be5fcf7f7e6bf436cf99683c186d\", \"ResolverAddr\": \"cf8938ae21a21a7ffb2d47a69742ef5ce7a669cc\", \"DepositPoolAddr\": \"658333a4ea7dd461b56592ed62839afc18d54a42\", \"HTLRegistryAddr\": \"a41bf533110e0b778f6757e04cf7c6d2a8e294b1\"}"
```

From the code ,you can see that the profile is a json String for creating Celer client.

```json
{"ETHInstance": "wss://ropsten.infura.io/ws",
"SvrRPC": "osp1-hack-ropsten.celer.app:10000",
"StoreDir": "%1$s",
"SvrETHAddr":  "f805979adde8d63d08490c7c965ee5c1df0aaae2", 
"ChanAddr": "011b1fa33797be5fcf7f7e6bf436cf99683c186d", 
"ResolverAddr": "cf8938ae21a21a7ffb2d47a69742ef5ce7a669cc",
"DepositPoolAddr": "658333a4ea7dd461b56592ed62839afc18d54a42",
"HTLRegistryAddr": "a41bf533110e0b778f6757e04cf7c6d2a8e294b1"}
```

### Step 3. Create a Celer client

From Step 1 and step 2, we got 3 parameters: keyStoreString, password, profile String.

Then we can create a Celer mobile client like this:

```swift
var error: NSError?

let client = MobileNewClient(keyStoreString, password, config, &error)
```
You need to check whether there is any error when creating a new client.

Celer mobile client is your starting point to call almost all the methods you need in Celer SDK. 

### Step 4. Join Celer Network

Joining Celer means entering the off-chain world. To join celer, you need to deposit a certain amount of tokens from your on-chain wallet to Celer's state channel, to make sure that you have some off-chain balance to send to others. Meanwhile, server should also deposit certain amount of tokens to the same channel.  

Remember that we have already generated new account in the first step, this account does not have any on-chain balance yet.
You can transfer some balance from your existing account if you already have some tokens. If you want to get some free testnet tokens, here is a quick tutorial to get free ethers on Ropsten:

* Get some free ETH:
Open this link in your browser https://apitester.com/
Put https://faucet.metamask.io/ in the URL field, and choose http POST method.
![Enter URL](https://s3.us-east-2.amazonaws.com/celer-mobile/Screen+Shot+2018-10-08+at+11.05.22+AM.png)

Put the wallet address in the post data field:
![Enter Address](https://s3.us-east-2.amazonaws.com/celer-mobile/Screen+Shot+2018-10-09+at+5.54.12+PM.png)

Click the "Test" button. Metamask will give you 1 ETH. You can see in the http response the transacton id. 

* Check your balance on Ropsten:
https://ropsten.etherscan.io/address/ Append this URL with your wallet address.

For example:
https://ropsten.etherscan.io/address/0x9f6b03cb6d8ab8239cf1045ab28b9df43dfcc823

It may take a while. Please refresh the above link till you see that the balance is no longer in "pending" status.

Once you have enough balance, you are good to go with this API call:

```swift
// Join Celer Network
do {
  try client?.joinCeler("0x0", amtWei: clientSideAmount, peerAmtWei: serverSideAmount)
} catch {
  print(error.localizedDescription)
}
```

Here, “0x0” represents the Ether token. If you need to use ERC20 tokens, please put the contract address instead.

If this process is successful, you will see the balance in the log. A general failure in this process is “Insufficient fund to join celer”, it means that you need to make sure the wallet has enough on-chain balance before joining Celer.

Joining Celer takes some time because it involves some on-chain transactions. The joinCeler function returns a channel id. If you see this channel id, that means your channel is ready to use.  

Congratulations, you are in the off-chain world.

### Step 5. Send off-chain payment

Now that you have opened the channel, you are able to send some off-chain Ether to someone who has also joined Celer.

How do you know that an address has already joined Celer like youself?

```kotlin
// check if an address has joined Celer Network
do {
  let maxReceivingCapacity = try client?.hasJoinedCeler("0x2718aaa01fc6fa27dd4d6d06cc569c4a0f34d399")
} catch {
  print(error.localizedDescription)
}
```

The name of “hasJoinedCeler” has not been refactored. It doesn’t explicitly mean whether this address has joined celer (like a boolean value). It is a String representing how much you can send to that address. If it is zero, it means this address has not joined Celer.

If the receiverAddress has joined, you can send some tokens to it

```swift
// send cETH to an address
do {
  try client?.sendPay(receiverAddr, amtWei: transferAmount)
} catch {
  print(error.localizedDescription)
}
```

# Multiplayer Mobile Game 

## API Overview

### Step 1. Start the app and connect to Celer
In this step, Celer does the following things: 
* Prepare ETH account
* Create Celer Client
* Join Celer with deposit

Implement the following code when you start your app.
* client = Mobile.createNewCelerClient()
* client.joinCeler()

### Step 2. Start game session, conditionally pay stake to the other parties
In this step, Celer does the following things for you: 
* Create cApp session
* Send payment with conditions

Implement the following code when your game session has just started:
* client.newCAppSession
* client.sendPaymentWithConditions()

### Step 3. When playing the game, send and receive game state
In this step, Celer does the following things for you: 
* Send state to other players
* Receive state from others

Implement the following code when the user is playing the game.
* client.sendCAppState()
* onReceiveState(byte[] state) 


