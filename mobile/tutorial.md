# Quickstart

It will be much easier to understand how everything works if you understand how Jasonette works. So watch the Youtube video first at https://www.jasonette.com

Also, remember the entire app is set up to run on Rinkeby testnet, so don't forget to use rinkeby address for everything.


## Step 1. Launch App

1. Download Jasonette: http://docs.jasonette.com/#quickstart
2. Set up the app with the JSON url: https://gliechtenstein.github.io/erc20/mobile/app/main.json
3. Run!

You should see the main UI.

## Step 2. Import your Rinkeby private key

To sign in with your rinkeby address:

1. Export your private key
2. Create a QR code from the private key: You can do it at https://www.qr-code-generator.com/ or any other qr code generator
3. From the app, press the key button at the bottom right corner. This should send you to the QR code scanner
4. Scan it and it should ask for a password. Enter the password and it will encrypt the wallet using the password, and return to the main view.
5. At this point you should see the top row has changed to "signed in", along with your public key value.

## Step 3. Send a token

To send an ERC20 token to another rinkeby address,

1. Get another rinkeby address (or generate one) from metamask
2. Press the green "Transfer Token" button
3. It will take you to another QR code screen.
4. From the metamask wallet, select "Show QR Code" and it should show you the QR code.
5. Scan it and it should ask you for the password you entered earlier to encrypt the wallet.
6. Once you enter the password, it should take a bit to decrypt the wallet, sign the transaction, and then broadcast it to the network. Finally it will bring you back to the main screen. Done!

## Step 4. Try alternative UI

You can try the alternative UI at https://gliechtenstein.github.io/erc20/mobile/app/simple.json


# How the Mobile App works

First, it's easier to understand everything if you remember that each JSON file is a view (or gets mixed into another JSON to form a view). It's pretty much like HTML in that everything centers around a "page", except that every "page" is a native view.

There are three folders: 

- app: contains the main app view JSON markup
- qr: QR code scanner markup
- wallet: JSON markup for handling private keys, signing, and broadcasting to rinkeby network.

Let's go through one by one.

## /app

### main.json

