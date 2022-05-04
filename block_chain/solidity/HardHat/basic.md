## HardHat 开发指南

Open a new terminal and run these commands:

```shell
mkdir hardhat-tutorial
cd hardhat-tutorial
npm init --yes
npm install --save-dev hardhat
```



TIP:

Installing **Hardhat** will install some Ethereum JavaScript dependencies, so be patient.

In the same directory where you installed **Hardhat** run:

```shell
npx hardhat
```

### Plugins

**Hardhat** is unopinionated in terms of what tools you end up using, but it does come with some built-in defaults. All of which can be overriden. Most of the time the way to use a given tool is by consuming a plugin that integrates it into **Hardhat**.

For this tutorial we are going to use the Ethers.js and Waffle plugins. They'll allow you to interact with Ethereum and to test your contracts. We'll explain how they're used later on. To install them, in your project directory run:

```shell
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```