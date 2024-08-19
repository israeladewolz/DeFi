## Overview of Script
This script demonstrates how to interact with multiple DeFi protocols to swap tokens, deposit assets for earning interest, and optionally engage in yield farming. The script primarily integrates Uniswap and Aave, with an optional extension for Yearn Finance.

Script Workflow:
Token Swap on Uniswap:
The script starts by swapping a specific amount of USDC for LINK using Uniswap’s router contract. This swap can also be configured for other tokens, such as ETH to DAI, if exploring other yield farming strategies.

Supply LINK to Aave:
After acquiring LINK from Uniswap, the script deposits the LINK into Aave’s lending pool. This step starts generating interest based on Aave’s dynamic interest rate.

Optional Yield Farming with Yearn Finance:
As an alternative, if the swap was from ETH to DAI, the script deposits the DAI into a Yearn Vault. Yearn Finance optimizes yield farming strategies, allowing the user to earn a higher yield through automatic reinvestment.

Monitoring & Earnings:
Once the assets are deposited in Aave or Yearn, the script enables you to monitor your earnings. You will accrue interest on Aave, while Yearn compounds yields automatically.

Withdraw Funds:
The script allows you to exit your position by withdrawing from Aave or Yearn. Once withdrawn, the funds are transferred back to your wallet.
### Protocols Used:
Uniswap: Token swap protocol for swapping between different ERC-20 tokens.
Aave: Lending protocol to deposit tokens and earn interest.
Yearn Finance (Optional): Yield optimization protocol to deposit stablecoins and earn optimized yields through automated strategies.
## Diagram Illustration
flowchart TD

    A[Start - User Wallet] --> B[Swap USDC for LINK on Uniswap]
    B --> C[Deposit LINK into Aave]
    C --> D[Monitor Earnings in Aave]

    A --> E[Optional - Swap ETH for DAI on Uniswap]
    E --> F[Deposit DAI into Yearn Vault]
    F --> G[Monitor Earnings in Yearn]

    D --> H[Withdraw from Aave]
    G --> I[Withdraw from Yearn]

  
