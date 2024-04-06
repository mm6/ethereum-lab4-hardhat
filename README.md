
## Ethereum-Lab4-Hardhat

# Working with Uniswap v3

## Acknowledgements

This lab is based on the one built by Arjun Brar and is found here

https://github.com/boomcan90/uniswap-v3

## Setup and prerequisites

We will be using git, Node.js, and Hardhat. These tools were installed in Lab 1. In addition, we need to establish an account at [Alchemy](https://alchemy.com/). We do this so that we can copy a part of the Ethereum mainnet to our local machine.

We will be using ETH, Wrapped ETH (WETH), DAI, and Uniswap version 3.

0. Create a new directory named Uniswap-Test and, witin it, clone a repository.

```
mkdir Uniswap-Test
cd Uniswap-Test
git clone https://github.com/boomcan90/uniswap-v3.git
```

1. cd into the new directory named uniswap-v3.

```
cd uniswap-v3

```
2. Edit the file hardhat.config.js so that it contains the following content:
```

require("@nomicfoundation/hardhat-toolbox");
/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  defaultNetwork: "localhost",
  networks: {
    hardhat: {
      initialBaseFeePerGas: 0,
      chainId: 31337
    },
    localhost: {
      url: "http://127.0.0.1:8545"
    }
  },
  solidity: {
    compilers : [
      {version : "0.5.14"},
      {version : "0.7.6"},
      {version :"0.7.5"},
      {version :"0.6.0"},
      {version :"0.7.0"}]
}
}

```
3. Run the following command:

```
npm install

```

4. Visit the Alchemy Dashboard and get an API key.

5. Run the following command:

```
npx hardhat node --fork https://eth-mainnet.g.alchemy.com/v2/<API_KEY>

```

6. Note that the server is running and the shell now displays private and public keys that we may use. The output looks similar to the following:

```
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

7. Open a new terminal shell and cd into the Uniswap-Test/uniswap-v3 directory.

8. In this second shell, run the following commands:
```
npx hardhat console --network localhost

```

The output should look similar to this:

```
Welcome to Node.js v20.11.1
Type ".help" for more information.
>
```
We need the ethers object from the hardhat module.

```js
const { ethers } = require("hardhat");
```
We need access to the WETH contract. So, we use the well known address of the WETH contract.
This is an ERC-20 contract.

```js
const WETH_ADDRESS = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2";
```
We need access to the DAI contract. The DAI contract provides a stable coin so that 1 DAI = 1 USD.
It also has a well known address. In a later command, we need the DAI_DECIMALs.

```js
const DAI_ADDRESS = "0x6B175474E89094C44Da98b954EedeAC495271d0F";
const DAI_DECIMALS = 18;
```

A swap router is an important player and is used to facilitate token swaps between
different ERC-20 tokens. A user can swap one token for another without using a
centralized exchange.

Liquidity providers place tokens into liquidity pools. The swap router interacts with
these pools to provide liquidity for trading pairs. The router can determine the path (sequence
of tokens) to achieve the desired swap. The swap router collects trading fees (e.g. 0.3% in Uniswap v2)
from users and these fees are distributed to liquidity providers based upon the liquidity
providers contribution to the pool.

Here, we use the well known address of the uniswap router.

```js
const SwapRouterAddress = "0xE592427A0AEce92De3Edee1F18E0157C05861564";
```

Here, we define an Application Binary Interface (ABI) for the ERC-20 contracts. The function
signatures are defined so that the caller will know how to encode the requests.

```js
const ercAbi = [
  // Read-Only Functions
  "function balanceOf(address owner) view returns (uint256)",
  // Authenticated Functions
  "function transfer(address to, uint amount) returns (bool)",
  "function deposit() public payable",
  "function approve(address spender, uint256 amount) returns (bool)",
];
```
Next, we deploy a simple swap contract. Note that it needs to know the address of the swap router.

```js
/* Deploy the SimpleSwap contract */
const simpleSwapFactory = await ethers.getContractFactory("SimpleSwap");
const simpleSwap = await simpleSwapFactory.deploy(SwapRouterAddress);
await simpleSwap.waitForDeployment();
```

Next, we need signers (with ETH) who are able to sign transactions.

```js
let signers = await ethers.getSigners();
```

Establish a constant (WETH) containing information that we need to perform
a transaction.

There is no deployment here. This contract already exists on the mainnet.


```js
const WETH = new ethers.Contract(WETH_ADDRESS, ercAbi, signers[0]);
```

Trade ETH for some WETH. Place the WETH into our account on the WETH contract.

```js
const deposit = await WETH.deposit({ value: ethers.parseEther("10") });
```
Wait for the deposit to complete.

```js
await deposit.wait();
```

The approve call is essential for granting permission to the DEX contract to handle
token transfers on behalf of the user during swaps.

We can now approve the contract to spend our WETH. This can be done
by running the following commands.

```js
await WETH.approve(simpleSwap.target, ethers.parseEther("1"));
```

Finally, we're ready to swap our WETH for DAI.

Get access to the DAI contract.

```js

const DAI = new ethers.Contract(DAI_ADDRESS, ercAbi, signers[0]);
```

Establish the amount.
```js

const amountIn = ethers.parseEther("0.1");
```
Perform the swap.

```js
const swap = await simpleSwap.swapERCforERC(WETH, DAI, amountIn, { gasLimit: 300000 });
await swap.wait();

```

Check our DAI balance:

```js

const expandedDAIBalance = await DAI.balanceOf(signers[0].address);
const DAIBalance = Number(ethers.formatUnits(expandedDAIBalance, DAI_DECIMALS));
console.log("DAI Balance: ", DAIBalance);
```
