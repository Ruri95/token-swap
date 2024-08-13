<img width="900" alt="30 Jul - Navigating the DeFi Ecosystem" src="https://github.com/user-attachments/assets/f4166974-50f5-400f-b084-5b95428f48ed">

# Quest 3 - Execute a Token Swap 🦄

Welcome to Quest 3 of the **Navigating the DeFi Ecosystem** campaign, where we will execute a token swap on Uniswap! 

## Pre-requisites

Before you begin, do ensure you have the following installed on your system:

- Git
- Node.js

## Project Setup ⚙️

1. Clone the repository
```bash
git clone https://github.com/clement-stackup/token_swap.git
```

2. Navigate to the project directory:
```bash
cd token_swap
```

3. Install the necessary dependencies & libraries
```bash
npm install --save
```

Now that you're set up, you're ready to start the quest! 🏁 

Create .env File

Next, proceed to create a .env file on the root of the project directory and paste the following file contents to your newly created .env file.
```bash
RPC_URL="REPLACE WITH RPC URL"
```
```bash
PRIVATE_KEY="REPLACE WITH PRIVATE KEY"
```
To generate a RPC URL, you can head over to Infura! Your RPC_URL should look something like this:

https://sepolia.infura.io/v3/9576ece978b44XXXXXXXXXXXX
Replace the placeholder values with your actual RPC_URL and PRIVATE_KEY
to  begin, head over to the index.js file!
Next, head over to the //Part A - Input Token Configuration comment where you should observe that we have initialized 2 variables - USDC and LINK.
change LINK and USDC contract addresschange LINK and USDC contract address
```bash
LINK =0x779877A7B0D9E8603169DdbD7836e478b4624789
USDC =0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238
```

