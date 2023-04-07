# starlight :stars:

Generate a zApp from a Solidity contract.

---

## Introduction

zApps are zero-knowledge applications. They're like dApps (decentralised applications), but with privacy. zApps are tricky to write, but Solidity contracts are lovely to write. So why not try to write a zApp with Solidity?  

`starlight` helps developers do just this...

- Write a Solidity contract
- Add a few new privacy decorators to the contract (to get a 'Zolidity' contract)
- Run `zappify`
- Get a fully working zApp in return

_Solidity contract --> Zolidity contract --> zappify --> zApp_


The main objective of this transpiler is to enable developers to quickly draft frameworks for zApps.

See [here](./doc/WRITEUP.md) for an enormously detailed explanation of how the transpiler works.


---

## Warnings

This code is not owned by EY and EY provides no warranty and disclaims any and all liability for use of this code. Users must conduct their own diligence with respect to use for their purposes and any and all usage is on an as-is basis and at your own risk.

**Note that this is an experimental prototype which is still under development. Not all Solidity syntax is currently supported. [Here](./doc/STATUS.md) is guide to current functionality.**  

**This code has not yet completed a security review and therefore we strongly recommend that you do not use it in production. We take no responsibility for any loss you may incur through the use of this code.**

**Due to its experimental nature, we strongly recommend that you do not use any zApps which are generated by this transpiler to transact items of material value.**

