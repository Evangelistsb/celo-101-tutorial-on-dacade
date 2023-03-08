# Build a Bank Dapp with React-Celo 

# Introduction :book:
If you were previously a Web2 developer and recently got interested in the Web3 world either because of the noise on Twitter or the never-ending meetups held by Web3 folks, or you are already a Web3 hacker, whichever categories you call into, this tutorial is for you. We want to build an amazing project using react-celo, a package provided by the great celo developers team, write the smart contract with solidity, then deploy it to the Celo blockchain. Grab a cup of coffee because these will going to be a long ride.


## Learning objectives :eyeglasses: 
By the end of this tutorial, you will have learned the following
- How to write smart contract with Solidity
- How to build an amazing UI using react
- How to connect react ui with the celo blockchain using react-celo
- How to deploy solidity codes to Celo using Remix

## Tech Stack used in this tutorial :computer: 
These following tools were used for this tutorial
- React
- React-Celo
- Remix 
- NodeJs
- Npm
- VSCode
- HTML and CSS
- Solidity

## Prerequisites :closed_book: 
Before starging this tutorial, make sure you are equiped with the following:
- solidity smart contract basics
- javascript basics
- command line usage

## A glance of the app :eyes: 
Before we continue let us take a glance of the application we wan tto build for this tutorial. It is a bank application that simulete the operations of banks in the real world. It allows users of the app to do the following; create an account with the bank using username alone, deposit money into the bank, withdraw money deposited into the bank, transfer money from one account to another. in addition, logged in users can see last time a deposit was made into their account, they can also see last time a withdraw was made from their account.

Let us interact with the app and see how it feels before building it.

This is the landing page of the app