The user begins with either USDC or ETH in their wallet.
Tokens are swapped using Uniswap, then deposited into either Aave for earning interest or Yearn Finance for yield farming.
The user monitors the earnings and can choose to withdraw their funds when necessary.
## Code Explanation
## 1. Environment Setup
The environment setup is handled by initializing the ethers.js library, which connects to the Ethereum Sepolia testnet using a provider (Infura/Alchemy) and a wallet to sign transactions.
```
require('dotenv').config();
const { ethers } = require('ethers');

// Connect to Sepolia testnet via Infura or Alchemy
const provider = new ethers.providers.JsonRpcProvider(process.env.INFURA_URL);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
```
Purpose: The script connects to the blockchain using the user's credentials, making it possible to interact with smart contracts.
Logic: The provider ensures access to the Sepolia testnet, while the wallet is required to sign and send transactions.
## 2. Uniswap Token Swap Function
This function enables the swap of one ERC-20 token (e.g., USDC) for another (e.g., LINK) via Uniswap’s router contract.
```
async function swapTokens(amountIn, tokenIn, tokenOut) {
    const uniswapRouter = new ethers.Contract(UNISWAP_ROUTER_ADDRESS, UNISWAP_ROUTER_ABI, wallet);

    // Approve the router to spend USDC
    const tokenContract = new ethers.Contract(tokenIn, ERC20_ABI, wallet);
    await tokenContract.approve(UNISWAP_ROUTER_ADDRESS, amountIn);

    // Define the token swap path and parameters
    const amountOutMin = ethers.utils.parseUnits('0', 18); // Accept any amount for simplicity
    const deadline = Math.floor(Date.now() / 1000) + 60 * 20; // 20 minutes from the current time

    // Execute the token swap
    const tx = await uniswapRouter.swapExactTokensForTokens(
        amountIn,
        amountOutMin,
        [tokenIn, tokenOut],
        wallet.address,
        deadline
    );
    
    await tx.wait();
    console.log(`Swapped ${amountIn} ${tokenIn} for ${tokenOut}`);
}
```
Purpose: This function performs the token swap between tokenIn (e.g., USDC) and tokenOut (e.g., LINK).
Logic: It first approves Uniswap to spend the tokens on behalf of the user, defines the swap parameters (such as minimum output and deadline), and executes the swap via the router contract.
## 3. Aave Lending (Supply LINK)
This function handles the deposit of LINK tokens into Aave’s lending pool, allowing the user to earn interest on the supplied tokens.
```
async function depositToAave(tokenAddress, amount) {
    const aaveLendingPool = new ethers.Contract(AAVE_LENDING_POOL_ADDRESS, AAVE_LENDING_POOL_ABI, wallet);

    // Approve the lending pool to spend LINK
    const tokenContract = new ethers.Contract(tokenAddress, ERC20_ABI, wallet);
    await tokenContract.approve(AAVE_LENDING_POOL_ADDRESS, amount);

    // Deposit LINK into Aave's lending pool
    const tx = await aaveLendingPool.deposit(tokenAddress, amount, wallet.address, 0);
    
    await tx.wait();
    console.log(`Deposited ${amount} of LINK into Aave`);
}
```
Purpose: Deposits LINK tokens into Aave’s lending pool to start earning interest.
Logic: The function approves Aave’s lending pool to spend the user's tokens and then executes the `deposit` transaction.
## 4. Optional Yield Farming with Yearn Finance
This function deposits DAI tokens into a Yearn Finance vault to participate in yield farming strategies for optimized returns.
```
async function depositToYearnVault(vaultAddress, amountDAI) {
    const yearnVault = new ethers.Contract(vaultAddress, YEARN_VAULT_ABI, wallet);

    // Approve the Yearn Vault to spend DAI
    const daiContract = new ethers.Contract(DAI_ADDRESS, ERC20_ABI, wallet);
    await daiContract.approve(vaultAddress, amountDAI);

    // Deposit DAI into the Yearn vault
    const tx = await yearnVault.deposit(amountDAI);
    
    await tx.wait();
    console.log(`Deposited ${amountDAI} DAI into Yearn Vault`);
}
```
Purpose: This function allows the user to deposit DAI into a Yearn Vault for yield farming.
Logic: It handles token approvals and deposits the user's DAI into the vault, where Yearn will manage the investment strategy for optimized returns
## 5. Withdrawing Funds
This function allows users to withdraw their assets from Aave, returning them to their wallet.
```
async function withdrawFromAave(tokenAddress, amount) {
    const aaveLendingPool = new ethers.Contract(AAVE_LENDING_POOL_ADDRESS, AAVE_LENDING_POOL_ABI, wallet);

    const tx = await aaveLendingPool.withdraw(tokenAddress, amount, wallet.address);
    await tx.wait();
    console.log(`Withdrew ${amount} of ${tokenAddress} from Aave`);
}
```
Purpose: Allows users to `withdraw` their tokens from Aave back into their wallet.
Logic: It uses the Aave lending pool’s withdraw method to release the deposited tokens and transfer them to the user's wallet.
## 6. Main Workflow
The main function orchestrates the full workflow of the script, facilitating token swaps, deposits, and optional yield farming.
```
async function main() {
    const amountToSwap = ethers.utils.parseUnits('100', 6); // Example amount: 100 USDC
    const linkAmount = ethers.utils.parseUnits('5', 18); // Example amount of LINK to deposit
    
    // Step 1: Swap USDC for LINK
    await swapTokens(amountToSwap, USDC_ADDRESS, LINK_ADDRESS);

    // Step 2: Deposit LINK into Aave
    await depositToAave(LINK_ADDRESS, linkAmount);

    // Optional: Use Yearn for DAI Yield Farming
    // await depositToYearnVault(YEARN_VAULT_ADDRESS, amountDAI);
}
main();
```
Purpose: The main function executes the full workflow: swapping tokens, depositing them into Aave (or Yearn), and optionally managing withdrawals.
Logic: It controls the order of operations and executes the functions sequentially to ensure smooth interaction with the DeFi protocols.

Summarizing all, the code is modular and designed to be easily extendable. It swaps tokens via Uniswap, deposits them into Aave to earn interest, and optionally deposits into Yearn Finance for yield farming.
