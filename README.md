##  Blockchain and SQL Fundamentals Fall 2024 Lab 4 (In-class exercise)
### Carnegie Mellon University
### Due: No due date
### 0 Points
### Deliverable: None

**Learning Objectives:**

In this short lab, we interact with a Uniswap V3 contract using Hardhat and Alchemy.

#### Acknowledgements

This lab is based on the one built by Arjun Brar and is found here:

https://github.com/boomcan90/uniswap-v3

## Setup and prerequisites

We will be using git, Node.js, and Hardhat. These tools were installed in Lab 1. In addition, we need to establish an account at Alchemy. You will need a free API key provided by Alchemy.
We do this so that we can copy a part of the Ethereum mainnet to our local machine.

We will be using ETH, Wrapped ETH (WETH), DAI, and Uniswap V3

Our objective is to buy some WETH with ETH and then exchange our WETH for the DAI stable coin. DAI is pegged one for one with the US dollar.

0. Create a new directory named Uniswap-Test and, within it, clone a repository.

```
mkdir Uniswap-Test
cd Uniswap-Test
git clone https://github.com/mm6/uniswap-v3.git

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
3. Run the following command to install dependencies and populate the node_modules directory:

```
npm install

```

4. Visit the Alchemy Dashboard [Alchemy](https://alchemy.com/) and get a free API key. You can do this by creating an app. (I chose the free version and I did not use a credit card of any kind.)

5. We want to run a local Ethereum node. We can deploy and test
smart contracts without using the Ethereum mainnet. The URL points to an archive node on Alchemy. The fork
command creates a local copy of the mainnet state (a snapshot).
We will be able to interact with a local node as if it had all of the state of the mainnet.

Run the following command and replace <API_KEY> with your API key.

```
npx hardhat node --fork https://eth-mainnet.g.alchemy.com/v2/<API_KEY>

```

6. Note that the server is running and the shell now displays private and public keys that we may use. The output looks similar to the following:

```
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on mainnet or any other live network WILL BE LOST.

Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

7. Open a new terminal shell and cd into the Uniswap-Test/uniswap-v3 directory.

8. In this second shell, run the following command to run the Hardhat console and connect to our local Ethereum node:

```
npx hardhat console --network localhost

```

The output should look similar to this:

```
Welcome to Node.js v20.11.1
Type ".help" for more information.
>
```
9. We need the ethers object from the hardhat module. Execute the following line of Javascript.

```js
const { ethers } = require("hardhat");
```

10. Next, we need signers (with ETH) who are able to sign transactions. Execute the following line of Javascript.

```js
let signers = await ethers.getSigners();
```
11. We need access to the WETH contract. So, we use the well known address of the WETH contract.

The WETH contract is an ERC-20 contract (Wrapped Eth) and trades 1 to 1 with ETH.

We can obtain WETH by sending ETH to the WETH smart contract.

We can also unwrap WETH back into ETH by sending WETH to the contract, which will burn the WETH and return the equivalent amount of ETH to your account.

Execute the following line of Javascript.

```js
const WETH_ADDRESS = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2";
```

12. We want to communicate with the WETH contract. Here, we define an Application Binary Interface (ABI) for the ERC-20 contract. The function
signatures are defined so that the caller will know how to encode the requests. Note that one of the functions is marked as payable and can accept ETH. Execute the following lines of Javascript.

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
13. Establish a local JavaScript object named WETH containing information that we need to perform a transaction.

There is no deployment here. This WETH contract already exists on the mainnet.

As an aside, if the WETH contract is not in our local cache, it will be fetched from Alchemy and placed in the cache for future calls.

We are using the first account (signers[0]) to sign and pay for transactions.

Execute the following line of Javascript.


```js
const WETH = new ethers.Contract(WETH_ADDRESS, ercAbi, signers[0]);
```

14. Check on how much WETH we currently own.

```
const wethbalance = await WETH.balanceOf(signers[0].address);
wethbalance
```

15. Trade ETH for some WETH. Place the WETH into our account on the WETH contract.
We are sending 10 ETH to the WETH contract in exchange for 10 WETH. The first signer is paying 10 ETH plus transaction fees.
Execute the following line of Javascript.

```js
const deposit = await WETH.deposit({ value: ethers.parseEther("10") });
```
Wait for the deposit to complete.

```js
await deposit.wait();
```
16. Check on how much WETH we own now.