![1-landing-page](https://i.imgur.com/IBhkG4J.png)


The first thing to do after visiting the langing page is to click on the Connect button to connect your wallet to the app. We used React-Celo package to access our walet and connect to the Celo blockchain. After clicking on the button, a pretty looking modal designed by the celo team will appear to us asking to choose from a varity of wallet. 

![2-react-celo-wallet-interface](https://i.imgur.com/IMksEGO.png)

Select Celo Extension Wallet and the wallet will pop up with a new window asking you to connect to the dapp.

After connecting Celo exteisn wallet, your dashboard will change to something similar to this

![3-sign-up-page](https://i.imgur.com/EsP9Fmz.png)


Enter your username and click on the sign up button. It will pop up the Celo extensionwallet asking you to confirm the transaction. Click on confirm and the trasaction will go through. AFter some seconds, it will complete. Refresh the page to see the changes (You might have to connect your wallet agaoin).

![3b-dashboard](https://i.imgur.com/WBRynA3.png)


Let us now perform some actins with the app

- **Deposit**

Let us try to deposit into the app by entering the amount and clicking on the large green 'deposit' button by the right hand side of the page. After entring the deposit amount and clicking the botton, wait for some seconds for the transaction to complete, and then refresh the page. The balance you just deposited will reflext in your dashboard like this:

![4-dashboard-update-after-deposit](https://i.imgur.com/xRkltWN.png)


- **Withdraw**

Initially, we made a deposit of 3 CELO into our bank acount, and it has reflected on our dashboard. Now let's withdraw 1 out of the 3 we sent. 
Enter the amount to be withdrawed into the field and click on the 'withdraw' button. Wait for transaction to complet and refresh page to see changes.

![5-balance-update-after-withdraw](https://i.imgur.com/RaOW4jl.png)


*Notice that our balance from the bank account has dropped and our Celo Extension wallet balance has increased

- **Transfer**
This is the last test we will carry out in out dapp. We will transfer from the 2 CELO we have to another account. Before you continue, creaete another account in you celo extesion wallet and switch to the account. Use the account to sign up on the dapp and then copy the account account address. Switch back you account to the first one and try to make a transfer by entering the address of the second account you just created followed by the amouny. Click on the transfer button and wait for some seconds for the transactio to succeed. After completion, here is our dashboard.

![6-dashboard-after-transfer](https://i.imgur.com/LSk99uN.png)


This is the dashboard of our second account.

![7-dashboard-of-second-account-after-transfer](https://i.imgur.com/5bI60IQ.png)


From what you have see soo far, I bet you can't wait to buiild this dapp on you own.


# Smart Contract Section :hammer_and_wrench: 
This is the section where we will write the smart contract for the app you just tested above. We will write the smart usng Solidity on Remix IDE. Inside this same sectin, we will also deploy the contract to the blockchain using the ui probided by remix for us.

## Coding the smart contract :writing_hand: 
Head on to https://remix.ethereum.org to open Remix right from your browser. You can use either brave, chrome, firefox, etc whichever one that you are move convenient with.

Create a new file with the name `Bank.sol` and open the file to start writng your solidity code inside it.

We will first show you the finished code before explaining every piece of it. 

paste the code below into your remix ide editor.

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
The solidity code above is the code for our smat contract. Let us now explain every pieces of the code below:

```bash
// SPDX-License-Identifier: MIT
```
THE code above is usually the first line fo code that is expected to be in our solidity file. It is called "the license specifier". It is the one that tells the compiler that we want this or that license in our solidity code. It. usually start with a double forward slash. Double forward slash means commenting in solidity, similar to how pound (#) is commenting is Python. You can see that we use MIT license in our code. You can also use other license in your solidity code. Check out https://spdx.dev to see other options of licenses that are available for your solidity cod.

```sol
pragma solidity ^0.8.0;
```
The second line is our solidity code file is the "version pragma of the file". It contians the version of solidity that we want to use for our smart contract code. `pragma` means that this file should be compile with the specific solidity version. for our code, we did't specify one version to compile our code, we specified a range of versions. Versions from 0.8.0 to but not included 0.9.0 will compile our solidity code. This ranage was made possible by the (^) sign in front of the number.

```sol
contract BankContract {
```
The line above is responsible for creating our contract. Solidity is similar to languages like Java and C++ that is why we are using `contract` as oppsed to using `class` as in languages like Java or C++. Obviously, the name of our contract is `BankContract`.

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

Inside our contract, the first thing we dis was to create a struct. A struct is a new and custom type in solidity that allows you to define your own data struture instead of using the existing ones like int and string. Our struct is called `Account`, which has all the properties of an acount. So each Accout will have all the necsesary info required. `Account` sstrut has accountId, accountUsername, accountBalance, lastDeposit, lastwithdraw which are all we need for our brand new Account.

The next are the events `Deposit`, `Transfer`, `Withdraw`, and `CreateAccount`, all which coresponds to all the functions we will be creating later in this tutorial. They will be emitted when any of the respective event occurs inside this contract.

Next line, we created a modifier and cal the modifier `validAccount`Modifier let's you create a wrapper around our function. This wrappers are executed before or after our function body, depending on the location. Inside the modifier we are checkingif the acount is a valid one, if not it fails to pass the require statement. `_` at the end of our modifier means that the remaining body of function shoule be plugked there.

lastly,we created a mapping to store all accounts created inside the contract. EAch one maps the owner address to the account details. All created accouns can be accessed by calling the `accounts` function. At the botton, we created a variable to keep track of ht eacount ids. We started counting from 1 because we want to make track of invalid accounts - accounts with id of 0.

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

- first function in the contract is _createAccount_ function. It is the one that get called when new users into the dapp want to create account. The username enter is first check to be valid. The account is then checked to make sure it si not duplicate. The account is add to the contract, accountId's increased and event is then emitted.

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
- The second function is _deposit_ function. It is the one that get's call when you want to make deposit into your bank account. The function used the validAccount() modifier to first check if the account is valid before continueing. Lastly, it adds the amount sent by user their accoubnt balance and update the lastDeposit variable, then emit an event.

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

- Third function is _withdraw_ which withdraws some or all of amount in your bank account. It also has the valid account modifier. It carries our some checks before sending the amount to your wallet address from you bank account. Finally it emit the Withdraw event.

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

- _transfer_ function makes a transfer from your account to another registered user. Essentially, what is does it to subtract that amount from your account balance and add it to the receivers balance. And then sets the lastDeposit and lastTransfer property of the both accounts before emitting an event.

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

- Getter function _accountDetails_ with return all the account details associated to the accout that called it.

## Deploy smart contract :arrow_up: 

We want to deploy the solidity smart contract to Celo blockchain from Remix IDE UI. 
- Search for the "Celo" extension from remix and install it by clicking of install
- Connect your celo extension to the wallet and make sure you have some celo to pay for gas fees while deploying the contract
- Compile the contract
- Deploy the contract
- Copy the address generated into your clipboard for use later

# UI Section :desktop_computer: 
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
- The command we specified above will run the install all the dependencies in we added inside of the package.json file.
- One of those package to highlight is the `@celo/react-celo` and `@celo/contractkit` package

![packages-highlight](https://i.imgur.com/Qfxn6q2.png)

These two package is what makes us to be able to interact with Celo blockchain from the React dapp.

- If you open index.js file, you will notice the unusall

![index.js](https://i.imgur.com/KBgyRP6.png)

We are wrapping our whole App component with the `CeloProvider` component. This is what makes the wallet the popped up earlier to pop asking you to select from multiple wallets.

- Open the App.js and you will notice a lot of things happening, Let me give us a summary of what is happening inside the file
    - We import a bunch of packages
    - We create a component, name it App
    - We created the App state using `useState` provided by React
    - We then created the functions - `connectWallet`,`getContract`, `getAccount`, `signUp`, `deposite`, `withdraw`, `transfer`.
        - `connectWallet` - Creates a connection between our dapp and get out balance and wallet address as well. Take note of the `connect` function we utelised in this function.
        - `getContract` - Connects to the deployed contract using the contract Abi and deployment address
        - `getAccount` - Fetch our registered account from the smart contract
        - `signUP` - Signs up our details to the smart contract so we can use if effectively
        - `deposit` - Makes a deposit to our bank account in the smart contract
        - `withdraw` - Withdraws some or all of our amount from our bank account
        - `transfer` - Transfer funds from your account to another registered user

After creating the functions,the HTML of the dapp is designed, which will contains what we wiil see when we visit dapp. All the functions created were used inside the HTML and and also some styling were applied using CSS.

Check the App.css file to see the styling applied to the HTML of the page.

Lastly, if you open the `contracts` folder, you will find two files which are `Bank.sol` and `bank.abi.json`. Bank.sol contains all the code for our smart contract. bank.abi.json contains the ABI of the smart contract in Bank.sol file.

After installing the packages from the command we ran earlier, you can now start your local server by running this connand

```bash
npm start
```
and then enter the localhost url in your browser to see the page. 

This is what the page will look like

![page-view-on-first-load-with-url-bar-included](https://i.imgur.com/92cEHmq.png)

# Conclusion :end: 
I hope that you have learnt how to use React-Celo to connect your React application to the celo blockchain.  I also hope you have leant how to write solidity smart contracts and deploy them to the Celo blockchain using Remix. This is just the beginning of this journey as we might build something larger using newer and shiny technologies. 

Try out the version of the project I hosted on [Github Pages](https://Evangelistsb.github.io/bank-app-web3)

Check out the project's source code here on [Github](https://github.com/Evangelistsb/bank-app-web3)

Drop a comment for what you think we should build next time :point_down: 


# FAQ :question: 

- **_What is Celo Blockchain?_** Celo is the carbon-negative, mobile-first, EVM-compatible blockchain ecosystem leading a thriving new digital economy for all
- **_What is Remix IDE?_** Remix, more commonly known as Remix IDE, is an open-source Ethereum IDE you can use to write, compile and debug Solidity code.
- _**What is a Smart Contract?**_ Smart contracts are just simple computer codes that are run on the blockchain, and they are self-executable when we meet certain conditions. They are immutable, transparent, and, most importantly, they are decentralized. Clearly, this gives them an edge over traditional contracts.
