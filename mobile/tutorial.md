# Quickstart

It will be much easier to understand how everything works if you understand how Jasonette works. A quick way to catch up on Jasonette:

1. Read the tutorial at: https://medium.freecodecamp.org/how-to-build-cross-platform-mobile-apps-using-nothing-more-than-a-json-markup-f493abec1873
2. Watch the intro video on basics of Jasonette markup syntax: http://docs.jasonette.com/#d-learn-jason-syntax

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


# Overview

Here's how the app is structured:

![structure](https://gliechtenstein.github.io/images/eth_structure.png)

Above diagram is a quick overview of the overall data flow.

- The user interacts with the native UI. 
- The native UI makes a request to the “DApp container” (which contains your web3 DApp)
- And the DApp container makes a request to the “Wallet container” (for now, just think of it as a mobile equivalent of MetaMask)
- The Wallet container then connects to Ethereum

All of these are inter-connected through the JSON-RPC protocol, and the application — all three modules — is entirely described as a JSON markup.

![markup](https://gliechtenstein.github.io/images/jasonette_markup.png)

Now let’s take a look at each module in detail.

> Note:
>
> If this is the first time you’ve heard of Jasonette, I recommend reading this quick intro to Jasonette first:

- Read tutorial: [How to build cross-platform mobile apps using nothing more than a JSON markup](https://medium.freecodecamp.org/how-to-build-cross-platform-mobile-apps-using-nothing-more-than-a-json-markup-f493abec1873)
- Watch video tutorial on JASON syntax: [Building a native app in 12 minutes with Jasonette](http://docs.jasonette.com/#d-learn-jason-syntax)

## 1. Let's build a DApp container in JSON

To integrate your existing DApp into the native app using Jasonette, you simply embed the DApp into the native app as a sandboxed HTML/JavaScript execution context ([“agent”](http://jasonette.com/agent)). It’s almost like embedding as an IFRAME but the difference is:

1. You’re embedding it into a mobile app, not into another web page.
2. Unlike an iframe, a 2-way communication channel is established by default

Declaring an agent involves simply adding 3 lines of JSON to the existing app markup:

```
{
  "$jason": {
    "head": {
      "title": "Web3 DApp in a mobile app",
      "agents": {
        "eth": {
          "url": "https://gliechtenstein.github.io/erc20/web"
        }
      },
      ...
    }
  }
}
```

We initialize the DApp container by loading from https://gliechtenstein.github.io/erc20/web and name it eth, which will be used as the ID when we make JSON-RPC calls in the future.

![agent initialize](https://gliechtenstein.github.io/images/jasonette_initialize.png)

Once initialized, we can [make JSON-RPC calls into this DApp container](http://docs.jasonette.com/agents/#3-make-json-rpc-calls) and [natively handle events](http://docs.jasonette.com/actions/#action-registry) triggered from the DApp container, all by using just a JSON markup. The overall event & data flow will look like this:

![agent communication](https://gliechtenstein.github.io/images/agent_communication.png)

Here’s the overview of above diagram:

1. The user ONLY interacts with the native UI. To the user the DApp is invisible and it simply functions as a data source. **(Step 1)**
2. The native module forwards the user request to the DApp container through JSON-RPC **(Step 2)**
3. The DApp container then makes a request to Ethereum network **(Step 3)**, and when it gets a response back **(Step 4)**, forwards it back to the parent native app **(Step 5)**

For example, we can make a JSON-RPC call into our DApp container every time the view is displayed, by listening to a `$show` event and making an `$agent.request` action call (Step 2 from the diagram):


```
{
  "$jason": {
    "head": {
      ...
      "actions": {
        "$show": [{
          "{{#if 'wallet' in $global}}": {
            "type": "$agent.request",
            "options": {
              "id": "eth",
              "method": "call",
              "params": [ "balanceOf", [ "{{$global.wallet.address}}" ] ]
            },
            "success": {
              ...
            }
          }
        }],
        ...
```

Above markup will first check if a global variable named wallet exists (`"{{#if ‘wallet' in $global}}"`). If it does, it means the user has already imported a wallet previously so we go on to the next step, the `"$agent.request"` action.

> Quick Primer on Jasonette Actions: 
>
> Actions are like functions, expressed in JSON. All [“actions”](https://docs.jasonette.com/actions/) in Jasonette are described with at most four attributes: [`"type"`, `"options"`, `"success"`, and `"error"`](https://docs.jasonette.com/actions/#syntax).
>
> `"type"` is the name of the action (sort of like an API method), `"options"` is the parameters you pass to it, `"success"` is the callback that gets executed when the action succeeds. `"error"` is the error callback that gets run when something goes wrong. The `"success"` and `"error"` callbacks can be nested to create a sequence of instructions.


The above JSON markup describes “Find an agent named eth (which we initialized from above), then find a JavaScript function named call from the agent. And finally, run it by passing the arguments `"balanceOf"` and `["{{$global.wallet.address}}"]` which evaluates to the wallet address if the user has already imported one.
So if the wallet address was `"0xabcdefghijklmnop"`, above JSON markup will make a JavaScript function call:

```
call("balanceOf", ["0xabcdefghijklmnop"])
```

If we look into the agent to see what’s going on inside the call JavaScript function, it would look something like this:

```
var call = function( ... ) {
  ...
  contract.balanceOf.call(function(err, result) {
    ...
  })
}
```

The DApp requests a [balanceOf method call](https://theethereum.wiki/w/index.php/ERC20_Token_Standard#Token_Balance) to the deployed ERC20 token contract and waits for the response with a callback function. (**Step 3** and **step 4** from the diagram)

When we get the response back from Ethereum network, we need to send it back to the native app who requested it in the first place through $agent.request API. We do this through the built-in [$agent.response](http://docs.jasonette.com/agents/#2-agentresponse) method **(Step 5)** which gets injected to every agent on load.

```
contract.balanceOf.call(function(err, result) {
  ...
  $agent.response(result);
})
```

Note that we’re basically using our DApp as a “backend” to the mobile app instead of rendering to its own DOM. The `$agent.response` JavaScript function will return the result back to the $agent.request action which triggered off everything. We’ll take a look at how we deal with this in the next section.

## 2. Let’s Build the Native UI in JSON

Now that we have the data from the `$agent.response` method call from the previous section, we are all ready to render it. Let’s return back to the original `$agent.request` action. Previously we only looked at the options object, but this time we will focus on the `"success"` callback which gets triggered when the agent responds through the `$agent.response` JavaScript method.

```
{
  "$jason": {
    "head": {
      ...
      "actions": {
        "$show": [{
          "{{#if 'wallet' in $global}}": {
            "type": "$agent.request",
            "options": {
              "id": "eth",
              "method": "call",
              "params": [ "balanceOf", [ "{{$global.wallet.address}}" ] ]
            },
            "success": {
              "type": "$set",
              "options": "{{$jason}}",
              "success": {
               "type": "$render"
              }
            }
          }
        }],
        ...
```

In the `"success”` callback we see another action ["$set"](http://docs.jasonette.com/actions/#set). This action is used to set a local variable.

We also see a template expression `"{{$jason}}"`. That expression would evaluate to the value returned from the agent via `$agent.response`.

> Quick Intro: How do return values work?
>
> In Jasonette whenever an action is run, its return value is always returned as $jason inside the next `"success"` callback action.

In this case if the return value was `{ "balanceOf": 100000 }`, that will be the value of `$jason` when the `$set` action is executed. The template expression would evaluate to the following result:

```
...
"type": "$set",
"options": {
  "balanceOf": 100000
},
"success": {
  "type": "$render"
}
...
```

The $set action will set the local variable `"balanceOf"`’s value to `100000`.

After the local variable is set, it goes onto the next success callback action ["$render"](http://docs.jasonette.com/actions/#render). The `$render` action renders the template with the current data on the stack. The `$render` action automatically looks for a JSON template under `$jason.head.templates.body` and renders it if it finds one.

The rendering is a two step process:

1. **Parse:** Select the data JSON in the current stack and transform it with a template JSON
2. **Render:** Interpret the transformed result into native UI layout.

The first step is “parsing” and it’s done by a library called [SelectTransform](https://selecttransform.github.io/site/) — built into Jasonette as a dependency—which is kind of like [Handlerbars](https://handlebarsjs.com/), but instead of using an HTML template, it uses a JSON template to transform one JSON to another:

![ST](https://selecttransform.github.io/site/src/st.gif)

After the parsing is complete, the second step is to actually render the parsed markup into a native layout. The Jasonette rendering engine takes the parsed result from step 1 and interprets it and turns it into native layout according to the syntax you can find here:

- [Jasonett View Markup Syntax](http://docs.jasonette.com/document/#body)

The native layout engine is flexible enough to describe most of what you would want in a native app, purely in a single JSON markup. Here are some places where you can check out public examples:

- [Jasonette Example Gallery](http://docs.jasonette.com/examples/)
- [Jasonette Web](http://web.jasonette.com/)

For our MVP app, we will serve it from a remote JSON hosted on github. The template starts here: https://github.com/gliechtenstein/erc20/blob/master/mobile/app/main.json#L70

The end result would look something like this:

![erc app](https://gliechtenstein.github.io/images/jasonette_erc.png)

## 3. Let’s Build a QR Code Scanner in JSON

From the app you will notice the key icon at the bottom right corner. When you press it, it should take you to a full screen camera screen that looks like this:

![jasonette qr code scanner](https://gliechtenstein.github.io/images/jasonette_qr.jpeg)

That’s because there’s an [`$href` action](https://docs.jasonette.com/actions/#href) attached to that button, which sends you to the URL that contains another JSON markup that self-constructs into the QR code scanner view:

```
...
{
  "type": "image",
  "url": "https://png.icons8.com/ios-glyphs/100/ffffff/key.png",
  "action": {
    "type": "$href",
    "options": {
      "url": "https://gliechtenstein.github.io/erc20/mobile/qr/privatekey.json",
      "options": {
        "caption": "Import private key by scanning QR code"
      }
    }
  }
},
...
```

This QR code scanner view is also described entirely as a JSON markup. If you look at the link https://gliechtenstein.github.io/erc20/mobile/qr/privatekey.json you will see that it starts with:

```
{
  "@": "https://gliechtenstein.github.io/erc20/mobile/qr/scanner.json",
  "onscan": {
    ...
```

The `"@"` attribute is called [Mixin](https://docs.jasonette.com/mixin/), and it lets you mix in another JSON into the current JSON at load time. Basically when this JSON document is loaded, it immediately makes another request to fetch the JSON at https://gliechtenstein.github.io/erc20/mobile/qr/scanner.json and mixes all its root attributes into the current JSON tree. You can learn more about mixins here:

- [Mixin Documentation](https://docs.jasonette.com/mixin/)
- Mixin Tutorial Series
  - Remote mixin: http://blog.jasonette.com/2017/02/27/mixins/
  - Self mixin: http://blog.jasonette.com/2017/03/02/self-mixin/

After all the mixin has been resolved we are left with a JSON that mostly looks like the [scanner.json](scanner.json) but with the `"onscan"` attribute mixed into `$jason.head.actions.$vision.onscan`.

- The background is set to `"type": "camera"` 
- The “import private key by scanning QR code” caption is a floating label layer

And we also listen for QR code scan event using the `$vision.onscan` event. Here’s what happens when the QR code is scanned:

1. The `$vision.onscan` event is triggered
2. It asks for a user input password using `$util.alert`.
3. It calls an agent function to encrypt the scanned private key with the password.
4. Take the encrypted wallet and set it as `$global.wallet` variable
5. Return back to the main view, thanks to [$back](https://docs.jasonette.com/actions/#back) action.

You can learn more about QR code scanning here:

- Tutorial: Build a QRCode/Barcode scanning app with 26 lines of JSON: https://medium.com/@gliechtenstein/build-a-qrcode-barcode-scanning-app-with-26-lines-of-json-b83453d39197
- Documentation: http://docs.jasonette.com/actions/#vision

## 4. Let’s Build a Wallet Container in JSON

We have so far learned how to query Ethereum, how to import a private key using QR code. Now it’s time to actually spend money from the wallet we imported.

Writing to Ethereum is trickier than reading. We must be more careful because it deals with creating actual transactions and sending real money.

Normally when building a regular DApp, we use the web3.js library to make an API call like this:

```
contract.transfer.sendTransaction(receiver, tokens, {
  to: contract.address,
  gasLimit: 21000,
  gasPrice: 20000000000
}, function(err, result) {
  // Render the DOM with result
})
```

This method sendTransaction actually does two things:

1. Create an encoded transaction object for a contract method called "transfer".
2. Sign and broadcast the transaction object to Ethereum via JSON-RPC.

For our project, instead of having the DApp handle both, we will:

1. Let our DApp container handle only the first part of encoding a transaction
2. and create a new separate container called “Wallet container” to handle the second part

By separating the two, the DApp container doesn’t have to deal with private keys. Instead it delegates it to the wallet container just like how MetaMask deals with this issue automatically. That way the DApp developer can just focus on the application logic.

So instead of using the sendTransaction method, we first use an API called getData to get a transaction object:

```
var tx = contract.transfer.getData(receiver, tokens)
```

And then pass that back to the parent app through the $agent.response API:

```
$agent.response({ tx: tx })
```

The parent native app then will pass it along to our new wallet view.

![wallet](https://gliechtenstein.github.io/images/jasonette_wallet.png)

The wallet view (and the wallet agent it contains) will take this unsigned transaction data, sign it, and then broadcast it through [Infura](https://infura.io). You can check out the wallet view source code here: https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.json

Here’s the wallet agent code: https://github.com/gliechtenstein/erc20/blob/master/mobile/wallet/wallet.html

Note that the “wallet view” is a completely separate sandboxed view of its own, just like how [MetaMask](https://metamask.io/) opens up in a new popup browser. This is by design. This insulates the DApp developer from ever having to deal with private keys.



# Step by Step Explanation on Implementation Details

We've looked at how the app is structured on a high level. Now let's walk through each file to see how it's **actually** implemented.

First, it's easier to understand everything if you remember that each JSON file is a view (or gets mixed into another JSON to form a view). It's pretty much like HTML in that everything centers around the concept of a "page", except that every "page" is a native view.

There are three folders: 

- **app:** contains the main app view JSON markup
- **qr:** QR code scanner markup
- **wallet:** JSON markup for handling private keys, signing, and broadcasting to rinkeby network.

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