```js
const balanceAfterDeposit = await WETH.balanceOf(signers[0].address);
balanceAfterDeposit
```

17. We need access to the DAI contract. The DAI contract provides an ERC-20 stable coin so that 1 DAI = 1 USD. It is part of the MakerDAO system.

Soon, we will use Uniswap to exchange our WETH for DAI.

You cannot send ETH to the DAI contract directly to receive DAI. Instead, DAI is typically obtained by locking up collateral (such as ETH) in a Maker Vault to generate DAI, or by trading on decentralized exchanges (DEXs) like Uniswap. We can, however,  exchange ETH for DAI using Uniswap. Here, we are using WETH to buy DAI.

The DAI contract also has a well known address. In a later command, we need the DAI_DECIMALs. Execute the following lines of Javascript.

```js
const DAI_ADDRESS = "0x6B175474E89094C44Da98b954EedeAC495271d0F";
const DAI_DECIMALS = 18;
```
18. Get access to the DAI contract. Execute the following line of Javascript.

```js

const DAI = new ethers.Contract(DAI_ADDRESS, ercAbi, signers[0]);
```

19. How much DAI do we own?

```
const initialDAIBalance = await DAI.balanceOf(signers[0].address);
const startingDAIBalance = Number(ethers.formatUnits(initialDAIBalance, DAI_DECIMALS));
console.log("DAI Balance: ", startingDAIBalance);

```
20. The answer may show 1e-18 which is effectively zero.

21. A swap router is part of Uniswap and is used to facilitate token swaps between
different ERC-20 tokens. A user can swap one token for another without using a
centralized exchange.

Liquidity providers place tokens into liquidity pools. The swap router interacts with
these pools to provide liquidity for trading pairs. The router can determine the path (sequence
of tokens) to achieve the desired swap. The swap router collects trading fees (e.g. 0.3% in Uniswap v2)
from users and these fees are distributed to liquidity providers based upon the liquidity
providers contribution to the pool.

Here, we use the well known address of the Uniswap router. Execute the following line of Javascript.

```js
const SwapRouterAddress = "0xE592427A0AEce92De3Edee1F18E0157C05861564";
```


22. Next, we deploy a simple swap contract named SimpleSwap. This contract should be in your contracts directory.

SimpleSwap needs to be deployed with the address of the swap router.

Execute the following lines of Javascript.

```js
/* Deploy the SimpleSwap contract */
const simpleSwapFactory = await ethers.getContractFactory("SimpleSwap");
const simpleSwap = await simpleSwapFactory.deploy(SwapRouterAddress);
await simpleSwap.waitForDeployment();
```

23. Next, we make an 'approval' call to grant permission for the SimpleSwap contract to access our WETH. The SimpleSwap contract will, in turn, grant approval to the decentralized exchange
(DEX) to spend our WETH.

Here, we approve the swap contract (SimpleSwap) to spend 1 WETH.

Execute the following line of Javascript. Note that later, the SimpleSwap contract will interact with the Uniswap router on our behalf. Here, we approve the SimpleSwap contract to spend 1 WETH.

```js
await WETH.approve(simpleSwap.target, ethers.parseEther("1"));
```

24. Establish the amount of WETH to exchange for DAI. This is .1, the amount of WETH to be swapped.


```js

const amountIn = ethers.parseEther("0.1");
```
Perform the swap. Since we are not providing DAI for the swap, there is no need to approve the contract to spend DAI. The DAI will be sent to your address as a result of the swap.

Note, we will not spend more on gas that 300000.

WETH: This is the address of the WETH token contract. It specifies the token you are swapping from.

DAI: This is the address of the DAI token contract. It specifies the token you are swapping to.

```js
const swap = await simpleSwap.swapERCforERC(WETH, DAI, amountIn, { gasLimit: 300000 });
await swap.wait();

```

25. Check our DAI balance by visiting the balanceOf on the DAI contract.
    Execute the following lines of Javascript.

```js

const newDAIBalance = await DAI.balanceOf(signers[0].address);
const DAIBalance = Number(ethers.formatUnits(newDAIBalance, DAI_DECIMALS));
console.log("DAI Balance: ", DAIBalance);
```

