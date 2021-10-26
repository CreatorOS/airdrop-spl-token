# Distribute your own SPL token on Solana via Airdrop
In this quest we will be working on distributing custom tokens to the different  User Solana wallets via Airdrop. We will learn to airdrop the tokens to the user’s wallets by creating an associated account for the user and airdropping the custom token.
Hence in this quest we will be working on the airdrop feature of custom token.

### What are different ways of distributing tokens to the other wallets ?
1. Minting Token - (Covered in the previous quest)
2. Transfer Token - (Not covered in this quest, as it deserves it's own quest)
3. Airdrop Token - (Covered in this quest)

### Why distribute your Token ?
Creating our own token can become the backbone of the DApps but if we are not able to transfer it to other wallets then creating our own token serves no purpose.
Hence to understand all the details of minting tokens to other wallets is very important. Which eventually helps in increasing the total supply of the token.

### What will you get after going through this quest ?
1. ou will be able to airdrop your own tokens via the Minting process.
2. You will understand the difference between the key concepts like Minting Token, Transfer Token and Airdrop.
3. You will understand the importance of associated accounts while minting new tokens via airdrop.

### Prerequisites to work on this quest
1. Knowledge of crypto wallet (Phantom wallet specifically).
2. A basic react app with wallet connectivity feature.
3. Solana cluster can be : local, devnet or testnet.
4. For better understanding, please go through our first quest on creating a wallet connection with react app.

## Subquest : Setting up
### Folder structure expected of the react app to run this quest
* Assuming app name as "DistributeToken"
    *  App-Name
        * node_modules
        * public
        * src
            * utils
                * createAssociateAccountFromMintKey.js
                * createAssociatedAccountFromMintKeyAndMint.js
                * mintTokenToAssociatedAccount.js
            * App.js
        * index.js
        * package.json

### Basic terminologies to digest before we jump into the quest
1. **Airdrop Token**: This is the most expensive option of all the above methods as in this feature, there is no expectation that the user is going to have any Solana balance or Associated account with a token.
This method is very helpful when we do the ICO for the application or when we want to incentivise people to use the services.
2. **Minting new Token**: When you try to print new currency notes in the real world, it increases the supply of the notes which are already present in the market. In the same, if we need to increase the supply of our token in the market then we mint (print) the tokens to an associated account.
Minting process will increase the total supply of the custom token.
3. **Transfer Token**: In the real world when you want to lend some money to your friend, you do not ask the government to print more money and give it to you, so that you can lend. What you do is you lend your money whatever you have with you to your friend. This lending doesn't increase the total supply of the fiat currency notes. In the similar way, when a user wants to send custom tokens to another user, then the user transfers tokens to a friend's wallet without ever minting new tokens.
This does not increase the total supply of the custom token.

## Subquest : Airdrop of tokens via Minting

![Alt text](/learn_src/learn_assets/airdrop-token.png)

### Steps involved in Airdrop of Tokens in the next subquests

## Subquest :  Connect the application with mint owner wallet

Token distribution can only happen from the owner of the mint authority as it was discussed in the "Create SPL token - Web3" quest. Without the mint authority, none of the transactions for the distribution can take place and it will always through the Invalid Signature error.
Only mint authority owners can sign the transaction, hence connecting to the mint owner wallet is important.

File: src/App.js
```const provider = window.solana
provider.connect()
 
provider.on("connect", async() => {
console.log("wallet got connected")
setProviderPub(provider.publicKey)
});
provider.on("disconnect", () => {
console.log("Disconnected from wallet");
});
```

In the code

* First we are fetching the "solana" variable from the window object, if the solana variable is present that means the phantom wallet is installed as a chrome extension.
* Once we have the provider, then we are calling the "connect" function to connect with the phantom wallet. It will open a popup to ask the phantom password. Once you provide the correct password, it will connect the wallet to the application.
* Phantom provides a way to listen to the connected or disconnected events by using hooks like "connect" and "disconnect". Once phantom wallet is successfully authorised then it will trigger the "connect" method and we can use this method to start out DApp functionality

## Subquest :  Use the provider instance as the owner of all subsequent transactions.

Once the wallet is connected, then it becomes important to assign the provider as the owner which can be used throughout the feature to sign the transactions related to the token distribution.

File: src/App.js

```const owner = provider;```

## Subquest : Fetch the correct public key of mint

Correctness of the mintkey is very important as mintkey is also the token address which you want to mint. Each mint key is associated to a mint authority (wallet owner mostly). Hence having the correct pair of mintkey and mint authority key is important for minting process.
More info can be found in the "Create SPL token - Web3" quest.

Example: A mint public key is declared below, but it won't work for you as the owner of this mint key is different.

File: src/App.js

```const mintPubkey = 'Bw94Agu3j5bT89ZnXPAgvPdC5gWVVLxQpud85QZPv1Eb' //SOLG mint authority```

## Subquest : Fetch the correct public key of the Solana wallet of the user
In this step we will use the Solana wallet address of the user to which we want to do the airdrop. This process we will create the Associated Account for Solana Wallet Address of the User (SWAU). Associated Account mapped with the correct Mint Key (MK) is important to receive the tokens which are of (MK) type.

```const userPubKey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz'// receiver Solana wallet address```

As per the Solana blockchain, you can not receive/send the tokens to an Associated Account which is not mapped with the correct (MK) of the token which needs to be sent/received.

Hence creating the right associated account is very important in this Airdrop Process.

Let’s dive into the process of creating an Associated Account for the user.

## Subquest: Create Associated Account for the user

In this step we will create the Associated Account (AC) for the user against the Solana wallet address fetched in the previous step.

Steps involved before creating the AC :
1. Create an address from the user’s Solana address and Mint Key (MK).
2. Fetch the account info of the address which was generated in the step1.
3. If the result of step 2 is non null then AC already exists for the user, else we need to proceed with the AC creation step
4. If step 3 fails, then we will create a new instruction to create the associated account

In the below function, below
* connection : It is the connection instance to the solana cluster
* payer: It is the instance of the connected wallet i.e mint authority owner wallet
* mintPubkey : Token address which will be minted to mint new tokens.
* ownerPubkey: Solana wallet address of the receiver’s wallet i.e (SWAU)
* associatedTokenPubkey : It is the address of the associated account if the user already has.
* doesAccountExist: It is a bool variable.
    * true: It means an associated account already exists, hence no need to create a new account.
    * false:  It means there is no associative account exists yet for the wallet with the mintPubkey. Hence an associative needs to be created by mapping it with mintPubkey, so that the custom SPL token can be transferred of the mintPubkey type.

createAssociateAccountFromMintKey:
This function is a wrapper to process multiple things before creating the associated account.
1. Check if the user has passed associated account key
2. If no associated account key passed then create a new key pair (NKP) which can be used to create an associated account by using findAssociatedTokenAddress
3. Once a new key pair (NKP) is created then check if NKP already exists as an associated account by using getAccountInfo
4. If NKP exists as associated account then no need to create new associated account else proceed to step 5
5. In this step, NKP will be mapped with mint key (MK) by creating a **createAssociatedTokenAccountInstruction**  instruction which will be used to create the new account.
6. ownerPubkey : This public key will act as an owner of the new associated account. In our case it will be the SWAU which we received in the first step.

**findProgramAddress**

This function will help us to get the key pair which will be the mapped version of mintPubkey and SWAU. If the associated account doesn’t exist already then this function will return a new keypair which can be used to create an associated account.

Below parameters are required for this function to return valid keypair
walletAddress: Address of the User’s Solana wallet

TOKEN_PROGRAM_ID: It is the ID of the on-chain Token program which has all the methods implemented to create a token.

tokenMintAddress: It is the public key of the mint key which needs to be mapped to create the associated account.

SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID: It is the ID of the on-chain Associated Token program which has all the methods implemented to create an associated account.

File: createAssociateAccountFromMintKey.js
```import * as splToken from '@solana/spl-token'
import {PublicKey, Transaction} from '@solana/web3.js';
const TOKEN_PROGRAM_ID = "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"
const SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID = "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL"
/**
* @remarks
* A utility function to create the associative account
* @param {*} connection // connection to the cluster
* @param {*} payer // payer of the transaction, if any takes place
* @param {*} mintPubKey // public key of the mint authority, with whom the associative accounts needs to be mapped
* @param {*} ownerPubkey // owner public key, most of the time minPubkey will be same
* @param {*} associatedTokenPubkey // the account which needs to be mapped/created as an associative account
* @returns
*/
export const createAssociateAccountFromMintKey = async (connection, payer, mintPubkey, ownerPubkey, associatedTokenPubkey) => { 
  if(!associatedTokenPubkey){
   /**
    * If no Associated account passed, then we need to create one.

 
  */
   associatedTokenPubkey = await findAssociatedTokenAddress(ownerPubkey, mintPubkey)
 }


 /**
  * Check if the user's wallet already has a associative account with the mintPubKey
  *
  */
 const doesAccountExist = await connection.getAccountInfo(associatedTokenPubkey)

 /**
  * if doesAccountExist is false, that means there is no associative account exists yet for the wallet with the mintPubkey,
  * Hence an associative needs to be created by mapping it with mintPubkey, so that the custom SPL token can be transferred of the mintPubkey type
  *
  * Once we map the associative account, then we will be able to transfer or mint the custom tokens on the associative account
  */
 if(!doesAccountExist){
   const transaction = new Transaction().add(
     splToken.Token.createAssociatedTokenAccountInstruction(
       SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID,
       TOKEN_PROGRAM_ID,
       mintPubkey,
       associatedTokenPubkey,
       ownerPubkey,
       payer.publicKey
     )
   );
   transaction.feePayer = payerPubkey;
   console.log('Getting recent blockhash');
   transaction.recentBlockhash = (
     await connection.getRecentBlockhash()
   ).blockhash;
   let signed = await payer.signTransaction(transaction);
   let signature = await connection.sendRawTransaction(signed.serialize());
   let confirmed = await connection.confirmTransaction(signature);
  
   return {status: true, signature: confirmed, associatedTokenPubkey};
 }else{
   return {status: true, associatedTokenPubkey}
 }


, associatedTokenPubkey};
 }else{
   return {status: true, associatedTokenPubkey}
 }

 
return (await PublicKey.findProgramAddress(
       [
           walletAddress.toBuffer(),
           TOKEN_PROGRAM_ID.toBuffer(),
           tokenMintAddress.toBuffer(),
       ],
       SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID
   ))[0];
}
```

## Subquest : Use the correct public key of the associated account.
Correctness of the associated public key is also very important. Associated accounts mapped with the above mint key can only receive these tokens. It cannot accept any other tokens. As associated accounts are associated with a particular token. Hence having the correct associated public key is important to receive the minted token.

Example: An associatedAccountPubkey is fetched from the previous subquest, but it won't work for you as the mapped mint key of this account is different from the mint authority.

File: src/App.js
const associatedAccountPubkey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz'//SOLG mint authority

## Subquest : mintToken utility function initiate the mint process
In this function, we are using the below parameters

connection : Connection instance to the solana cluster. Prefer to use devnet for the non-production application.

owner: Wallet provider by which the application is connected with.
mintOwner: Mint Owner account will be the same as owner in most of the case while creating the new mint.

amount: Number of tokens which needs to be transferred in Lamports.
mintPubkey: This is the public key of the mint, which you received while creating the new mint authority.

associatedAccountPubkey: This is the public key of the receiver's associated account which was created by mapping  it with mintPubkey.

transaction.add : This statement will add mintTo internal function which will add the mint transaction schema.

mintAuthorityPubkey: This is the public key of the mint authority who has the rights to sign or decline the transaction.

signAndSendTransaction: This is the internal function which will verify the transaction by signing it.

File: src/utils/mintTokenToAssociatedAccount.js
```async function mintToken({
connection,
owner,
mintOwner, // Account to hold token information
amount, // Number of tokens to issue
mintPubkey,
associatedAccountPubkey, // Account to hold newly issued tokens, if amount > 0
}) {
let transaction = new Transaction();
let signers = [mintOwner];
transaction.add(
mintTo({
//when sending for the first time where mint is derived for the first time
mintPubkey,
//when sending for the first time where initialAccount is derived for the first time to receive the newly created SPL token
destination: associatedAccountPubkey,
amount,
mintAuthorityPubkey: owner.publicKey,
}),
);
return await signAndSendTransaction(connection, transaction, mintOwner, signers);
}
```

## Subquest : mintTo utility function to complete the mint process
This utility function will return the transaction object which will hold the instructions to make the transaction.

**keys**: It is an array of objects which contains different publicKeys which can be used in the transaction to perform relevant actions like send a token, create a new account, etc... Each object contains properties like pubkey, isSigner, isWritable.

**pubkey**: publicKey which will be used in the transaction<br>
**isSigner**: boolean variable

        true: publicKey will be used as a signer
        false: publicKey will not be used as a signer

**isWritable**: boolean variable

        true: publicKey’s account will be writable, that means the signer can update the values
        false: publicKey’s account can not be writable, that means the signer can not update any value

In this case it will be<br>
**mintPubkey** : Token address which will be minted to mint new tokens.<br>
**destination**: This is the pubkey of the destination account.<br>
**minAuthorityPubKey**: This is the public key of the mint authority who can perform the action like sign the transaction to mint new tokens.<br>
Except the mintAuthorityPubkey public key  isSigner:true other keys can be false. As only mintAuthorityPubkey will be used and has the rights to sign the transaction.
**TransactionInstruction**: It will create the instruction by encoding the instruction which will be decoded by the on-chain program to execute the mint.
**encodeTokenInstructionData**: It's a utility function to encode each instruction in bytes array and make it              compatible with on-chain solana programs.
**TOKEN_PROGRAM_ID** : It is the public key of the deployed on-chain program for the Solana Token, which has all the inbuilt functions to create the mint authority and associated accounts.

File: src/utils/mintTokenToAssociatedAccount.js
```function mintTo({ mintPubkey, destination, amount, mintAuthorityPubkey }) {
let keys = [
{ pubkey: mintPubkey, isSigner: false, isWritable: true },
{ pubkey: destination, isSigner: false, isWritable: true },
{ pubkey: mintAuthorityPubkey, isSigner: true, isWritable: false },
];
return new TransactionInstruction({
keys,
data: encodeTokenInstructionData({
mintTo: {
amount,
},
}),
programId: TOKEN_PROGRAM_ID,
});
}
```

## Subquest : Signing the transactions
### Once we are ready with transaction, then signing it is important - signAndSendTransaction

In this step we will be going through the flow of signing the mint transaction. To complete the mint process we need to write our block to the solana blockchain. To get this transaction on the blockchain it needs to be signed from the wallet owner and sent for the validation.
Few key variables to be used in above function.

* transaction.recentBlockhash : As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated with the new transaction. Without the getRecentBlockhash, the validators will not be able to verify the transaction.
* proof of history: Proof of History is a sequence of computation that can provide a way to cryptographically verify passage of time between two events. Validator nodes "timestamp" blocks with cryptographic proofs that some duration of time has passed since the last proof. All data hashed into the proof most certainly have occurred before the proof was generated. The node then shares the new block with validator nodes, which are able to verify those proofs. The blocks can arrive at validators in any order or even could be replayed years later. With such reliable synchronization guarantees, Solana is able to break blocks into smaller batches of transactions called entries. Entries are streamed to validators in realtime, before any notion of block consensus.
This is very different and efficient from the Proof of Work or Proof of Stake where the energy consumption is high as every node in the blockchain has to perform the action to validate a node. The winner gets the reward but other validator’s work gets wasted in the process.
* wallet.publicKey: Every transaction needs a fee to be paid to the computers who are making the transaction possible. This parameter tells Solana who will be paying the fees for this transaction.

    ```wallet.publicKey = provider.publicKey```
In Ethereum, we have smart contracts and ethereum accounts which deploys these smart contracts. Hence the address of the smart contract is used to fetch the public function and interact with the smart contract. But there are few functions in smart contracts which can only be executed by the owner of the smart contract. 

* signers.map: List of accounts which will be used to sign the ongoing transaction, that is defined by this list of signers. If any action is going to be taken on this account, it requires the signature using the account’s private key. This ensures that the program never updates the account of a user without the permission of the owner of that account.

* transaction.serialize() : All the data must be stored on the blockchain. To keep the content format agnostic of the programming language used, the data is serialized before storing. 

* skipPreflight: It’s a bool data type
true: preflight transaction checks for the available methods before sending the transaction, which involves a very little latency.
false: (default value) : It is turned off by default to save the network bandwidth.

* preflightCommitment: Every transaction on Solana goes through a process of finalization. The longer the transaction has existed on the blockchain, the less likely it is to get reverted. A transaction is reverted when the blockchain realizes later that the transaction was actually not supposed to be allowed. However, the process of finalization takes some time. Depending on the security required in your code, you can choose between single, confirmed and finalized security levels. Single means that there is atleast one “confirmation” given to the transaction by a validator on Solana.

File: src/utils/mintTokenToAssociatedAccount.js
```async function signAndSendTransaction(
connection,
transaction,
wallet,
signers,
skipPreflight = false,
) {
transaction.recentBlockhash = (
await connection.getRecentBlockhash('max')
).blockhash; // As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated
// Without the getRecentBlockhash, the validators will not be able to verify the transaction
transaction.setSigners(
// fee payed by the wallet owner
wallet.publicKey,
...signers.map((s) => s.publicKey), // we will add all the intermediate signers required for the transaction, in our case mint keypair is also required.
// Hence we have added the mint signer from the signers array.
);
transaction = await wallet.signTransaction(transaction);
const rawTransaction = transaction.serialize(); //Serialization is the process of converting an object into a stream of bytes,
//which can be used by on-chain programs to again de-serialize it to read the instructions
//and perform actions on it.
return await connection.sendRawTransaction(rawTransaction, {
skipPreflight, //preflight transaction check checks for the available methods before sending the transaction, which involves a very little latency
//that is why skipPreflight is generally kept false, to save the network bandwidth.
preflightCommitment: 'single',
//For preflight checks and transaction processing,
//Solana nodes choose which bank state to query based on a commitment requirement set by the client.
//The commitment describes how finalized a block is at that point in time. When querying the ledger state,
//it's recommended to use lower levels of commitment to report progress and higher levels to ensure the state will not be rolled back.
//For processing many dependent transactions in series, it's recommended to use "confirmed" commitment,
//which balances speed with rollback safety. For total safety, it's recommended to use"finalized" commitment.
});
}
```

## Subquest : Making our transaction Blockchain compliant
LAYOUT class will help us to build the structure in the required format as the same as the struct format required by the Token on-chain program to read the instructions and process it.
BufferLayout provides us the conversion of Layout to an array of bytes called Encoding which can be transferred over the network and Decoding takes place in the solana on-chain program to find the instruction.
On-chain programs perform the action based on the instructions it receives in the form of struct.

It also provides us the options to define the exact data types required by the solana on-chain program to process. Like u8, struct


**Key points to know:**

u8 : unsigned 8 bit integer, which means only positive integer will be supported of 8 bit

Example: BufferLayout.u8('instruction') means 'instruction' is a variable of an unsigned 8 bit integer.<br>
struct: is equivalent to class but it's a very light weight data type as compared to classes in lower level languages like c++,c.<br>
Example: BufferLayout.struct([BufferLayout.nu64('amount')])  here we are defining the struct with the amount variable with its respective data type.<br>
encodeTokenInstructionData: It's a utility function to encode each instruction in bytes array and make it compatible with on-chain Solana programs.

File: src/utils/mintTokenToAssociatedAccount.js
```const LAYOUT = BufferLayout.union(BufferLayout.u8('instruction'));
LAYOUT.addVariant(
7,
BufferLayout.struct([BufferLayout.nu64('amount')]),
'mintTo',
);
```
## Subquest :  Wrapper function to create an associated account and mint new tokens to it.
In this wrapper function the objective is to wrap the two files/functions together to build the airdrop flow.<br>
The functions this wrapper invokes are
1. createAssociateAccountFromMintKey - To check if the associated account already exists. If not then create the account and return it to the parent function ie. createAssociatedAccountFromMintKeyAndMint
2. mintTokenToAssociatedAccount - This utility function will mint new tokens to the passed associated account.

```import { createAssociateAccountFromMintKey } from "./createAssociateAccountFromMintKey";
import { mintTokenToAssociatedAccount } from "./mintTokenToAssociatedAccount";


export const createAssociatedAccountFromMintKeyAndMint = (connection, provider, mintPubkey, ownerPubkey, associatedTokenPubkey, tokensToMint) => {
   const result = await createAssociateAccountFromMintKey(connection, provider, mintPubkey, ownerPubkey, associatedTokenPubkey)

   if(!result.status){
       return { status: false, error:"Error in creating the associated account"}
   }

   const tokenMintingResult = await mintTokenToAssociatedAccount(provider,connection,tokensToMint,mintPubkey,associatedTokenPubkey,provider)

   if(!tokenMintingResult.status){
       return {status: false, error: `Error in minting the associated account ${result.associatedTokenPubkey}`}
   }

   return {
       status: true,
       transactionSignature: tokenMintingResult.signature
   }
}
```

## Subquest :  Final code to mint new tokens and distribute it to the associated account

File: src/utils/createAssociatedAccountFromMintKeyAndMint.js
```
import { createAssociateAccountFromMintKey } from "./createAssociateAccountFromMintKey";
import { mintTokenToAssociatedAccount } from "./mintTokenToAssociatedAccount";


export const createAssociatedAccountFromMintKeyAndMint = (connection, provider, mintPubkey, ownerPubkey, associatedTokenPubkey, tokensToMint) => {
   const result = await createAssociateAccountFromMintKey(connection, provider, mintPubkey, ownerPubkey, associatedTokenPubkey)

   if(!result.status){
       return { status: false, error:"Error in creating the associated account"}
   }

   const tokenMintingResult = await mintTokenToAssociatedAccount(provider,connection,tokensToMint,mintPubkey,result.associatedTokenPubkey,provider)

   if(!tokenMintingResult.signature){
       return {status: false, error: `Error in minting the associated account ${result.associatedTokenPubkey}`}
   }

   return {
       status: true,
       transactionSignature: tokenMintingResult.signature
   }
}
```

File: src/utils/createAssociateAccountFromMintKey.js
```
import * as splToken from '@solana/spl-token'
import {PublicKey, Transaction} from '@solana/web3.js';
const TOKEN_PROGRAM_ID = "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"
const SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID = "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL"
/**
* @remarks
* A utility function to create the associative account
* @param {*} connection // connection to the cluster
* @param {*} payer // payer of the transaction, if any takes place
* @param {*} mintPubKey // public key of the mint authority, with whom the associative accounts needs to be mapped
* @param {*} ownerPubkey // owner public key, most of the time minPubkey will be same
* @param {*} associatedTokenPubkey // the account which needs to be mapped/created as an associative account
* @returns
*/
export const createAssociateAccountFromMintKey = async (connection, payer, mintPubkey, ownerPubkey, associatedTokenPubkey) => { 
  if(!associatedTokenPubkey){
   /**
    * If no Associated account passed, then we need to create one.
    */
   associatedTokenPubkey = await findAssociatedTokenAddress(payer.publicKey, mintPubkey)
 }

 /**
  * Check if the user's wallet already has a associative account with the mintPubKey
  *
  */
 const doesAccountExist = await connection.getAccountInfo(associatedTokenPubkey)

 /**
  * if doesAccountExist is false, that means there is no associative account exists yet for the wallet with the mintPubkey,
  * Hence an associative needs to be created by mapping it with mintPubkey, so that the custom SPL token can be transferred of the mintPubkey type
  *
  * Once we map the associative account, then we will be able to transfer or mint the custom tokens on the associative account
  */
 if(!doesAccountExist){
   const transaction = new Transaction().add(
     splToken.Token.createAssociatedTokenAccountInstruction(
       SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID,
       TOKEN_PROGRAM_ID,
       mintPubkey,
       associatedTokenPubkey,
       ownerPubkey,
       payer.publicKey
     )
   );
   transaction.feePayer = payer.publicKey;
   console.log('Getting recent blockhash');
   transaction.recentBlockhash = (
     await connection.getRecentBlockhash()
   ).blockhash;
   let signed = await payer.signTransaction(transaction);
   let signature = await connection.sendRawTransaction(signed.serialize());
   let confirmed = await connection.confirmTransaction(signature);
  
   return {status: true, signature: confirmed, associatedTokenPubkey};
 }else{
   return {status: true, associatedTokenPubkey}
 }

}


async function findAssociatedTokenAddress(
 walletAddress,
 tokenMintAddress
) {
   return (await PublicKey.findProgramAddress(
       [
           walletAddress.toBuffer(),
           TOKEN_PROGRAM_ID.toBuffer(),
           tokenMintAddress.toBuffer(),
       ],
       SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID
   ))[0];
}

```

File: src/utils/mintTokenToAssociatedAccount.js
```import { TransactionInstruction, Transaction } from "@solana/web3.js";
import * as BufferLayout from 'buffer-layout';
import { TOKEN_PROGRAM_ID } from "./programIds";
const LAYOUT = BufferLayout.union(BufferLayout.u8('instruction'));
LAYOUT.addVariant(
7,
BufferLayout.struct([BufferLayout.nu64('amount')]),
'mintTo',
);
/**
* @why : This function will enable you to mint new tokens to user's associated account
*
* @Remarks: mintTokenToAssociatedAccount
* A utility which can mint new custom SPL token to the existing associative token account,
* which will directly increase the total supply of the custom SPL token.
* @param {*} wallet //wallet provider, to sign the transaction in case mintOwner is not same as wallet provider
* @param {*} connection //connection instance to the Solana Cluster
* @param {*} tokensToMint //Number of custom SPL tokens to mint
* @param {*} mintPubkey //Mint's public Key i.e custom SPL token's public key which was created by the new createNewMintAuthority process.
* @param {*} associatedAccountPubkey //Public key of the account mapped to Mint's public key which was created by the user or SPL token mint authority.
* @param {*} mintOwner //mintOwner is the mint authority of the mint i.e. custom token, mostly it will be the same as the wallet provider who is going to make the transaction.
* @returns
* {status: false, error:message } //in case of any failure
* {status: true, signature} //in case of transaction success
*
*/
export const mintTokenToAssociatedAccount = async (wallet, connection, tokensToMint, mintPubkey, associatedAccountPubkey, mintOwner) =>{
if(!tokensToMint){
return {status: false, error:"You can't mint 0 tokens"}
}
const tokensToMintInLamports = tokensToMint * 1000000000;
const signature = await mintToken({
connection,
owner: wallet,
mintOwner,
amount: tokensToMintInLamports,
mintPubkey,
associatedAccountPubkey
})
console.log("Waiting for the signature confirmation")
await connection.confirmTransaction(signature);
console.log("Signature confirmed")
return {signature};
}
async function signAndSendTransaction(
connection,
transaction,
wallet,
signers,
skipPreflight = false,
) {
transaction.recentBlockhash = (
await connection.getRecentBlockhash('max')
).blockhash; // As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated
// Without the getRecentBlockhash, the validators will not be able to verify the transaction
transaction.setSigners(
// fee paid by the wallet owner
wallet.publicKey,
...signers.map((s) => s.publicKey), // we will add all the intermediate signers required for the transaction, in our case mint keypair is also required.
// Hence we have added the mint signer from the signers array.
);
transaction = await wallet.signTransaction(transaction);
const rawTransaction = transaction.serialize(); //Serialization is the process of converting an object into a stream of bytes,
//which can be used by on-chain programs to again de-serialize it to read the instructions
//and perform actions on it.
return await connection.sendRawTransaction(rawTransaction, {
skipPreflight, //preflight transaction check checks for the available methods before sending the transaction, which involves a very little latency
//that is why skipPreflight is generally kept false, to save the network bandwidth.
preflightCommitment: 'single',
//For preflight checks and transaction processing,
//Solana nodes choose which bank state to query based on a commitment requirement set by the client.
//The commitment describes how finalized a block is at that point in time. When querying the ledger state,
//it's recommended to use lower levels of commitment to report progress and higher levels to ensure the state will not be rolled back.
//For processing many dependent transactions in series, it's recommended to use "confirmed" commitment,
//which balances speed with rollback safety. For total safety, it's recommended to use"finalized" commitment.
});
}
async function mintToken({
connection,
owner,
mintOwner, // Account to hold token information
amount, // Number of tokens to issue
mintPubkey,
associatedAccountPubkey, // Account to hold newly issued tokens, if amount > 0
}) {
let transaction = new Transaction();
let signers = [mintOwner];
transaction.add(
mintTo({
//when sending for the first time where mint is derived for the first time
mintPubkey,
//when sending for the first time where initialAccount is derived for the first time to receive the newly created SPL token
destination: associatedAccountPubkey,
amount,
mintAuthorityPubkey: owner.publicKey,
}),
);
return await signAndSendTransaction(connection, transaction, mintOwner, signers);
}
function mintTo({ mintPubkey, destination, amount, mintAuthorityPubkey }) {
let keys = [
{ pubkey: mintPubkey, isSigner: false, isWritable: true },
{ pubkey: destination, isSigner: false, isWritable: true },
{ pubkey: mintAuthorityPubkey, isSigner: true, isWritable: false },
];
return new TransactionInstruction({
keys,
data: encodeTokenInstructionData({
mintTo: {
amount,
},
}),
programId: TOKEN_PROGRAM_ID,
});
}
const instructionMaxSpan = Math.max(
...Object.values(LAYOUT.registry).map((r) => r.span),
);
function encodeTokenInstructionData(instruction) {
let b = Buffer.alloc(instructionMaxSpan);
let span = LAYOUT.encode(instruction, b);
return b.slice(0, span);
}
```

## Subquest : How to run this quest
1. Install phantom wallet chrome extension and add some SOL to the wallet.
2. Create a react app 
    * npx create-react-app distribute-token
    * cd distribute-token
3. Copy paste the below initiator code in the App.js to invoke mintTokenToAssociatedAccount.js

```import './App.css';
import { Connection, PublicKey } from "@solana/web3.js";
import * as web3 from '@solana/web3.js';
import { createNewMintAuthority } from './utils/createNewMintAuthority';
import { useEffect, useState } from 'react';
import { mintTokenToAssociatedAccount } from './utils/mintTokenToAssociatedAccount';
import { transferCustomToken } from './utils/transferCustomToken';
import { createAssociatedAccountFromMintKeyAndMint } from './utils/createAssociatedAccountFromMintKeyAndMint';
const NETWORK = web3.clusterApiUrl("devnet");
const connection = new Connection(NETWORK);
const decimals = 9

function App() {

 const [provider, setProvider] = useState()
 const [providerPubKey, setProviderPub] = useState()
 const [mintSignature, setMintTransaction] = useState()
 const [tokenSignature, setTokenTransaction] = useState()
 const [tokenSignatureAirdrop , setAirdropTransaction] = useState()

 const mintNewToken = async () =>{
   if(provider && !provider.isConnected){
       provider.connect()
     }
   try{
     const mintResult = await createNewMintAuthority(provider, decimals, connection)
     console.log(mintResult.signature,'--- signature of the transaction---')
     console.log(mintResult.mintAccount,'----mintAccount---')
   }catch(err){
     console.log(err
       )
   }
}

const mintTokenToAssociateAccountHandler = async () =>{
   try{
     const tokensToMint = 1
     const mintPubkey = 'Bw94Agu3j5bT89ZnXPAgvPdC5gWVVLxQpud85QZPv1Eb' //SOLG mint authority
     const associatedAccountPubkey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz'
     const transactionSignature = await mintTokenToAssociatedAccount(provider, connection, tokensToMint, new PublicKey(mintPubkey), new PublicKey(associatedAccountPubkey), provider)
     setMintTransaction(transactionSignature.signature)
   }catch(err){
     console.log(err)
   }
 }


const transferTokenToAssociateAccountHandler = async () =>{
   try{
     const tokensToMint = 1
     const fromCustomTokenAccountPubkey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz' //associated account's public key of the connected wallet
     const toAssociatedAccountPubkey = 'EfhdzcbMiAToWYke12ZqN8PmYmyEgRWdFaSKBEhxXYir' //associated account's public key of the receiver's wallet
     const transactionSignature = await transferCustomToken(provider, connection, tokensToMint, new PublicKey(fromCustomTokenAccountPubkey), new PublicKey(toAssociatedAccountPubkey))
     setTokenTransaction(transactionSignature.signature)
   }catch(err){
     console.log(err)
   }
}

const airdropToUserWallet = async () =>{
 try{
   const tokensToAirdrop = 1
   const mintPubkey = 'Bw94Agu3j5bT89ZnXPAgvPdC5gWVVLxQpud85QZPv1Eb' // mintKey of the token to be minted
   const ownerPubkey = '4deyFHL6LG6NYvceo7q2t9Bz66jSjrg8A1BxJH1wAgie' //receiver's Solana wallet address (SWAU)
  
   const transactionSignature = await createAssociatedAccountFromMintKeyAndMint(connection, provider, new PublicKey(mintPubkey), new PublicKey(ownerPubkey),"",tokensToAirdrop)
   setAirdropTransaction(transactionSignature.transactionSignature)
 }catch(err){
   console.log(err)
 }
}

const connectToWallet = () =>{
 if(!provider && window.solana){
   setProvider(window.solana)
 }
 if(!provider){
   console.log("No provider found")
   return
 }
 if(provider && !provider.isConnected){
   provider.connect()
 }
}


 useEffect(() => {
   if (provider) {
       provider.on("connect", async() => {
         console.log("wallet got connected")
         setProviderPub(provider.publicKey)

       });
       provider.on("disconnect", () => {
         console.log("Disconnected from wallet");
       });
   }
 }, [provider]);

 useEffect(() => {
   if ("solana" in window && !provider) {
     console.log("Phantom wallet present")
     setProvider(window.solana)
   }
 },[])

 return (
   <div className="App">
     <header className="App-header">
        
          <button onClick={connectToWallet}> {providerPubKey ? 'Connected' : 'Connect'} to wallet {providerPubKey ? (providerPubKey).toBase58() : ""}</button>
          {/* <button onClick={mintNewToken}>Create new token</button>
          <button onClick={mintTokenToAssociateAccountHandler}> {mintSignature ? `Minted new token, signature: ${mintSignature}`: 'Mint New Token'} </button>
          <button onClick={transferTokenToAssociateAccountHandler}> {tokenSignature ? `Token transferred, signature: ${tokenSignature}`:'Transfer Token' } </button> */}
          <button onClick={airdropToUserWallet}> {tokenSignatureAirdrop ? `Token airdropped, signature: ${tokenSignatureAirdrop}`:'Airdrop Token' } </button>
     </header>
   </div>
 );
}

export default App;
```

4. To install the required dependencies:  “npm install” from the root folder of the project.
5. To run the project : “npm run start” from the root folder of the project.
6. Navigate to http://localhost:3000 
7. Click on the 'Connect to wallet' button to connect with the wallet.
8. Once connected, click on "Airdrop token" to airdrop a new token to the given user’s wallet address

## Subquest : What next?
### What can you build taking this quest as a base ?
* Distribute your own token to the user’s solana account via Airdrop.
* You can increase the total supply of your custom tokens by Airdrop campaigns.
* You can plan the IDO of the custom token based on the business logic.
* Incentivise the user if they use your services with your custom tokens.


