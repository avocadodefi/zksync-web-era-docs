---
head:
  - - meta
    - name: "twitter:title"
      content: JS SDK Create2 Example | zkSync Era Docs
---

# Deploy contract with CREATE2 opcode

With zkSync Era contract can be deployed using the CREATE2 by simply building the contract into a binary format and deploying it to the
zkSync Era network.

- [Storage](https://github.com/zksync-sdk/zksync2-examples/blob/main/solidity/storage/Storage.sol): Contract without constructor.
- [Incrementer](https://github.com/zksync-sdk/zksync2-examples/blob/main/solidity/incrementer/Incrementer.sol): Contract with constructor.
- [Demo](https://github.com/zksync-sdk/zksync2-examples/blob/main/solidity/demo/Demo.sol): Contract that has a dependency on
  [Foo](https://github.com/zksync-sdk/zksync2-examples/blob/main/solidity/demo/Foo.sol) contract.

There is a [user guide](https://github.com/zksync-sdk/zksync2-examples/blob/main/solidity/README.md) on how to compile Solidity smart contracts using `zksolc`
compiler. `zksolc` compiler generates a `*.zbin` and a `combined.json` file that contains the bytecode and ABI of a smart contract.

## Deploy contract

```ts
import { Provider, types, Wallet, ContractFactory } from "zksync-ethers";
import { ethers, Contract, Typed } from "ethers";

const provider = Provider.getDefaultProvider(types.Network.Sepolia);
const PRIVATE_KEY = process.env.PRIVATE_KEY;
const wallet = new Wallet(PRIVATE_KEY, provider);

async function main() {
  const conf = require("../../solidity/storage/build/combined.json");
  const abi = conf.contracts["Storage.sol:Storage"].abi;
  const bytecode: string = conf.contracts["Storage.sol:Storage"].bin;

  const factory = new ContractFactory(abi, bytecode, wallet, "create2");
  const storage = (await factory.deploy({
    customData: { salt: ethers.hexlify(ethers.randomBytes(32)) },
  })) as Contract;
  console.log(`Contract address: ${await storage.getAddress()}`);

  console.log(`Value: ${await storage.get()}`);

  const tx = await storage.set(Typed.uint256(200));
  await tx.wait();

  console.log(`Value: ${await storage.get()}`);
}

main()
  .then()
  .catch((error) => {
    console.log(`Error: ${error}`);
  });
```

## Deploy contract with constructor

```ts
import { Provider, types, Wallet, ContractFactory } from "zksync-ethers";
import { Contract, ethers } from "ethers";

const provider = Provider.getDefaultProvider(types.Network.Sepolia);
const PRIVATE_KEY = process.env.PRIVATE_KEY;
const wallet = new Wallet(PRIVATE_KEY, provider);

async function main() {
  const conf = require("../../solidity/incrementer/build/combined.json");
  const abi = conf.contracts["Incrementer.sol:Incrementer"].abi;
  const bytecode: string = conf.contracts["Incrementer.sol:Incrementer"].bin;

  const factory = new ContractFactory(abi, bytecode, wallet, "create2");
  const incrementer = (await factory.deploy(2, {
    customData: { salt: ethers.hexlify(ethers.randomBytes(32)) },
  })) as Contract;
  console.log(`Contract address: ${await incrementer.getAddress()}`);

  console.log(`Value before Increment method execution: ${await incrementer.get()}`);

  const tx = await incrementer.increment();
  await tx.wait();

  console.log(`Value after Increment method execution: ${await incrementer.get()}`);
}

main()
  .then(() => console.log('Deployment successful')) // Added success message
  .catch((error) => {
    console.error(`Error during contract deployment: ${error}`);
  });
```

## Deploy contract with dependencies

```ts
import { Provider, types, Wallet, ContractFactory } from "zksync-ethers";
import { Contract, ethers } from "ethers";

const provider = Provider.getDefaultProvider(types.Network.Sepolia);
const PRIVATE_KEY = process.env.PRIVATE_KEY;
const wallet = new Wallet(PRIVATE_KEY, provider);

async function main() {
  const conf = require("../../solidity/demo/build/combined.json");
  const abi = conf.contracts["Demo.sol:Demo"].abi;
  const bytecode: string = conf.contracts["Demo.sol:Demo"].bin;

  const factory = new ContractFactory(abi, bytecode, wallet, "create2");
  const demo = (await factory.deploy({
    customData: {
      salt: ethers.hexlify(ethers.randomBytes(32)),
      factoryDeps: [conf.contracts["Foo.sol:Foo"].bin],
    },
  })) as Contract;
  console.log(`Contract address: ${await demo.getAddress()}`);

  console.log(`Value: ${await demo.getFooName()}`);
}

main()
  .then()
  .catch((error) => {
    console.log(`Error: ${error}`);
  });
```