Proceed to replace the placeholders with the correct token contract addresses on Ethereum Sepolia!
# Next, head over to the //Part B - Write Approve Token Function comment. Proceed to paste the following code snippet to overwrite the approveToken function.
```bash
async function approveToken(tokenAddress, tokenABI, amount, wallet) {
  try {
    const tokenContract = new ethers.Contract(tokenAddress, tokenABI, wallet);
    const approveAmount = ethers.parseUnits(amount.toString(), USDC.decimals);
    const approveTransaction = await tokenContract.approve.populateTransaction(
      SWAP_ROUTER_CONTRACT_ADDRESS,
      approveAmount
    );
    const transactionResponse = await wallet.sendTransaction(
      approveTransaction
    );
    console.log(`-------------------------------`);
    console.log(`Sending Approval Transaction...`);
    console.log(`-------------------------------`);
    console.log(`Transaction Sent: ${transactionResponse.hash}`);
    console.log(`-------------------------------`);
    const receipt = await transactionResponse.wait();
    console.log(
      `Approval Transaction Confirmed! https://sepolia.etherscan.io/tx/${receipt.hash}`
    );
  } catch (error) {
    console.error("An error occurred during token approval:", error);
    throw new Error("Token approval failed");
  }
}
```
Function Summary

The approveToken function is essential for approving a specified amount of a token (e.g., USDC) so that it can be utilized in a swap by a smart contract (e.g., Uniswap V3 Swap Router). This function operates as follows:

tokenAddress - The address of the token contract to be approved

tokenABI - The ABI JSON of the token contract

amount - The amount of tokens to approve

wallet - The wallet instance to sign the transaction

# Head over to the //Part C - Write Get Pool Info Function comment. Proceed to paste the following code snippet to overwrite the getPoolInfo function.
```bash
async function getPoolInfo(factoryContract, tokenIn, tokenOut) {
  const poolAddress = await factoryContract.getPool(
    tokenIn.address,
    tokenOut.address,
    3000
  );
  if (!poolAddress) {
    throw new Error("Failed to get pool address");
  }
  const poolContract = new ethers.Contract(poolAddress, POOL_ABI, provider);
  const [token0, token1, fee] = await Promise.all([
    poolContract.token0(),
    poolContract.token1(),
    poolContract.fee(),
  ]);
  return { poolContract, token0, token1, fee };
}
```
Function Summary

The getPoolInfo function retrieves information about a liquidity pool from a Uniswap V3 factory contract. This function operates as follows:

factoryContract - The factory contract instance used to get the pool address

tokenIn - The input token for the pool (i.e USDC)

tokenOut - The output token for the pool (i.e LINK)

# Head over to the //Part D - Write Prepare Swap Params Function comment. Proceed to paste the following code snippet to overwrite the prepareSwapParams function.
```bash
async function prepareSwapParams(poolContract, signer, amountIn) {
  return {
    tokenIn: USDC.address,
    tokenOut: LINK.address,
    fee: await poolContract.fee(),
    recipient: signer.address,
    amountIn: amountIn,
    amountOutMinimum: 0,
    sqrtPriceLimitX96: 0,
  };
}
```
Function Summary

The prepareSwapParams function prepares the parameters required for executing a token swap on a Uniswap V3 pool contract. This function operates as follows:

poolContract - The pool contract instance from which to retrieve the fee

signer - The wallet instance that will sign the transaction and receive the swapped tokens

amountIn - The amount of input tokens (i.e USDC) to be swapped

The function creates a new object containing the necessary swap parameters in particular:

tokenIn - The address of the input token (i.e USDC)

tokenOut - The address of the output token (i.e LINK)

fee - The fee tier of the pool

recipient - The address of the signer who will receive the swapped tokens

amountIn - The specified amount of input tokens to be swapped

The function then returns the constructed swap parameters object.

# Head over to the //Part E - Write Execute Swap Params Function comment. Proceed to paste the following code snippet to overwrite the executeSwap function.
```bash
async function executeSwap(swapRouter, params, signer) {
  const transaction = await swapRouter.exactInputSingle.populateTransaction(
    params
  );
  const receipt = await signer.sendTransaction(transaction);
  console.log(`-------------------------------`);
  console.log(`Receipt: https://sepolia.etherscan.io/tx/${receipt.hash}`);
  console.log(`-------------------------------`);
}
```
Function Summary

The executeSwap function executes a token swap on the Uniswap V3 Swap Router using the provided swap parameters. The function operates as follows:

swapRouter - The Swap Router contract instance used to perform the swap

params - The swap parameters object which contains the necessary parameters such as the tokens involved, amount to swap, recipient

signer - The wallet instance that will sign and send the transaction

# Finally, piecing all the different functions all together, head over to the //Part F - Write Main Function comment.

Proceed to paste the following code snippet to overwrite the main function.
```bash
async function main(swapAmount) {
  const inputAmount = swapAmount;
  const amountIn = ethers.parseUnits(inputAmount.toString(), USDC.decimals);

  try {
    await approveToken(USDC.address, TOKEN_IN_ABI, inputAmount, signer);
    const { poolContract } = await getPoolInfo(factoryContract, USDC, LINK);
    const params = await prepareSwapParams(poolContract, signer, amountIn);
    const swapRouter = new ethers.Contract(
      SWAP_ROUTER_CONTRACT_ADDRESS,
      SWAP_ROUTER_ABI,
      signer
    );
    await executeSwap(swapRouter, params, signer);
  } catch (error) {
    console.error("An error occurred:", error.message);
  }
}
```
Function Summary

The main function covers the entire process of swapping a specified amount of USDC for LINK on Uniswap V3 by taking in a swapAmount parameter (i.e amount of USDC to be swapped for LINK)

This function operates as follows:

Convert Swap Amount

The function first converts the specified swapAmount into the correct units based on USDC’s token decimals

Approve Token

The function then calls the approveToken function to approve the Swap Router contract to use the specified amount of USDC

Get Pool Information

The function then retrieves the required pool contract information for the USDC-LINK pair by calling getPoolInfo.

Prepare Swap Parameters
The function then prepares the necessary swap parameters by calling prepareSwapParams with the pool contract, signer and the converted amount.

Create Swap Router Contract Instance

The Swap Router contract instance is then created and calls the executeSwap function to perform the token swap!

# Execute Token Swap
to  execute the swap, proceed to run the following command.
```bash
node index.js
```

If the process is successful it will produce a link like this! If the process is successful it will produce a link like this! 
![Screenshot_2024-08-13-10-40-40-081_com android chrome-edit](https://github.com/user-attachments/assets/f6c22bf3-4df5-454e-b440-6cc632c8b5f9)


then visit the link.  You will be taken to a page showing proof that the process you have carried out has been successful.

# Let’s Ace Your Submissions! Preparing Your Submission!
We finally reached the end of the campaign! 🔥

Now, to make sure you successfully completed this quest, there is 1 deliverable that is required for this quest - a screenshot.

From Step 8, you are required to take a screenshot of your Explorer page showing a successful token swap!

Your screenshot should show:

your full screen, including your taskbar (for Windows and Linux) / dock (for MacOS)
Success transaction status
Exact Input Single function executed
USDC swapped for LINK on Ethereum Sepolia network
make sure that all parts visible in the ‘Expected output’ image below are also visible in your screenshot
Expected Output
<img width="809" alt="download (2)" src="https://github.com/user-attachments/assets/23e97749-f9cc-43cd-a22c-310e7f74d293">


 # simple diagram when code to programmatically execute the token swap 
<img width="1072" alt="download (3)" src="https://github.com/user-attachments/assets/723dda1e-5ff1-45e2-93f1-032b22c5a3a9">

# For a clearer and easier to understand tutorial, visit the site below :
https://earn.stackup.dev/campaigns/navigating-the-defi-ecosystem/quests/quest-3-execute-a-token-swap-a969
