# Building a Decentralized Lottery on CELO Blockchain
## Building a Decentralized Lottery on the CELO Blockchain: From Smart Contract Development to Deployment

The rise of blockchain technology has paved the way for innovative applications in various fields, including finance and entertainment. Among these are decentralized applications (DApps), offering transparent, secure, and trustless alternatives to traditional centralized systems. 

This article delves into the fascinating world of DApps by guiding you through the development and deployment of a decentralized lottery smart contract on the CELO blockchain.

## Table of Contents

- [Section 1: Understanding the Basics](#understanding-the-basics)
  - [1.1 Overview of Lottery Smart Contracts](#11-overview-of-lottery-smart-contracts)
- [Section 2: Code Explanation of the Smart Contract](#section2-code-explanation-of-the-smart-contracts)
  - [2.1 Key Variables](#21-key-variables)
  - [2.2 Event](#22-event)
  - [2.3 Modifiers](#23-modifiers)
  - [2.4 Constructor](#24-constructor)
  - [2.5 Managing Rounds and Selecting Winners](#25-managing-rounds-and-selecting-winners)
    - [2.5.1 Entering a Round](#251-entering-a-round)
    - [2.5.2 Starting a New Round](#252-starting-a-new-round)
    - [2.5.3 Ending the Current Round](#253-ending-the-current-round)
  - [2.6 Checking Round Status and Participants](#26-checking-round-status-and-participants)
    - [2.6.1 Getting Current Round Status](#261-getting-current-round-status)
    - [2.6.2 Generating a Random Number](#262-generating-a-random-number)
    - [2.6.3 Retrieving Players](#263-retrieving-players)
- [Section 3: Complete Code](#complete-code)
- [Section 4: Getting Celo faucet](#getting-celo-faucet)
- [Section 5: Deployment to remix](#deployment-to-remix)
- [Section 6: Conclusion](#section-6-conclusion)
- [Section 7: Next Steps](#section-7-next-steps)


### Section 1: Understanding the Basics

### 1.1 Overview of Lottery Smart Contracts
Traditional lotteries often raise concerns about transparency, fairness, and trust in centralized authorities. In contrast, decentralized lotteries, powered by smart contracts on blockchains like CELO, offer a compelling solution. These contracts, self-executing agreements with codified terms, eliminate the need for intermediaries, ensuring verifiable randomness, tamper-proof record-keeping, and immutable transaction history. This fosters trust and confidence in the lottery system, creating a more engaging and secure experience for participants.

Decentralized lottery smart contracts bring transparency and fairness to the process of organizing and participating in lotteries. This guide explores the fundamental concepts and components involved in building such a contract.

## Section 2: Code Explanation of the Smart Contract

### 2.1 Key Variables
```solidity
    address public manager;
    address[] public players;
    address public lastWinner;
    uint256 public roundEndTime;
    uint256 public minimumPlayers;
    bool public isRoundActive;
```
The journey begins with understanding the core components of the smart contract. Key variables manage essential aspects like the manager (contract creator), players, winners, round duration, minimum participation thresholds, and round status. Events act as notifications, signaling winner selection and prize distribution. Modifiers enforce access control and restrict certain functions to authorized users. The constructor initializes the contract, setting the manager, minimum players, and initial round status.

These variables represent essential components of the lottery smart contract. The manager initiates the lottery, players contribute funds to participate, lastWinner keeps track of the previous winner, roundEndTime determines when a round ends, minimumPlayers sets the threshold for starting a new round, and isRoundActive indicates whether a round is currently active.

### 2.2 Event
```solidity
event LotteryWinner(address winner, uint256 amount);
```
When this event is emitted within the smart contract, it records the winner's address and the corresponding prize amount. This logged information can be observed by external applications or user interfaces, enabling them to track and display details about lottery winners and prize amounts on the blockchain.

### 2.3 Modifiers
```solidity
 modifier restricted() {
        require(msg.sender == manager, "Only the manager can call this function");
        _;
    }

    modifier roundActive() {
        require(isRoundActive, "The current round is not active");
        _;
    }
```
The `restricted` modifier ensures that only the manager can call certain functions, providing access control. The `roundActive` modifier checks whether the current round is active before allowing certain operations.

### 2.4 Constructor
```solidity
   constructor(uint256 _minimumPlayers) {
        manager = msg.sender;
        minimumPlayers = _minimumPlayers;
        isRoundActive = false;
    }
```
The constructor function initializes the decentralized lottery smart contract by setting the contract manager to the deployer's address (msg.sender), defining the minimum required players (_minimumPlayers), and initially marking the round as inactive (isRoundActive = false).

### 2.5 Managing Rounds and Selecting Winners
The heart of the lottery lies in managing rounds and selecting winners. Players can contribute funds to enter a round, adhering to minimum contribution requirements and ensuring entries during active rounds are prohibited. The manager initiates new rounds, ensuring sufficient player participation before setting the round duration and marking it active. Round closure involves randomly selecting a winner (proportional to their contribution) and transferring the entire contract balance as the prize. Functions are further provided to retrieve player lists, check round status, and access end times

  ### 2.5.1 Entering a Round
  ```solidity
    function enter() public payable {
        require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether");
        require(!isRoundActive, "Cannot enter while a round is active");
        
        players.push(msg.sender);
    }
  ```
The enter function allows players to participate in the lottery by contributing funds. It imposes a minimum contribution requirement and ensures that entries are not allowed during an active round.

  ### 2.5.2 Starting a New Round
  ```solidity   
    function startNewRound() public restricted {
        require(!isRoundActive, "A round is already active");

        // Ensure there are enough players to start a round
        require(players.length >= minimumPlayers, "Not enough players to start a round");

        roundEndTime = block.timestamp + 24 hours; // Round lasts for 24 hours
        isRoundActive = true;
    }
  ```
The `startNewRound` function is responsible for initiating a new lottery round. It resets the round-related variables, sets a new round end time, and marks the round as active.

  ### 2.5.3 Ending the Current Round
  ```solidity
    function endRound() public restricted roundActive {
        require(block.timestamp >= roundEndTime, "Round is still active");
        // Pick a winner only if there are enough players
        if (players.length > 0) {
            uint256 index = random() % players.length;
            lastWinner = players[index];

            // Emit winner event
            emit LotteryWinner(lastWinner, address(this).balance);

            // Transfer the entire contract balance to the winner
            payable(lastWinner).transfer(address(this).balance);
        }
  ```
The `endRound` function is responsible for finalizing the current round, selecting a winner randomly, transferring the winnings to the winner, and resetting the player's array and round status for the next round.

### 2.6 Checking Round Status and Participants
  ### 2.6.1 Getting Current Round Status
  ```solidity
   function getCurrentRoundStatus() public view returns (bool active, uint256 endTime) {
        return (isRoundActive, roundEndTime);
    }
   ```
This function queries the contract and obtains information about the current round's status and end time.

  ### 2.6.2 Generating a Random Number
```solidity
function random() private view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));
    }
  ```
  This function is a private view function designed to generate a pseudo-random number.
  
  ### 2.6.3 Retrieving Players
  ```solidity
  function getPlayers() public view returns (address[] memory) {
        return players;
    }
  ```
 This function retrieves the array of participant addresses (players) in the current lottery round.
 
### Section 3: Complete Code
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Lottery {
    address public manager;
    address[] public players;
    address public lastWinner;
    uint256 public roundEndTime;
    uint256 public minimumPlayers;
    bool public isRoundActive;

    event LotteryWinner(address winner, uint256 amount);

    modifier restricted() {
        require(msg.sender == manager, "Only the manager can call this function");
        _;
    }

    modifier roundActive() {
        require(isRoundActive, "The current round is not active");
        _;
    }

    constructor(uint256 _minimumPlayers) {
        manager = msg.sender;
        minimumPlayers = _minimumPlayers;
        isRoundActive = false;
    }

    function enter() public payable {
        require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether");
        require(!isRoundActive, "Cannot enter while a round is active");
        
        players.push(msg.sender);
    }

    function startNewRound() public restricted {
        require(!isRoundActive, "A round is already active");

        // Ensure there are enough players to start a round
        require(players.length >= minimumPlayers, "Not enough players to start a round");

        roundEndTime = block.timestamp + 24 hours; // Round lasts for 24 hours
        isRoundActive = true;
    }

    function endRound() public restricted roundActive {
        require(block.timestamp >= roundEndTime, "Round is still active");

        // Pick a winner only if there are enough players
        if (players.length > 0) {
            uint256 index = random() % players.length;
            lastWinner = players[index];

            // Emit winner event
            emit LotteryWinner(lastWinner, address(this).balance);

            // Transfer the entire contract balance to the winner
            payable(lastWinner).transfer(address(this).balance);
        }

        // Reset the players array and round status for the next round
        players = new address[](0);
        isRoundActive = false;
    }

    function getPlayers() public view returns (address[] memory) {
        return players;
    }

    function getCurrentRoundStatus() public view returns (bool active, uint256 endTime) {
        return (isRoundActive, roundEndTime);
    }

    function random() private view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));
    }
}

```
### Section 4: Getting Celo faucet 
The Celo Faucet is a tool that allows developers to obtain testnet CELO tokens for development and testing purposes on the CELO blockchain.

Visit Celo Faucet: [celo faucet]:https://faucet.celo.org/alfajores
Paste your wallet address. You can authenticate with your Github to get 10x of the token.
![celoFaucet](https://github.com/bodmandao/celo_decentralized_lottery/assets/154741685/d0ac677e-e78c-4c89-a888-6f06a6ebce2d)

Click on the faucet to get your token.

### Section 5: Deployment to remix
To bring the smart contract to life, we leverage Celo Faucet to obtain testnet CELO tokens for development and testing on the CELO blockchain. By configuring your Metamask wallet for the Celo Alfajores testnet and connecting it to Remix IDE, you can compile and deploy the contract. Once deployed, you can interact with the contract functions, enter rounds, check statuses, and witness the transparent execution of the lottery on the blockchain.

Go to remix[]:https://remix.ethereum.org/

Create a new file named `lottery.sol` and paste the complete code.
![remix](https://github.com/bodmandao/celo_decentralized_lottery/assets/154741685/2168e375-cdbb-4f3d-b3d8-aff239b9d148)

Compile the lottery contract.

Don't forget to configure `celo alfajores testnet` on your metamask. You can refer to this documentation [Link]:https://celo.academy/t/3-simple-steps-to-connect-your-metamask-wallet-to-celo/84

Under the `environment` tab, select `injected provider` to connect to metamask or any equivalent.

Set `minimum player` and click on `deploy` button on remix after you have successfully connected your wallet.

Congratulations! You have successfully deployed a decentralized lottery on the CELO blockchain. You can go ahead to interact with your contract.
![deployed](https://github.com/bodmandao/celo_decentralized_lottery/assets/154741685/8b2c461a-c6a6-48c5-b7df-f5c66b2d1fc0)

### Section 6: Conclusion

Decentralized lotteries on CELO represent a fascinating application of blockchain technology, offering transparency, fairness, and trust to the world of chance. By building and deploying your smart contract, you gain a deeper understanding of DApp development and witness the power of blockchain in action. So, take the plunge, explore the code, and unleash your creativity to build a decentralized lottery that resonates with your community. Remember, the possibilities are boundless, and the future of entertainment lies in your hands (or rather, your smart contract!)

### Section 7: Next Steps

  While this guide provides a solid foundation for building a decentralized lottery, further enhancements can be explored. Integrating Chainlink VRF (Verifiable Random Function) can enhance randomness generation, eliminating potential biases. Implementing additional features like tiered prizes, bonus systems, or automated round transitions can add complexity and excitement to the game. The possibilities are endless, allowing you to customize your lottery DApp and cater to diverse player preferences.
