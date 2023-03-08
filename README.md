# Build a Bank Dapp With React and Celo

## Table of Content
- [Build a Bank Dapp With React and Celo](#build-a-bank-dapp-with-react-and-celo)
  - [Table of Content](#table-of-content)
  - [Introduction :book:](#introduction-book)
  - [Learning Objectives :eyeglasses:](#learning-objectives-eyeglasses)
  - [Tech Stack :computer:](#tech-stack-computer)
  - [Prerequisites :closed\_book:](#prerequisites-closed_book)
  - [A Glance of the App :eyes:](#a-glance-of-the-app-eyes)
    - [Deposit](#deposit)
    - [Withdraw](#withdraw)
    - [Transfer](#transfer)
  - [Smart Contract Section :hammer\_and\_wrench:](#smart-contract-section-hammer_and_wrench)
    - [Coding the smart contract :writing\_hand:](#coding-the-smart-contract-writing_hand)
    - [Deploy Smart Contract :arrow\_up:](#deploy-smart-contract-arrow_up)
  - [UI Section :desktop\_computer:](#ui-section-desktop_computer)
  - [Conclusion :end:](#conclusion-end)
  - [FAQ :question:](#faq-question)

## Introduction :book:
If you were previously a Web2 developer and recently got interested in the Web3 world either because of the noise on Twitter or the never-ending meetups held by Web3 folks, or you are already a Web3 hacker, whichever categories you call into, this tutorial is for you. We want to build an amazing project using `react-celo`, a package provided by the great Celo developers team, write the smart contract with Solidity, then deploy it to the Celo blockchain. Grab a cup of coffee because this is going to be a long ride.


## Learning Objectives :eyeglasses: 
By the end of this tutorial, you will have learned the following:
- How to write smart contracts with Solidity
- How to build an amazing UI using React
- How to connect a React UI with the Celo blockchain using **react-celo**
- How to deploy smart contracts to Celo using Remix

## Tech Stack :computer: 
The following tools are used for this tutorial:
- [React](https://reactjs.org/)
- [React-Celo](https://github.com/celo-org/react-celo)
- [Remix](https://remix.ethereum.org/) 
- [Node.js](https://nodejs.org/en/)
- [Npm](https://www.npmjs.com/)
- [VS Code](https://code.visualstudio.com/)
- [HTML](https://html.com/) and [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [Solidity](https://soliditylang.org/)

## Prerequisites :closed_book: 
Before starting this tutorial, make sure you have a basic understanding of the following:
- Solidity smart contract basics
- JavaScript basics
- command line usage

## A Glance of the App :eyes: 
Before we continue let us take a glance at the dapp we want to build for this tutorial. It is a bank application that simulates the operations of banks in the real world. It allows users of the app to do the following; create an account with the bank using their username alone, deposit money into the bank, withdraw money deposited into the bank, and transfer money from one account to another. In addition, logged-in users can see the last time a deposit was made into their account, and they can also see the last time a `withdrawal` was made from their account.

Let us interact with the dapp and see how it feels before building it.

This is the landing page of the dapp

![1-landing-page](https://i.imgur.com/IBhkG4J.png)


The first thing to do after visiting the landing page is to click on the **Connect** button to connect your wallet to the dapp. We used the `react-celo` package to access our wallet and connect to the Celo blockchain. After clicking on the button, a pretty looking modal designed by the Celo team will appear to us asking us to choose from a variety of wallet. 

![2-react-celo-wallet-interface](https://i.imgur.com/IMksEGO.png)

Select the Celo Extension Wallet and the wallet will pop up with a new window asking you to connect to the dapp.

After connecting the Celo Extension Wallet, your dashboard will change to something similar to this

![3-sign-up-page](https://i.imgur.com/EsP9Fmz.png)


Enter your username and click on the sign-up button. It will pop up the Celo Extension Wallet asking you to confirm the transaction. Click on **confirm** and the transaction will go through. After some seconds, it will complete. Refresh the page to see the changes (You might have to connect your wallet again).

![3b-dashboard](https://i.imgur.com/WBRynA3.png)


Let us now perform some actions with the dapp

### Deposit

Let us try to deposit into the dapp by entering the `amount` and clicking on the large green 'deposit' button on the right-hand side of the page. After entering the *deposit amount* and clicking the button, wait for some seconds for the transaction to complete, and then refresh the page. The balance you just deposited will reflect in your dashboard like this:

![4-dashboard-update-after-deposit](https://i.imgur.com/xRkltWN.png)


### Withdraw

Initially, we deposited three CELO into our bank account, and it has been reflected on our dashboard. Now let's withdraw one out of the three CELO we sent. 
Enter the amount to be withdrawn into the field and click on the 'withdraw' button. Wait for the transaction to complete and refresh the page to see the changes.

![5-balance-update-after-withdraw](https://i.imgur.com/RaOW4jl.png)


>**_Note:_** Notice that our balance from the bank account has dropped and our Celo Extension wallet balance has increased

### Transfer
This is the last test we will carry out in our dapp. We will transfer from the two CELO we have to another account. Before you continue, create another account in your Celo Extension Wallet and switch to the account. Use the account to signup up to the dapp and then copy the account address. Switch back your account to the first one and try to make a transfer by entering the address of the second account you just created followed by the amount. Click on the transfer button and wait for some seconds for the transaction to succeed. After completion, here is our dashboard.

![6-dashboard-after-transfer](https://i.imgur.com/LSk99uN.png)


This is the dashboard of our second account.

![7-dashboard-of-second-account-after-transfer](https://i.imgur.com/5bI60IQ.png)


From what you have seen soo far, I bet you can't wait to build this dapp on your own.


## Smart Contract Section :hammer_and_wrench: 
This is the section where we will write the smart contract for the dapp you just tested above. We will write the smart contract using Solidity on the Remix IDE. Inside this same section, we will also deploy the contract to the blockchain using the UI provided by Remix for us.

### Coding the smart contract :writing_hand: 
Head on to https://remix.ethereum.org to open Remix right from your browser. You can use either Brave, Chrome, Firefox, etc whichever one you are more comfortable with.

Create a new file with the name `Bank.sol` and open the file to start writing your Solidity code inside it.

We will first show you the finished code before explaining every piece of it. 

Paste the code below into the file:

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
The Solidity code above is the code for our smart contract. Let us now explain every piece of the code below:

```sol
// SPDX-License-Identifier: MIT
```
The code above is usually the first line of code that is expected to be in our Solidity file. It is called "the license specifier". It is the one that tells the compiler that we want this or that license in our Solidity code. It usually starts with a double-forward slash which is used in Solidity for single-line comments just like the pound (#) is used in Python. You can see that we use the  **MIT** license in our code. You can also use other licenses in your Solidity code. Check out https://spdx.dev to see other options of licenses that are available for your Solidity code.

```sol
pragma solidity ^0.8.0;
```
The second line is our Solidity code file is the "version pragma of the file". It contains the version of Solidity that we want to use for our smart contract code. `pragma` means that this file should be compiled with the specific solidity version. For our code, we didn't specify one version to compile our code, we specified a range of versions from 0.8.0 but less than version 0.9.0 that the compiler can use for our code. This range was made possible by the (^) sign in front of the number.

```sol
contract BankContract {}
```
The line above is responsible for creating our contract. Solidity is similar to languages like Java and C++ which is why we are using `contract` as opposed to using `class` as in languages like Java or C++. The name of our contract is `BankContract`.

Next, add the following code to the `BankContract` body:

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

Inside our contract, the first thing we did was to create a struct. A **struct** is a new and custom type in Solidity that allows you to define custom data types instead of using the existing ones like `int` and `string`. Our struct is called `Account`, which has all the properties of an account. So each `Account` will have all the necessary info required. `Account` struct has `accountId`, `accountUsername`, `accountBalance`, `lastDeposit`, and `lastwithdraw` which are all we need for our brand new Account.

Next is the events `Deposit`, `Transfer`, `Withdraw`, and `CreateAccount`, all of which correspond to all the functions we will be creating later in this tutorial. They will be emitted when any of the respective events occur inside this contract.

Next line, we created a modifier and called the modifier `validAccount`Modifier lets you create a wrapper around our function. These wrappers are executed before or after our function body, depending on the location. Inside the modifier, we are checking if the account is a valid one, if not it fails to pass the *require* statement. `_` at the end of our modifier means that the remaining body of the function should be plugged there.

Lastly, we created a mapping to store all accounts created inside the contract. The `accounts` mapping maps an address to an `Account` which contains the account details. All created accounts can be accessed by calling the `accounts` function. At the bottom, we created a variable to keep track of the account IDs. We started counting from 1 as invalid accounts will have an ID of `0`.

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

- The first function in the contract is the _createAccount_ function. It is the one that gets called when new users into the dapp want to create an account. The username entered is first checked to be valid. The account is then checked to make sure we are not overwriting an existing account. The account is added to the contract, `accountIds` is incremented by one and the `CreateAccount` event is then emitted.

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
- The second function is the _deposit_ function. It is the one that gets called when you want to deposit into your bank account. The function uses the `validAccount()` modifier to first check if the account is valid before continuing. Lastly, it adds the amount sent by the user to their account balance and updates the `lastDeposit` variable, then emits the `Deposit` event.

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

- The third function is the _withdraw_ which withdraws some or all of the amount in your bank account. It also has the `validAccount()` modifier. It carries out some checks before sending the amount to your wallet address from your bank account. Finally, it emits the `Withdraw` event.

    ```sol
        // check balance
        function checkBalance(address account) public view validAccount(msg.sender) returns (uint256) {
            Account memory details = accounts[account];
            uint256 balance = details.accountBalance;
            return balance;
        }
    ```
- Fourth function - _checkBalance_, will return the account balance of whoever called it.

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

- The _transfer_ function makes a transfer from your account to another registered user. Essentially, what it does is subtract that amount from your account balance and add it to the receiver's balance, and then sets the `lastDeposit` and `lastTransfer` properties of both accounts before emitting the `Transfer` event.

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

- The function _accountDetails_ is a getter function that will return all the account details associated with the sender of the transaction.

### Deploy Smart Contract :arrow_up: 

We want to deploy the Solidity smart contract to the Celo blockchain from the Remix IDE UI. 
- Search for the "Celo" plugin from Remix and install it by clicking on the **activate** button
- Connect the Celo plugin with your wallet and make sure you have some CELO to pay for gas fees while deploying the contract
- Compile the contract
- Deploy the contract
- Copy the address generated into your clipboard for use later

## UI Section :desktop_computer: 
- UI - User Interface
- We will build the user interface we interacted with earlier in this section.
- We will use React and the `react-celo` package for the UI.
- Clone the project repository by running this command in your terminal:

```bash
git clone https://github.com/bank-app-web3
```

- After cloning it, open it in your VS Code editor or any editor of your choice
- Then run the command 
```bash
npm install
```
- The command we specified above will install all the dependencies we added inside of the `package.json` file.
- One of those package to highlight is the `@celo/react-celo` and the `@celo/contractkit` package

![packages-highlight](https://i.imgur.com/Qfxn6q2.png)

These two packages are what make us able to interact with the Celo blockchain from the React dapp.

- If you open the `index.js` file, you should see this:

![index.js](https://i.imgur.com/KBgyRP6.png)

We are wrapping our whole `App` component with the `CeloProvider` component. This is what makes the wallet that popped up earlier asking you to select from multiple wallets.

- Open the `App.js` file and you will notice a lot of things happening, Let me give us a summary of what is happening inside the file
    - We import a bunch of packages
    - We create a component, name it `App`
    - We created the App state using `useState` provided by React
    - We then created the functions - `connectWallet`, `getContract`, `getAccount`, `signUp`, `deposite`, `withdraw`, and `transfer`.
        - `connectWallet` - Creates a connection between our dapp and gets out the balance and wallet address as well. Take note of the `connect` function we utilized in this function.
        - `getContract` - Connects to the deployed contract using the contract ABI and deployment address
        - `getAccount` - Fetch our registered account from the smart contract
        - `signUP` - Signs up our details to the smart contract so we can use it effectively
        - `deposit` - Deposits to our bank account in the smart contract
        - `withdraw` - Withdraws some or all of our amount from our bank account
        - `transfer` - Transfer funds from your account to another registered user

After creating the functions, the HTML of the dapp is designed, which will contain what we will see when we visit the dapp. All the functions created were used inside the HTML and also some styling was applied using CSS.

Check the App.css file to see the styling applied to the HTML of the page.

Lastly, if you open the `contracts` folder, you will find two files which are `Bank.sol` and `bank.abi.json`. The `Bank.sol` file contains all the code for our smart contract. The `bank.abi.json` contains the ABI of our smart contract.

After installing the packages from the command we ran earlier, you can now start your local server by running this command

```bash
npm start
```
and then enter the localhost URL in your browser to see the page. 

This is what the page will look like

![page-view-on-first-load-with-url-bar-included](https://i.imgur.com/92cEHmq.png)

## Conclusion :end: 
I hope that you have learned how to use the `react-celo` package to connect your React application to the Celo blockchain.  I also hope you have learned how to write Solidity smart contracts and deploy them to the Celo blockchain using Remix. This is just the beginning of this journey as we might build something larger using newer and shiny technologies. 

Try out the version of the project I hosted on [Github Pages](https://Evangelistsb.github.io/bank-app-web3)

Check out the project's source code here on [Github](https://github.com/Evangelistsb/bank-app-web3)

Drop a comment for what you think we should build next time :point_down: 


## FAQ :question: 

- **_What is Celo Blockchain?_** Celo is the carbon-negative, mobile-first, EVM-compatible blockchain ecosystem leading a thriving new digital economy for all
- **_What is Remix IDE?_** Remix, more commonly known as Remix IDE, is an open-source Ethereum IDE you can use to write, compile and debug Solidity code.
- _**What is a Smart Contract?**_ Smart contracts are just simple computer codes that are run on the blockchain, and they are self-executable when we meet certain conditions. They are immutable, transparent, and, most importantly, they are decentralized. This gives them an edge over traditional contracts.
