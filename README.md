# Presentation link https://aka.ms/Trufflecon2018
# Demo code https://github.com/DavidBurela/Trufflecon2018-Demo

# Intro
- Live in Australia (30 hour travel time to Trufflecon)
- Blockchain Domain lead @ Microsoft.
- CSE (Commercial Software Engineering), work with Microsoft's top 400 enterprises globally to help their dev teams be awesome on Azure.
- Been using Truffle since 2016, I created a lot of the early Truffle on Windows tutorials https://truffleframework.com/tutorials


# Dev environment

## Ubuntu on Windows (Windows Subsystem for Linux)
Because some npm packages don't run nicely on windows (due to node-gyp), I recommend devs use Ubuntu on Windows.

- [Installing Truffle on Ubuntu](https://davidburela.wordpress.com/2017/05/12/how-to-install-truffle-testrpc-on-ubuntu-or-windows-10-with-windows-subsystem-for-linux/)
- Use Hyper as a better terminal https://hyper.is/
- [WSL Install guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
- [WSL overview blog post](https://blogs.msdn.microsoft.com/wsl/2016/04/22/windows-subsystem-for-linux-overview/)

> **Demo:** show that `screenfetch` & `htop` work


## Visual Studio Code
Free. Open Source. Runs on Windows / Linux / Mac. Lots of plugins available.

- https://code.visualstudio.com/
- [Setting up VS code](https://davidburela.wordpress.com/2016/11/18/configuring-visual-studio-code-for-ethereum-blockchain-development/)
- [Solidity extension for VS Code](https://marketplace.visualstudio.com/items?itemName=JuanBlanco.solidity)

# Enterprise truffle usage

## Setting up the project

Start with metacoin box as it has things nicely configured
``` bash
truffle unbox metacoin
```

> **PAIN POINT:** Truffle boxes missing licensing.

## Continuous test runner
> **PAIN POINT:** `truffle watch` does not work

Need to use a tool like `nodemon` or [chokidar-cli](https://www.npmjs.com/package/chokidar-cli) to watch and run tests.
``` bash
npm install -g chokidar-cli
chokidar 'contracts/*.sol' 'test/*.js' -c 'truffle test' --initial
```

> **PAIN POINT:** I want a common artifact format across Truffle & Nethereum. Load a `metacoin.json` file into Nethereum and have it auto detect the ABI & deployed address.

## Async tests
> **PAIN POINT:** the sample tests are sync. Async tests are smaller and cleaner.

Change unit tests to async
- [Blog post](https://davidburela.wordpress.com/2017/09/21/writing-truffle-tests-with-asyncawait/)
- [Direct link to test sample](https://github.com/DavidBurela/TruffleAsyncTests/blob/master/test/asyncmetacoin.js)

# Blockchain DevOps

> Prerequisite: Install truffle as a dev dependency
``` bash
npm init -y
npm install truffle --save-dev
```

## Azure DevOps Pipeline

> Cloud-hosted pipelines for Linux, macOS, and Windows with unlimited minutes and **10 free parallel jobs for open source.** https://azure.microsoft.com/en-us/services/devops/pipelines/

## Install and configure via GitHub marketplace
https://github.com/marketplace/azure-pipelines

### Compile & test

``` yaml
# azure-pipelines.js - npm install, then run the tests
- script: |
    npm install
  displayName: 'npm install'

- script: |
    npx truffle compile
    npx truffle test
  displayName: 'truffle compile & test'
```

## Advanced test results

``` bash
# Install mocha & junit reporter, allowing output in JUnit XML
npm install truffle mocha mocha-junit-reporter --save-dev
```

``` javascript
// truffle.js - specify mocha output options
mocha: {
    reporter: "mocha-junit-reporter",
    reporterOptions: {
      mochaFile: 'TEST-truffle.xml'
    }
  },
```

> **PAIN POINT:** Need to change test runner manually. Can I pass a param into `truffle test` to change the test runner.

``` yaml
# azure-pipelines.js - Publish Test Results in 
- task: PublishTestResults@2
  condition: always()
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: '**/TEST-*.xml' 
```

## break a test and do a pull request
- modify metacoin.sol to break test
- check broken code on a branch and push
- create a pull request

## add the build status badge
https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started-yaml?view=vsts#get-the-status-badge

# Ethereum on Azure
- [Ethereum PoA marketplace listing](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoft-azure-blockchain.azure-blockchain-ethereum?tab=Overview)
- [Ethereum PoA deployment guide](https://docs.microsoft.com/en-us/azure/blockchain/templates/ethereum-poa-deployment)

## Automated deployment
``` bash
npm install truffle-hdwallet-provider --save
```
> **PAIN POINT:** requires node-gyp. Can we have this web packed like Truffle? 

Note: don't put raw mnemonic in truffle.js, use pipeline variables to keep them secret 
- https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups
- https://stackoverflow.com/questions/49578709/is-there-a-way-to-provide-environment-variables-to-a-vsts-ci-npm-task

``` javascript
// truffle.js
var HDWalletProvider = require("truffle-hdwallet-provider");
var mnemonic = "candy maple cake sugar pudding cream honey rich smooth crumble sweet treat";
var networkEndpoint = "http://eth5kzzgs-dns-reg1.westus.cloudapp.azure.com:8540";
// var mnemonic = process.env.deploymentMnemonic;
// var networkEndpoint = process.env.deploymentNetworkEndpoint;


module.exports = {
//...
	networks: {
		azure: {
			provider: function () {
			    return new HDWalletProvider(mnemonic, networkEndpoint, 0)
		},
		gasPrice : 0,
		network_id: "*"
		}
	}
//...
}
```

``` yaml
- script: |
    npx truffle migrate --network azure
  displayName: 'contract deployment'
```

> **PAIN POINT:** Once migrations are run, where to store the artifacts? EthPM?
