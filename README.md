##  Developing Blockchain Use Cases Spring 2024 Lab 4 (Optional work)
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
We do this so that we can copy a part of the Ethereum Mainnet to our local machine.

We will be using ETH, Wrapped ETH (WETH), DAI, and Uniswap version 3.

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
3. Run the following command:

```
npm install

```

4. Visit the Alchemy Dashboard [Alchemy](https://alchemy.com/) and get a free API key.

5. Run the following command and replace <API_KEY> with the API key.

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
9. We need the ethers object from the hardhat module. Execute the following line of Javascript.

```js
const { ethers } = require("hardhat");
```
10. We need access to the WETH contract. So, we use the well known address of the WETH contract.
This is an ERC-20 contract (Wrapped Eth) and trades 1 to 1 with eth. Execute the following line of Javascript.

```js
const WETH_ADDRESS = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2";
```

11. We need access to the DAI contract. The DAI contract provides a stable coin so that 1 DAI = 1 USD.
It also has a well known address. In a later command, we need the DAI_DECIMALs. Execute the following lines of Javascript.

```js
const DAI_ADDRESS = "0x6B175474E89094C44Da98b954EedeAC495271d0F";
const DAI_DECIMALS = 18;
```

12. A swap router is an important player and is used to facilitate token swaps between
different ERC-20 tokens. A user can swap one token for another without using a
centralized exchange.

Liquidity providers place tokens into liquidity pools. The swap router interacts with
these pools to provide liquidity for trading pairs. The router can determine the path (sequence
of tokens) to achieve the desired swap. The swap router collects trading fees (e.g. 0.3% in Uniswap v2)
from users and these fees are distributed to liquidity providers based upon the liquidity
providers contribution to the pool.

Here, we use the well known address of the uniswap router. Execute the following line of Javascript.

```js
const SwapRouterAddress = "0xE592427A0AEce92De3Edee1F18E0157C05861564";
```

13. Here, we define an Application Binary Interface (ABI) for the ERC-20 contracts. The function
signatures are defined so that the caller will know how to encode the requests. Execute the following lines of Javascript.

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
14. Next, we deploy a simple swap contract. Note that it needs to know the address of the swap router.
Execute the following lines of Javascript.

```js
/* Deploy the SimpleSwap contract */
const simpleSwapFactory = await ethers.getContractFactory("SimpleSwap");
const simpleSwap = await simpleSwapFactory.deploy(SwapRouterAddress);
await simpleSwap.waitForDeployment();
```

15. Next, we need signers (with ETH) who are able to sign transactions. Execute the following line of Javascript.

```js
let signers = await ethers.getSigners();
```

16. Establish a constant (WETH) containing information that we need to perform
a transaction.

There is no deployment here. This contract already exists on the mainnet.

We are using the first account (signers[0]) to sign and pay for transactions.

Execute the following line of Javascript.


```js
const WETH = new ethers.Contract(WETH_ADDRESS, ercAbi, signers[0]);
```

17. Trade ETH for some WETH. Place the WETH into our account on the WETH contract.
We are sending 10 eth to the WETH contract in exchange for 10 WETH.
Execute the following line of Javascript.

```js
const deposit = await WETH.deposit({ value: ethers.parseEther("10") });
```
Wait for the deposit to complete.

```js
await deposit.wait();
```

18. The approve call is essential for granting permission to the decentralized exchange
(DEX) contract to handle token transfers on behalf of the user during swaps.

We can now approve the swap contract to spend 1 WETH. It will need to reduce
our WETH and increase our DAI. This can be done by running the following commands.

Execute the following line of Javascript.

```js
await WETH.approve(simpleSwap.target, ethers.parseEther("1"));
```

19. Finally, we're ready to swap our WETH for DAI.

Get access to the DAI contract. Execute the following line of Javascript.

```js

const DAI = new ethers.Contract(DAI_ADDRESS, ercAbi, signers[0]);
```

Establish the amount to exchange. This is .1, the amount of WETH to be swapped.
Note, we will not spend more on gas that 300000.

```js

const amountIn = ethers.parseEther("0.1");
```
Perform the swap.

```js
const swap = await simpleSwap.swapERCforERC(WETH, DAI, amountIn, { gasLimit: 300000 });
await swap.wait();

```

20. Check our DAI balance by visiting the balanceOf on the DAI contract.
    Execute the following lines of Javascript.

```js

const expandedDAIBalance = await DAI.balanceOf(signers[0].address);
const DAIBalance = Number(ethers.formatUnits(expandedDAIBalance, DAI_DECIMALS));
console.log("DAI Balance: ", DAIBalance);
```

21. Let's examine the contract that is called by this console interaction. Comments have been
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
    uint24 public constant feeTier = 3000;    // Three fee tier's are available
                                              // These are .05%, .30 %, and 1.00%.
                                              // 1.00% is used for pools with less
                                              // liquidity.

    // Inititialze SimpleSwap with a reference to the Uniswap router.
    constructor(ISwapRouter _swapRouter) {
        swapRouter = _swapRouter;
    }

    // The arguments from and to are pointers to the two token contracts.
    function swapERCforERC(address from, address to, uint amountIn) external returns (uint256 amountOut) {

        // Transfer the specified amount of ERC to this contract.
        // msg.sender is a holder of the from tokens.
        // We are taking tokens from the sender and giving the contract ownership.
        TransferHelper.safeTransferFrom(from, msg.sender, address(this), amountIn);
        // Approve the router to spend ERC.
        // This approval is necessary for the router to execute the swap on behalf of the contract.
        TransferHelper.safeApprove(from, address(swapRouter), amountIn);
        // Create the params that will be used to execute the swap
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: from,                  // The from token
                tokenOut: to,                   // The to token
                fee: feeTier,                   // Each liquidity pool has a specific fee tier
                recipient: msg.sender,          // The receiver of the to token
                deadline: block.timestamp,      // The deadline when the swap must be executed
                amountIn: amountIn,             // The amount of the from token
                amountOutMinimum: 0,            // Minimal acceptable out token
                sqrtPriceLimitX96: 0            // When not zero, allows control of price impact of swaps
            });
        // The call to `exactInputSingle` executes the swap.
        amountOut = swapRouter.exactInputSingle(params);
        return amountOut;
    }
}
```