**Output zApps use the proof system Groth16 which is known to have [vulnerabilities](https://zokrates.github.io/toolbox/proving_schemes.html#g16-malleability). Check if you can protect against them, or are protected by the zApps logic.**

**In the same way that Solidity does not protect against contract vulnerabilities, Starlight cannot protect against zApp vulnerabilities once they are written into the code. This compiler assumes knowledge of smart contract security.**

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Quick User Guide](#quick-user-guide)
- [Install via npm](#install-via-npm)
  - [Troubleshooting](#troubleshooting)
- [Install via clone](#install-via-clone)
- [Run](#run)
  - [CLI options](#cli-options)
- [Troubleshooting](#troubleshooting-1)
  - [Installation](#installation)
  - [Compilation](#compilation)
- [Developer](#developer)
  - [Testing](#testing)
    - [full zapp](#full-zapp)
    - [circuit](#circuit)
  - [Contributing](#contributing)
- [License](#license)
- [Acknowledgements](#acknowledgements)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## Requirements

To run the `zappify` command:
- Node.js v15 or higher - v16 recommended.  
  (Known issues with v13 and v18).

To run the resulting zApp:
- Node.js v15 or higher.  
- Docker (with 16GB RAM recommended)
- Mac and Linux machines with at least 16GB of memory and 10GB of disk space are supported.

---

## Quick User Guide

Take a 'normal' smart contract, like this one:

```solidity
// SPDX-License-Identifier: CC0

pragma solidity ^0.8.0;

contract Assign {

  uint256 private a;

  function assign(uint256 value) public {
    a = value;
  }
}
```

Then add `secret` in front of each declaration you want to keep secret:
```solidity
// SPDX-License-Identifier: CC0

pragma solidity ^0.8.0;

contract Assign {

  secret uint256 private a; // <--- secret

  function assign(secret uint256 value) public { // <--- secret
    a = value;
  }
}
```

Save this decorated file with a `.zol` extension ('zolidity').

Run `zappify -i <./path/to/file>.zol` and get an entire standalone zApp in return!


---
## Install via npm

Starlight is now available on **github's npm package** repository!
To install, you need to make sure that `npm` is looking in that registry, not the default `npmjs`:

`npm config set @eyblockchain:registry https://npm.pkg.github.com` <-- this sets any `@eyblockchain` packages to be installed from the github registry

`npm i @eyblockchain/starlight@<latest-version>`

Then you can import the `zappify` command like so:

`import { zappify } from "@eyblockchain/starlight";`

The only required input for `zappify` is the file path of the `.zol` file you'd like to compile, e.g. `zappify('./contracts/myContract.zol')`. You can optionally add a specific zApp name and the output directory:

`zappify('./contracts/myContract.zol', 'myzApp', './test_zApps')`

Then it works just as described in the below section [Run](#run).

### Troubleshooting

Since we push the npm package to github's repository, you may need to specify the package location in your project's `.npmrc` file and authenticate to github. Try adding something like:

```
@eyblockchain:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=<your_github_token>
```

...and re-run the install command, or manually add `"@eyblockchain/starlight"` to your `package.json`.
## Install via clone

To install via github:

Clone the repo.

`cd starlight`

`./bin/start` (You might need to run chmod +x ./bin/start for permission to execute the newly created shell scripts)

This runs `tsc` and `npm i -g ./` which will create a symlink to your node.js bin, allowing you to run the commands specified in the `"bin":` field of the `package.json`; namely the `zappify` command.

## Run

`zappify -i ./path/to/MyZolidityContract.zol`

... converts a Zolidity contract into a zApp. By default, the zApp is output to a `./zapps/` folder.

### CLI options

| option  | abbr.  | description  |
|:--|:--|:--|
| `--input <./path/to/contract.zol>`  | `-i`  | Specify an input contract file with a `.zol` extension.  |
| `--output <./custom/output/dir/>`  | `-o`  | Specify an output directory for the zApp. By default, the zApp is output to a `./zapps/` folder.  |
| `--zapp-name <customZappName>` | `-z`  | Otherwise files get output to a folder with name matching that of the input file.  |
| `--log-level <debug>`  | -  | Specify a Winston log level type.  |
| `--enc`  | `-e`  | Add secret state encryption events for every new commitment*.  |
| `--help`  | `-h`  | CLI help.  |

*Note that encrypting and broadcasting secret state information to its owner is costly gas-wise, so switching it on with this option may give you `stack too deep` errors in the contract.

An alternative is to use our `encrypt` syntax in front of the state edit you'd like to broadcast, like this:

```solidity
contract Assign {

  secret uint256 private a; // <--- secret

  function assign(secret uint256 value) public { // <--- secret
    encrypt a = value; // <--- the new value of a will be encrypted and sent to a's owner
  }
}
```

Partitioned states which are incremented and have an owner which is clearly different from the function caller has encryption included by default.

---

## Troubleshooting

### Installation

If the `zappify` command isn't working, try the [Install](#install-via-clone) steps again. You might need to try `npm i --force -g ./`.

In very rare cases, you might need to navigate to your node.js innards and delete zappify from the `bin` and `lib/node_modules`.
To find where your npm lib is, type `npm` and it will tell you the path.

E.g.:
```
~/.nvm/versions/node/v15.0.1/lib/node_modules/npm
                              ^
                              lib
                              ^
                              bin is also at this level
```

### Compilation

If you find errors to do with 'knownness' or 'unknownness', try to mark incrementations. If you have any secret states which can be **incremented** by other users, mark those incrementations as `unknown` (since the value may be unknown to the caller).

```solidity
// SPDX-License-Identifier: CC0

pragma solidity ^0.8.0;

contract Assign {

  secret uint256 private a; // <--- secret

  function add(secret uint256 value) public { // <--- secret
    unknown a += value; // <--- may be unknown to the caller
  }

  function remove(secret uint256 value) public { // <--- secret
    a -= value;
  }
}
```

However, if you want the incrementation to only be completed by the secret owner, mark it as known:

```solidity
// SPDX-License-Identifier: CC0

pragma solidity ^0.8.0;

contract Assign {

  secret uint256 private a; // <--- secret

  function add(secret uint256 value) public { // <--- secret
    known a += value; // <--- must be known to the caller
  }

  function remove(secret uint256 value) public { // <--- secret
    a -= value;
  }
}
```

Failing to mark incrementations will throw an error, because the transpiler won't know what to do with the state. See the [write up](./doc/WRITEUP.md) for more details.

If your input contract has any external imports, make sure those are stored in the `./contracts` directory (in root) and compile with `solc` 0.8.0.


---

## Developer

### Testing

#### full zapp

To test an entire zApp, which has been output by the transpiler:

Having already run `zappify`, the newly-created zApp will be located in the output dir you specified (or in a dir called `./zapps`, by default). Step into that directory:

`cd zapps/MyContract/`

Install dependencies:

`npm install`

Start docker.

(At this stage, you might need to run `chmod +x ./bin/setup && chmod +x ./bin/startup` for permission to execute the newly created shell scripts)

Run trusted setups on all circuit files:

`./bin/setup` <-- this can take quite a while!

After this command, you're ready to use the zApp. You can either run a Starlight-generated test, or call the included APIs for each function.

For the compiled test see below:

`npm test` <-- you may need to edit the test file (`zapps/MyContract/orchestration/test.mjs`) with appropriate parameters before running!

`npm run retest` <-- for any subsequent test runs (if you'd like to run the test from scratch, follow the below instructions)

It's impossible for a transpiler to tell which order functions must be called in, or the range of inputs that will work. Don't worry - If you know how to test the input Zolidity contract, you'll know how to test the zApp. The signatures of the original functions are the same as the output nodejs functions. There are instructions in the output `test.mjs` on how to edit it.

All the above use Docker in the background. If you'd like to see the Docker logging, run `docker-compose -f docker-compose.zapp.yml up` in another window before running.

**NB: rerunning `npm test` will not work**, as the test script restarts the containers to ensure it runs an initialisation, removing the relevant dbs. If you'd like to rerun it from scratch, down the containers with `docker-compose -f docker-compose.zapp.yml down -v --remove-orphans` and delete the file `zapps/myContract/orchestration/common/db/preimage.json` before rerunning `npm test`.

For using APIs:

`npm start` <-- this starts up the `express` app and exposes the APIs

Each route corresponds to a function in your original contract. Using the `Assign.zol` example above, the output zApp would have an endpoint for `add` and `remove`. By default, requests can be POSTed to `http://localhost:3000/<function-name>` in JSON format. Each variable should be named like the original parameter. For example, if you wanted to call `add` with the value 10, send:

```
{
  value: 10
}
```
to `http://localhost:3000/add`. This would (privately) increase the value of secret state `a` by 10, and return the shield contract's transaction.

To access your secret data locally, you can look at all the commitments you have by sending a GET request to `http://localhost:3000/getAllCommitments`.

You can also filter these commitments by variable name. Using the example above, if you wanted to see all commitments you have representing the variable `a`, send:
```
{
    "name": "a"
}
```
as a GET request to `http://localhost:3000/getCommitmentsByVariableName`.

#### Deploy on public testnets

Apart from local ganache instance, Starlight output zapps can now be deployed in Sepolia, Goerli and Polygon Mumbai as cli options. Connection to Sepolia and Goerli are made through [infura](https://infura.io/) endpoints and that of Polygon Mumbai is provided via [maticvigil](https://rpc.maticvigil.com/).

The configuration can be done during `./bin/setup` phase in the following way.

`./bin/setup -n network -a account -k privatekey -m "12 letter mnemonic" -r APIKey`

##### CLI options

| option  | description  |
|:--|:--|
| `-n`  | Network : Specify testnet you want to connect to. possible options: `mumbai/ sepolia/ goerli`  |
| `-a`  | Ethereum address |
| `-k`  | Private key of above ethereum address |
| `-m` -  | 12 letter mnemonic passphrase  |
| `-r`  | API key or APPID of endpoint |
| `-s`  | Zkp setup flag , Default to yes . If you had already created zkp keys before and just want to configure deployment infrastructure, pass `-s n`  |

#### circuit

`cd ./path/to/myCircuit.zok`

`docker run -v $PWD:/app/code -ti docker.pkg.github.com/eyblockchain/zokrates-worker/zokrates_worker:1.0.8 /bin/bash`

`./zokrates compile --light -i code/myCircuit.zok` <-- it should compile

### Contributing

See [here](./CONTRIBUTIONS.md) for our contribution guidelines.


---

## License

[CC0 1.0 Universal](./LICENSE)

**Notes:**
- This repository contains some modules of code and portions of code from Babel (https://github.com/babel/babel). All such code has been modified for use in this repository. Babel is MIT licensed. See [LICENSE](./LICENSE) file.
- This repository contains some portions of code from The Super Tiny Compiler (https://github.com/jamiebuilds/the-super-tiny-compiler). All such portions of code have been modified for use in this repository. The Super Tiny Compiler is CC-BY-4.0 licensed. See [LICENSE](./LICENSE) file.
---

## Acknowledgements

Authors:

 - MirandaWood
 - iAmMichaelConnor

Important packages:

- [solc-js](https://github.com/ethereum/solc-js)
- [zokrates](https://github.com/Zokrates/ZoKrates)


Inspirational works:
- [Babel handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#toc-scopes)
- [The Super Tiny Compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)