26. Use [this converter](https://www.coinbase.com/converter/eth/dai) to see if we were paid the correct amount of DAI for our WETH.

27. Let's examine the contract that is called by this console interaction. Comments have been
added to the code.

```
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity =0.7.6;

// We want to use the new v2 abi encoder. v2 provides more features
// than v1. This is all about encoding requests.

pragma abicoder v2;

// We want to interact with Uniswap's V3 swap router.
import '@uniswap/v3-periphery/contracts/interfaces/ISwapRouter.sol';

// Utilities are available for transferring tokens and handling errors.
import '@uniswap/v3-periphery/contracts/libraries/TransferHelper.sol';

contract SimpleSwap {
    ISwapRouter public immutable swapRouter;
    uint24 public constant feeTier = 3000;    

    // Three fee tier's are available
    // These are .05%, .30 %, and 1.00%.
    // 1.00% is used for pools with less
    // liquidity.

    // Inititialze SimpleSwap with a reference to the
    // Uniswap router.
    constructor(ISwapRouter _swapRouter) {
        swapRouter = _swapRouter;
    }

    // The arguments from and to are pointers to the two
    // token contracts.
    function swapERCforERC(address from, address to, uint amountIn) external returns (uint256 amountOut) {

        // Transfer the specified amount of 'from' tokens to this contract.
        // msg.sender is a holder of the 'from' tokens.
        // We are transferring tokens from the sender to this contract. The sender must have approved
        // this contract to spend these tokens on the their
        // behalf.
        TransferHelper.safeTransferFrom(from, msg.sender, address(this), amountIn);
        // Approve the router to spend ERC.
        // This approval is necessary for the router to
        // execute the swap on behalf of the contract.
        TransferHelper.safeApprove(from, address(swapRouter), amountIn);
        // Create the params that will be used to execute the swap
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: from,        // The from token
                tokenOut: to,         // The to token
                fee: feeTier,         // Each liquidity pool has a specific fee tier
                recipient: msg.sender, // The receiver of the to token
                deadline: block.timestamp, // The deadline when the swap must be executed
                amountIn: amountIn,       // The amount of the from token
                amountOutMinimum: 0,     // Minimal acceptable out token
                sqrtPriceLimitX96: 0    // When not zero, allows control of price impact of swaps
            });
        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
        return amountOut;
    }
}
```
## Notes on SimpleSwap smart contract

This smart contract is working on our behalf. We could also do these commands in Javascript from the client side.

License and Solidity Version:

The contract is licensed under GPL-2.0-or-later.
It uses Solidity version 0.7.6 and the ABI encoder v2.

Imports:

ISwapRouter: Interface for interacting with the Uniswap V3 swap router.

TransferHelper: Library for safely transferring tokens and handling errors.

Contract Variables:

swapRouter: An immutable reference to the Uniswap V3 router.
feeTier: A constant representing the fee tier for the liquidity pool (0.3% in this case).

Constructor:

Initializes the contract with a reference to the Uniswap router.

Function swapERCforERC:

Parameters: from (address of the input token), to (address of the output token), amountIn (amount of input tokens to swap).

Token Transfer: Transfers the specified amount of from tokens from the sender to the contract.

Approval: Approves the router to spend the from tokens.

Swap Execution: Creates the parameters for the swap and calls exactInputSingle on the router to execute the swap.

Return Value: Returns the amount of to tokens received from the swap.

## Preconditions before calling SimpleSwap

Before calling the swapERCforERC function in this SimpleSwap contract, there are several preconditions that must be met to ensure the function executes successfully:

Token Approval:

The user must approve the SimpleSwap contract to spend the specified amount of the from token (e.g., WETH) on their behalf.
This is done using the approve function of the from token's ERC-20 contract.
await WETH.approve(simpleSwap.address, amountIn);

Sufficient Token Balance:

The user must have a sufficient balance of the from token in their wallet to cover the amount they want to swap.
Ensure the amountIn specified in the function call does not exceed the user's token balance.

Contract Deployment:

The SimpleSwap contract must be deployed on the Ethereum network and its address known.

The contract should be initialized with a reference to the Uniswap V3 router.

Network Connection:

The user must be connected to the Ethereum network via a provider (e.g., Infura, Alchemy) and have a wallet set up with sufficient ETH to cover gas fees for the transaction.

Gas Limit:

Ensure that the transaction includes an appropriate gas limit to cover the execution of the swap.
const swap = await simpleSwap.swapERCforERC(WETH, DAI, amountIn, { gasLimit: 300000 });

Valid Token Addresses:

The from and to parameters must be valid ERC-20 token contract addresses.
Ensure that the tokens are supported by Uniswap V3 and have sufficient liquidity in the relevant pool.
