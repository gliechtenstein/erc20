# Full Stack ERC20 App

This repository contains EVERYTHING.

1. Backend: ERC20 contract written in solidity
2. Web Frontend: Web3.js DApp.
3. Mobile Frontend: JSON markup for [Jasonette](https://www.jasonette.com)

This document will explain how these all come together, and walk you through everything you need to do to deploy a contract to Ethereum, build a web app, and build a mobile app.

# 1. Backend

The "backend" is just a set of solidity contracts. This tutorial is not about writing ERC20 tokens or smart contracts so we're just going to use the most basic token possible.

![zeppelin](https://openzeppelin.org/img/logo-zeppelin.png)

99% of it is copy and pasted from Openzeppelin's [Zeppelin-solidity](https://github.com/OpenZeppelin/zeppelin-solidity) repository. The following code are 100% copy and pastes:

- [BasicToken.sol](contracts/BasicToken.sol)
- [ERC20.sol](contracts/ERC20.sol)
- [ERC20Basic.sol](contracts/ERC20Basic.sol)
- [SafeMath.sol](contracts/SafeMath.sol)
- [StandardToken.sol](contracts/StandardToken.sol)

1% of it is customization:

- [ExampleToken.sol](contracts/ExampleToken.sol)

The `ExampleToken` is the only contract that I've tweaked, just to customize the symbol, decimals, and total supply

For simplicity, I didn't use any frameworks like [Truffle](https://github.com/trufflesuite/truffle).

## Deployment

You can go to [Remix](https://remix.ethereum.org), copy and paste all the files in [contracts](contracts) folder, and deploy to Ethereum.

There are many tutorials on how to do this online. Google them.

In my case, I have deployed it to [rinkeby](https://www.rinkeby.io/#stats) testnet, and the resulting contract address is [0x3823619872186eff668aad8192590faaffef6a5f](https://rinkeby.etherscan.io/address/0x3823619872186eff668aad8192590faaffef6a5f).

We will use [this contract address in the DApp](https://github.com/gliechtenstein/erc20/blob/master/web/index.html#L190) to connect to the contract.

---

# 2. Web

The entire DApp is a single HTML file. It uses a JavaScript app framework called [cell.js](https://www.celljs.org) to implement the app in as simple manner as possible. 

**You can try it out here: https://gliechtenstein.github.io/erc20/web/**

![img](https://gliechtenstein.github.io/images/dapp.png)

Note that:

- For simplicity, only [two external libraries](https://github.com/gliechtenstein/erc20/blob/master/web/index.html#L38) are used: [web3.js](https://github.com/ethereum/web3.js/) and [cell.js](https://github.com/intercellular/cell)
- You can see the entire code transparently (no compilation, etc.) when you ["view source" from the web app](https://gliechtenstein.github.io/erc20/web/)
- The app uses version 0.14.0 (not ver 1.0+) of web3.js library

Learn more about the web frontend: [Web app tutorial](web/tutorial.md)


# 3. Mobile

The mobile frontend is powered by [Jasonette](https://www.jasonette.com), a markup driven approach to building mobile apps.

![img](https://gliechtenstein.github.io/images/erc20.gif)

Jasonette is view-oriented. Therefore everything revolves around views. There are three views in the app:

1. Main view: Display ERC20 token stats and send tokens
2. Private key QR code scanner: For importing user's private key
3. Receiver Public key QR code scanner: To send a token to another user, you need to scan the user's public key

Learn more about the mobile frontend: [Mobile app tutorial](mobile/tutorial.md)
