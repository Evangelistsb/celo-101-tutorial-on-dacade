# Build a Bank Dapp With React-Celo 

## Introduction :book:
If you were previously a Web2 developer and recently got interested in the Web3 world either because of the noise on Twitter or the never-ending meetups held by Web3 folks, or you are already a Web3 hacker, whichever categories you call into, this tutorial is for you. We want to build an amazing project using **react-celo**, a package provided by the great Celo developers team, also write the smart contract with solidity, then deploy it to the **Celo blockchain**. Grab a cup of coffee because these will be a long and interesting ride.

## Table of Contents
  * [Introduction :book:](#introduction--book-)
  * [Table of Contents](#table-of-contents)
  * [Learning Objectives :eyeglasses:](#learning-objectives--eyeglasses-)
  * [Tech Stack :computer:](#tech-stack--computer-)
  * [Prerequisites :closed_book:](#prerequisites--closed-book-)
  * [A Glance at the App :eyes:](#a-glance-at-the-app--eyes-)
    + [Deposit](#deposit)
    + [Withdraw](#withdraw)
    + [Transfer](#transfer)
  * [Smart Contract Section :hammer_and_wrench:](#smart-contract-section--hammer-and-wrench-)
    + [Coding the Smart Contract :writing_hand:](#coding-the-smart-contract--writing-hand-)
    + [Deploy smart contract :arrow_up:](#deploy-smart-contract--arrow-up-)
  * [UI Section :desktop_computer:](#ui-section--desktop-computer-)
- [Conclusion :end:](#conclusion--end-)
- [FAQ :question:](#faq--question-)


## Learning Objectives :eyeglasses: 
By the end of this tutorial, you will have learned the following
- How to write a smart contract with Solidity
- How to build an amazing UI using React
- How to connect react ui with the Celo blockchain using react-celo
- How to deploy solidity codes to Celo using Remix

## Tech Stack :computer: 
These following tools were used for this tutorial
- [React](https://reactjs.org/)
- [React-Celo](https://github.com/celo-org/react-celo)
- [Remix IDE](https://remix.ethereum.org/)
- [NodeJs](https://nodejs.org/en/download/)
- [NPM()Node Package Manager](https://www.npmjs.com/)
- [VSCode](https://code.visualstudio.com/download)
- HTML and CSS
- [Solidity](https://docs.soliditylang.org/#:~:text=Solidity%20is%20an%20object%2Doriented,Ethereum%20Virtual%20Machine%20(EVM).)

## Prerequisites :closed_book: 
Before starting this tutorial, make sure you are equiped with the following:
- solidity smart contract basics
- javascript basics
- experience using the command line
- Celo Extension Wallet. If you dont have one, [download](https://chrome.google.com/webstore/detail/celoextensionwallet/kkilomkmpmkbdnfelcpgckmpcaemjcdh?hl=en).

## A Glance at the App :eyes: 
Before we continue, let us take a glance at the application we want to build for this tutorial. It is a bank application that simulate the operations of banks in the real world. It allows users of the app to do the following:
- Create an account with the bank using username alone.
- Deposit money into the bank.
- Withdraw money deposited into the bank.
- Transfer money from one account to another. 
In addition, logged-in users can see last time a deposit was made into their account, they can also see last time a withdraw was made from their account.

Let us interact with the app and see how it feels before building it.

This is the landing page of the app

![1-landing-page](https://i.imgur.com/IBhkG4J.png)


The first thing to do after visiting the landing page is to click on the **Connect** button to connect your wallet to the app. We used react-celo package to access our wallet and connect to the Celo blockchain. After clicking on the button, a pretty looking modal designed by the Celo team will appear. This modal prompt users to choose from a variety of wallets. 

![2-react-celo-wallet-interface](https://i.imgur.com/IMksEGO.png)

Select **Celo Extension Wallet** and the wallet will pop up with a new window asking you to connect to the dapp.

After connecting Celo extension wallet, your dashboard will change to something similar to this.

![3-sign-up-page](https://i.imgur.com/EsP9Fmz.png)


Enter your username and click on the sign up button. It will pop up the Celo extension wallet, asking you to confirm the transaction. Click on confirm and the transaction will go through. After some seconds, the transaction will complete. Refresh the page to see the changes (You might have to connect your wallet again).

![3b-dashboard](https://i.imgur.com/WBRynA3.png)


Let us now perform some actions with the app.

### Deposit

Let us try to deposit into the app by entering the amount and clicking on the large green **deposit** button on the right hand side of the page. Wait for some seconds for the transaction to complete, then refresh the page. The balance you just deposited will reflect in your dashboard like this:

![4-dashboard-update-after-deposit](https://i.imgur.com/xRkltWN.png)


### Withdraw

Initially, we made a deposit of 3 CELO into our bank acount, and it has reflected on our dashboard. Now let's withdraw 1 CELO out of the 3 CELO we sent. 
Enter the amount to be withdrawN into the field, then click on the 'withdraw' button. Wait for the transaction to complete, then refresh the page to see changes.

![5-balance-update-after-withdraw](https://i.imgur.com/RaOW4jl.png)


Notice that our balance from the bank account has dropped and our Celo Extension wallet balance has increased

### Transfer
This is the last test we will carry out in out Dapp. We will transfer from the 2 CELO we have to another account. Before you continue, create another account in your celo extension wallet and switch to the account. Use the account to sign up on the Dapp and then copy the account address. Switch back your account to the first one, then try to make a transfer by entering the address of the second account you just created followed by the amount. Click on the transfer button, wait for some seconds for the transaction to complete. After completion, here is our dashboard.

![6-dashboard-after-transfer](https://i.imgur.com/LSk99uN.png)


This is the dashboard of our second account.

![7-dashboard-of-second-account-after-transfer](https://i.imgur.com/5bI60IQ.png)


From what you have see so far, I bet you can't wait to buiild this Dapp on you own.


## Smart Contract Section :hammer_and_wrench: 
In this section, we will write the smart contract for the app you just tested above. We will write the smart usng Solidity in Remix IDE. Inside this same section, we will also deploy the smart contract to the blockchain using the user interface(ui) provided by remix for us.

### Coding the Smart Contract :writing_hand: 
Visit [Remix](https://remix.ethereum.org) from your browser. You can use either brave, chrome, or firefox, whichever one that you are more convenient with.

Create a new file with the name `Bank.sol` and open the file to start writing your solidity code inside it.

We will first show you the finished code before explaining every piece of it. 

paste the code below into your remix IDE.

```sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract BankContract {
    struct Account {
        uint256 accountId;
        string accountUsername;
        uint256 accountBalance;
        uint256 lastDeposit;
        uint256 lastTransfer;
    } 

    event Deposit(address indexed account, uint256 amount);
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Withdraw(address indexed account, uint256 amount);
    event CreateAccount(address indexed owner);

    modifier validAccount(address account) {
        require(accounts[account].accountId != 0, "invalid account");
        _;
    }

    mapping(address => Account) private accounts;
    uint256 private accountIds = 1;

    // create new account
    function createAccount(string calldata username) external {
        require(bytes(username).length > 0, "invalid username");
        require(accounts[msg.sender].accountId == 0, "existing account detected");
        accounts[msg.sender] = Account(accountIds, username, 0, 0, 0);
        accountIds++;
        emit CreateAccount(msg.sender);
    }

    // deposit into existing account
    function deposit() external payable validAccount(msg.sender) {
        Account storage account = accounts[msg.sender];
        uint256 amount = msg.value;
        account.accountBalance += amount;
        account.lastDeposit = block.timestamp;

        emit Deposit(msg.sender, amount);
    }

    // withdraw from existing account
    function withdraw(uint256 amount) external payable validAccount(msg.sender) {
        Account storage account = accounts[msg.sender];
        require(amount > 0, "invalid amount");
        require(account.accountBalance >= amount, "insufficient funds");
        // Debit amount from balance
        account.accountBalance -= amount;
        // Transfer to user wallet 
        (bool success,) = payable(msg.sender).call{value: amount}("");
        require(success, "transfer unsuccessful");

        emit Withdraw(msg.sender, amount);
    }

    // check balance
    function checkBalance(address account) public view validAccount(msg.sender) returns (uint256) {
         Account memory details = accounts[account];
         uint256 balance = details.accountBalance;
         return balance;
    }

    // transfer to another account
    function transfer(address to, uint256 amount) external payable validAccount(to) {
        require(amount > 0, "invalid amount");
        require(checkBalance(msg.sender) >= amount, "insufficient funds");

        // Get accounts involved in transaction
        Account storage senderAccount = accounts[msg.sender];
        Account storage receiverAccount = accounts[to];

        // Debit sender
        senderAccount.accountBalance -= amount;
        // Credit receiver
        receiverAccount.accountBalance += amount;

        senderAccount.lastTransfer = block.timestamp;
        receiverAccount.lastDeposit = block.timestamp;
        emit Transfer(msg.sender, to, amount);
    }

    // get account details
    function accountDetails() external view returns (
        uint256, 
        string memory, 
        uint256, 
        uint256,
        uint256
    ) {
        return (
            accounts[msg.sender].accountId,
            accounts[msg.sender].accountUsername,
            accounts[msg.sender].accountBalance,
            accounts[msg.sender].lastDeposit,
            accounts[msg.sender].lastTransfer
        );
    }
}
```
The solidity code above is the code for our smart contract. Let us explain every piece of the code below:

```sol
// SPDX-License-Identifier: MIT
```
The code above is usually the first line of code that is expected to be in our solidity file. It is called [the license specifier](https://docs.soliditylang.org/en/v0.6.8/layout-of-source-files.html). It is the one that tells the compiler what license is used in our solidity code. It starts with a double forward slash. Double forward slash means commenting in solidity, similar to how pound (#) commentdis Python. You can see that we use an licenses in our code. You can also use other licenses in your solidity code. Check out [SPDX](https://spdx.dev) to see other options of licenses that are available for your solidity cod.

```sol
pragma solidity ^0.8.0;
```
The second line in our solidity code file is the "version pragma of the file". It contains the version of solidity that we want to use for our smart contract code. `pragma` means that this file should be compiled with the specific solidity version. for our code, we didn't specify one version to compile our code, we specified a range of versions. Versions from 0.8.0 to less than version 0.9.0 will compile our solidity code. This range was made possible by the (^) sign in front of the number.

```sol
contract BankContract {
```
The line above is responsible for creating our contract. Solidity is similar to languages like Java and C++ that is why we are using `contract` as opposed to using `class` as in languages like Java or C++. Obviously, the name of our contract is `BankContract`.

```sol
  struct Account {
        uint256 accountId;
        string accountUsername;
        uint256 accountBalance;
        uint256 lastDeposit;
        uint256 lastTransfer;
    } 

    event Deposit(address indexed account, uint256 amount);
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Withdraw(address indexed account, uint256 amount);
    event CreateAccount(address indexed owner);

    modifier validAccount(address account) {
        require(accounts[account].accountId != 0, "invalid account");
        _;
    }

    mapping(address => Account) private accounts;
    uint256 private accountIds = 1;
```

Inside our contract, the first thing we did was to create a struct. A struct is a new and custom type in solidity that allows you to define your own data struc-
ture instead of using the existing ones like int and string. Our struct is called `Account`, which has all the properties of an acount. So each Accout will have all the necsesary info required. `Account` struct has:
- accountId
- accountUsername
- accountBalance
- lastDeposit
- lastwithdraw. These are all we need for our new Account.

The next are the events `Deposit`, `Transfer`, `Withdraw`, and `CreateAccount`, all which correspond to all the functions we will be creating later in this tutorial. They will be emitted when any of the respective events occur inside this contract.

Next line, we created a modifier and cal the modifier `validAccount`Modifier lets you create a wrapper around our function. These wrappers are executed before or after our function body, depending on the location. Inside the modifier, we are checking if the acount is a valid one, if not it fails to pass the require statement. `_` at the end of our modifier means that the remaining body of function shoule be pluged there.

lastly, we created a mapping to store all accounts created inside the contract. Each one maps the owner's address to the account details. All accounts created can be accessed by calling the `accounts` function. At the bottom, we created a `accountIds` variable to keep track of each acount ids. We started counting from 1 because we want to make track of invalid accounts - accounts with an id of 0.

```sol
   // create new account
    function createAccount(string calldata username) external {
        require(bytes(username).length > 0, "invalid username");
        require(accounts[msg.sender].accountId == 0, "existing account detected");
        accounts[msg.sender] = Account(accountIds, username, 0, 0, 0);
        accountIds++;
        emit CreateAccount(msg.sender);
    }
```

- The first function in the contract is `createAccount` function. It is the one that get called when new users want to create an account. The username entered is first checked to be valid. The account is then checked to make sure it does not exist in our contract. The account is added to the contract, accountId's increased and event is then emitted.

```sol
    // deposit into existing account
    function deposit() external payable validAccount(msg.sender) {
        Account storage account = accounts[msg.sender];
        uint256 amount = msg.value;
        account.accountBalance += amount;
        account.lastDeposit = block.timestamp;

        emit Deposit(msg.sender, amount);
    }
```
- The second function is the `deposit` function. It is called when you want to make deposit into your bank account. The function uses the `validAccount()` modifier to check if the account is valid before it continues. Lastly, it adds the amount sent by user to their account balance and update the `lastDeposit` variable, then emit an event.

```sol
   // withdraw from existing account
    function withdraw(uint256 amount) external payable validAccount(msg.sender) {
        Account storage account = accounts[msg.sender];
        require(amount > 0, "invalid amount");
        require(account.accountBalance >= amount, "insufficient funds");
        // Debit amount from balance
        account.accountBalance -= amount;
        // Transfer to user wallet 
        (bool success,) = payable(msg.sender).call{value: amount}("");
        require(success, "transfer unsuccessful");

        emit Withdraw(msg.sender, amount);
    }
```

- The third function is `withdraw` which withdraws some or all of the amount in your bank account. It also has the valid account modifier. It carries out some checks before sending the amount to your wallet address from your bank account. Finally it emits the Withdraw event.

```sol
  // check balance
    function checkBalance(address account) public view validAccount(msg.sender) returns (uint256) {
         Account memory details = accounts[account];
         uint256 balance = details.accountBalance;
         return balance;
    }
```
- Fourth function - `checkBalance` returns the account balance of whoever calls it.

```sol
  // transfer to another account
    function transfer(address to, uint256 amount) external payable validAccount(to) {
        require(amount > 0, "invalid amount");
        require(checkBalance(msg.sender) >= amount, "insufficient funds");

        // Get accounts involved in transaction
        Account storage senderAccount = accounts[msg.sender];
        Account storage receiverAccount = accounts[to];

        // Debit sender
        senderAccount.accountBalance -= amount;
        // Credit receiver
        receiverAccount.accountBalance += amount;

        senderAccount.lastTransfer = block.timestamp;
        receiverAccount.lastDeposit = block.timestamp;
        emit Transfer(msg.sender, to, amount);
    }
```

- `transfer` function makes a transfer from your account to another registered user. Essentially, what it does is to subtract that amount from your account balance and add it to the receiver's balance. And then sets the lastDeposit and lastTransfer properties of both accounts before emitting an event.

```sol
   // get account details
    function accountDetails() external view returns (
        uint256, 
        string memory, 
        uint256, 
        uint256,
        uint256
    ) {
        return (
            accounts[msg.sender].accountId,
            accounts[msg.sender].accountUsername,
            accounts[msg.sender].accountBalance,
            accounts[msg.sender].lastDeposit,
            accounts[msg.sender].lastTransfer
        );
    }
```

- Getter function _accountDetails_ with return all the account details associated to the account that called it.

### Deploy smart contract :arrow_up: 

We want to deploy the solidity smart contract to Celo blockchain from Remix IDE UI. 
- Search for the **Celo plugin** from remix and install it by clicking the install button. Here is a [documentation](https://docs.celo.org/developer/deploy/remix) for better explanation.
- Connect your celo extension to the wallet and make sure you have some CELO to pay for gas fees while deploying the contract. follopw this [link](https://faucet.celo.org/) to fund your Celo account.
- Compile the contract.
- Deploy the contract.
- Copy the address generated into your clipboard for use later.

## UI Section :desktop_computer: 
- UI - User Interface
- We will build the user interface we interacted with earlier in this section.
- We will use React and React-celo package for the UI.
- Clone the project repository by running the command in your terminal:

```bash
git clone https://github.com/bank-app-web3
```

- After cloning it, open it in your VSCode editor or any of your choice
- Then run the command 
```bash
npm install
```
- The command we specified above will install all the dependencies inside the package.json file.
- One of those package to highlight is the `@celo/react-celo` and `@celo/contractkit` package

![packages-highlight](https://i.imgur.com/Qfxn6q2.png)

These two packages enable our smart contract and React Dapp to interact.

- If you open index.js file, it should be similar to the image below.

![index.js](https://i.imgur.com/KBgyRP6.png)

We are wrapping our whole App component with the `CeloProvider` component. This is what makes the wallet pop up earlier asking you to select from multiple wallets.

- Open the App.js and you will notice a lot of things happening, Let me give us a summary of what is happening inside the file
    - We import a bunch of packages
    - We created a component, name it App.
    - We created the App state using `useState` provided by React
    - We then created the functions - `connectWallet`,`getContract`, `getAccount`, `signUp`, `deposit`, `withdraw`, and `transfer`.
    
        - `connectWallet` - Creates a connection between our Dapp and got balance and wallet address as well. Take note of the `connect` function we utilised in this function.
        - `getContract` - Connects to the deployed contract using the contract [ABI(Application Binary Interfaces)](https://www.alchemy.com/overviews/what-is-an-abi-of-a-smart-contract-examples-and-usage) and deployment address.
        - `getAccount` - Fetches registered account from the smart contract.
        - `signUp` - Signs up our details to the smart contract so we can use if effectively.
        - `deposit` - Makes a deposit to our bank account in the smart contract.
        - `withdraw` - Withdraws some or all of our amount from our bank account.
        - `transfer` - Transfer funds from your account to another registered user.

After creating the functions, the HTML of the dapp is designed, which contains what we wiil see when we visit the Dapp.  All the functions created were used inside the HTML, and some styling was applied using CSS.

Check the App.css file to see the styling applied to the HTML of the page.

Lastly, if you open the `contracts` folder, you will find two files which are `Bank.sol` and `bank.abi.json`. Bank.sol contains all the code for our smart contract. bank.abi.json contains the ABI of the smart contract in Bank.sol file.

You can now start your local server by running this command.

```bash
npm start
```
Then enter the localhost url in your browser to see the page. 

This is what the page will look like

![page-view-on-first-load-with-url-bar-included](https://i.imgur.com/92cEHmq.png)

## Conclusion :end: 
I hope you have learned how to use react-celo to connect your React application to the Celo blockchain.  I also hope you have leant how to write solidity smart contracts and deploy them to the Celo blockchain using Remix. This is just the beginning of this journey as we might build something larger using newer and shiny technologies. 

Try out the version of the project I hosted on [Github Pages](https://Evangelistsb.github.io/bank-app-web3)

Check out the project's source code here on [Github](https://github.com/Evangelistsb/bank-app-web3)

Drop a comment for what you think we should build next time :point_down: 


## FAQ :question: 

- **_What is Celo Blockchain?_** Celo is the carbon-negative, mobile-first, EVM-compatible blockchain ecosystem leading a thriving new digital economy for all
- **_What is Remix IDE?_** Remix, more commonly known as Remix IDE, is an open-source Ethereum IDE you can use to write, compile and debug Solidity code.
- _**What is a Smart Contract?**_ Smart contracts are just simple computer codes that are run on the blockchain, and they are self-executable when we meet certain conditions. They are immutable, transparent, and, most importantly, they are decentralized. Clearly, this gives them an edge over traditional contracts.