![erc20](https://gliechtenstein.github.io/images/erc20.gif)

This is the main user interface.

First of all, it's important to notice that we're reusing the web app in the mobile app. You can see the web app at [https://gliechtenstein.github.io/erc20/web](https://gliechtenstein.github.io/erc20/web) is being loaded as an agent:

```
{
  "$jason": {
    "head": {
      "title": "Main e20",
      "agents": {
        "erc20": { "url": "https://gliechtenstein.github.io/erc20/web" }
      },
      ...
```

You can view the code for the web app under [web/index.html](web/index.html).

In this case we're using the same web app as an agent for simplicity, but you don't have to. You can write a minimal HTML specifically for agent usage. 

Also we're loading the agent from the remote https URL, but you can also [package the HTML file locally and load it over `file://` url scheme](http://docs.jasonette.com/offline/#1-loading-from-local-file).



Now let's go through the code.


When the app loads (`$show` or `$foreground`) it triggers a `balance` action.

The `balance` action looks like this:

```
"balance": [{
  "{{#if 'wallet' in $global}}": {
    "type": "$agent.request",
    "options": {
      "id": "erc20",
      "method": "call",
      "params": [ "balanceOf", [ "{{$global.wallet.address}}" ] ]
    }
  }
}],
```

It looks for a global variable named `wallet`. If such a variable exists, it means we've imported a wallet already (This app is just an MVP so it only supports one private key under `$global.wallet`)

Then it makes an `$agent.request` into an agent named `erc20`, which is defined under: `$jason.head.agents.erc20`, which links to [https://gliechtenstein.github.io/erc20/web](https://gliechtenstein.github.io/erc20/web).

The `options` object passed to the `$agent.request` action is the actual JSON-RPC request.

This request calls a function named `call`, and passes `"balanceOf"` and the wallet's address (public key) if there is one.

Then you look into the agent DApp's `call` function.

If you look inside the `call` function, it basically makes a request to Ethereum via infura node. And when it successfully fetches the content, it triggers a `fetched` event with the response as a payload https://github.com/gliechtenstein/erc20/blob/master/web/index.html#L104

["fetched"](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L9) is a user defined custom event. And in this case it sets the local variables with the event payload from the agent, and then goes on to the next success callback action which is [$render](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L13).

The $render action renders the template under [$jason.head.templates.body](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L70) with all the data in the current execution stack, which includes the local variable we've just set.

Just to highlight some of the template details:

1. It renders a wallet address if signed in (`$global.wallet` exists) https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L96 but displays "not signed in" if the variable doesn't exist yet.
2. It loops through the local variable attributes we just set from querying Ethereum via `call()` function https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L102 and displays a vertical layout of two label components https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L102
3. Also it creates a button component that says "transfer", and attaches an action handler which will trigger an action called "transfer" when tapped: https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L115

You can find the user-defined `"transfer"` event under `$jason.head.actions.transfer` https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L28


Let's look into what happens when the `"transfer"` event is triggered.

The first thing is an `$href` to another view: https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L28

It will send us to [publickey.json](#publickey.json) which will let the user scan a public key QR code, and send it back to the current view once done.

With the return value from the QR code scanner view, the action call chain continues on. It goes to the next `success` callback, which is [$util.alert](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L28). It asks for how many tokes you want to send.

When you answer, it will try to find an agent with and ID of [eth](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L49) and try to make a function call into it (the [transaction](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L50) function)

If you look at the JavaScript code, the [transaction](https://github.com/gliechtenstein/erc20/blob/master/web/index.html#L30) function takes a method name and calls it using the web3.js library. Since we're passing the parameters "transfer" and the user input token count from above step, the agent will try to make a "transfer" method call into the contract we instantiated from the web3.js library.

The DApp agent [will look for `window.$agent`](https://github.com/gliechtenstein/erc20/blob/master/web/index.html#L54), and since this was loaded as a Jasonette agent, it will go to execute `getData` to export the transaction and send it back to the native app: https://github.com/gliechtenstein/erc20/blob/master/web/index.html#L58

Now let's not forget that we haven't even yet touched the user's private key or signed anything, not to mention broadcasting to Ethereum. We only have an unsigned raw transaction at this point.

So we call the next action which opens a new view ([wallet.json](https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L54)). This wallet agent is where all the private key handling will happen. It's intentionally split into these two steps (`getData` to get a raw transaction, and then send it to the wallet agent for actual signing and broadcasting) because we want to reuse this in the future.

Now we head over to [wallet.json](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.json), the first thing that happens is the `$load` event https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.json#L9 and in this case it asks the user to enter the password used to encrypt the wallet (if the user already has imported a wallet, but if not, we will discuss how the app also has a QR code scanner for importing private keys below).

After taking the password, it passes the unsigned transaction along with the password into the [wallet agent](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.html), and calls the function [send](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.json#L20).

Now, if we look into the wallet.html agent, the [send](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.html#L60) function basically decrypts the wallet with the password, signs the transaction and broadcasts it to Ethereum (Infura).

After that it again calls [$agent.response](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.html#L84) which returns control back to the parent app, and the app goes on to display the "success!" banner https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.json#L29 and goes back to the previous view via [$back](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.json#L34).

And that's about it for how the app displays Ethereum data, and how you can send tokens.


### simple.json

![erc20ethan](https://gliechtenstein.github.io/images/erc20ethan.gif)

This is just an alternative interface, used to demonstrate how easy it is to fork an app and build a custom one.

Simply load this JSON instead of the `main.json` from above, and you'll get the default interface.

I made this by tweaking the `main.json` a bit. Here are the two customizations I made:

- The UI is just a simple red button that says "Send".
- Instead of scanning a receiver's public key through QR code, it has a receiver address hardcoded (mine), to demonstrate transactions can be easily customized as well.

https://github.com/gliechtenstein/erc20/blob/master/mobile/app/simple.json

---

## /qr

This folder contains all the modules for scanning QR codes.

There are three files:

- [scanner.json](qr/scanner.json): This is the core QR code module which we will use as a stub. This file won't be used directly but will be mixed into the following two files for custom QR code scanning (See [mixins](http://docs.jasonette.com/mixin/) to learn how this works). This file gets mixed in to the following two files [here](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/publickey.json#L2) and [here](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/privatekey.json#L2).
- [privatekey.json](qr/privatekey.json): This is a sandboxed view that scans a private key as QR code and adds to a global variable named [wallet](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/privatekey.json#L33), after which it will call the [$back action](http://docs.jasonette.com/actions/#back) to [return back to the previous view](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/privatekey.json#L36) which called this module
- [publickey.json](qr/publickey.json): This is another sandboxed view inherited from [scanner.json](qr/scanner.json), but in this case it's not for importing the user's private key. Instead it's used to scan soemone else's public key for sending. It calls an [$ok](http://docs.jasonette.com/actions/#ok) action which returns the control back to the previous view but also [with a return value](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/publickey.json#L4)

### scanner.json

This is the core QR scanner module, but is not meant to be used as a standalone. It's meant to be used as a mixin. Think of it as sort of an "abstract class".

It's designed so that it requires a self mixin.

If you look under https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/scanner.json#L13 you'll see that there's a self-mixin. The `$document.onscan` attribute means, when any JSON that uses the `scanner.json` as a mixin loads, it will look for the root level `onscan` attribute (the `$document` means the root level of the entire JSON tree) and automatically replace the `"@": "$document.onscan"` with the found subtree. I'll explain this further in the following sections where both `privatekey.json` and `publickey.json` employs this appraoch to "inherit" from this JSON file.

Also note that there's a template (`$jason.head.templates.body`) which gets [rendred automatically at view load event](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/scanner.json#L16) 

The rendered body would include a `camera` type background which displays a full screen camera as background. And the camera will trigger `$vision.ready` event once initialized.

### privatekey.json

The [onscan](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/privatekey.json#L3) attribute gets mixed in under [$vision.onscan](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/scanner.json#L13) through [self-mixin](http://docs.jasonette.com/mixin/#self-mixin).

After it gets mixed in, the result would look something like this:

```
...
"actions": {
  "$vision.ready": {
    "type": "$vision.scan"
  },
  "$vision.onscan": {
    "type": "$set",
    "options": {
      "imported": "{{$jason.content}}"
    },
    "success": {
      "type": "$util.alert",
      "options": {
        "title": "Enter Password",
        "description": "Enter a password to encrypt the wallet",
        "form": [
          {
            "name": "password",
            "type": "secure"
          }
        ]
      },
      "success": {
        "type": "$agent.request",
        "options": {
          "id": "eth",
          "method": "encrypt",
          "params": [
            "{{$get.imported}}",
            "{{$jason.password}}"
          ]
        },
        "success": {
          "type": "$global.set",
          "options": {
            "wallet": "{{$jason}}"
          },
          "success": {
            "type": "$back"
          }
        }
      }
    }
  },
  "$load": {
    "type": "$render"
  }
 ...
```


1. First, the `$vision.ready` event gets triggered, and the app triggers `$vision.scan`.
2. The camera starts scanning for barcodes. And when it detects one, it will trigger `$vision.onscan`, whose final markup looks like what we saw above.
3. When we look under `$vision.scan`, it sets a local variable named `imported` through the `$set` action. We need this local variable further down along the way.
4. Next, its `success` callback is executed. The next action is `$util.alert`. It opens an alert dialog to ask user input.
5. The user enters a password and presses OK, and the `success` callback is triggered.
6. The next action that's called as the success callback is the `$agent.request`. It makes a JSON-RPC call into the `eth` agent we described [here](https://github.com/gliechtenstein/erc20/blob/master/mobile/qr/scanner.json#L6), which initializes with a web app we wrote [here](https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.html)
7. The wallet.html agent calls the `encrypt` function with the two parameters passed in.
8. The `encrypt` function takes the private key and the password, encrypts it, and returns the result through `$agent.response` api.
9. The `$agent.response()` from the JS side then calls the next `success` callback where we left off when we called the `$agent.request`. The next action is `$global.set`.
10. Here we set an app wide global variable named `wallet` with the encrypted content we received from the agent from the previous step.
11. Finally, all jobs are done so we call `$back`, which takes the view back to the previous view which called this view.


### publickey.json

This file is much simpler than `privatekey.json`. This represents a view which scans a public key QR code, and sends it back to the previous view as a return value.

We use the `$ok` action to go back one step with a return value.

---

## /wallet

This folder contains `wallet.json` for native UI, and `wallet.html` for an agent specifically designed to deal with user private key, signing, broadcasting, etc.

Most of what they do have been explained above while explaining the `/app` folder.

