# `CrossChain NFT`

> Effortlessly transfer NFT's from one chain to another. Made using Router Cross-Talk.

🚀DEMO: [Link to be given]

This project is built with [Router CrossTalk](https://dev.routerprotocol.com/crosstalk-library/overview/introduction)

Router Protocol is a solution introduced to address the issues hindering the usability of cross-chain liquidity migration in the DeFi ecosystem. It acts as a bridge connecting various layer 1 and layer 2 blockchains, allowing for the flow of contract-level data across them. The Router Protocol can either transfer tokens between chains or initiate operations on one chain and execute them on another.

Please check the [official documentation of Router Protocol](https://www.routerprotocol.com/) 

![CrossChain NFT](Demo gif to placed here)

# ⭐️ `Star us`

If this repository helps you build cross-chain dapps faster and easier - please star this project, every star makes us very happy!

# 🤝 `Need help?`

If you need help or have other some questions - don't hesitate to write in our discord channel and we will check asap. [Discord link](https://discord.gg/xvx2pFu9). The best thing about this is the super active community ready to help at any time! We help each other.

# 🚀 `Quick Start`

📄 Clone or fork `CrossChain NFT`:

```sh
git clone https://github.com/protocol-designer/CrossChain-NFT.git
```

💿 Install all dependencies:

```sh
cd cross-chat-main
npm install
```

🚴‍♂️ Run your App:

```sh
npm start
```
# 🧭 `Table of contents`
- [🚀 Quick Start](#-quick-start)
- [🧭 Table of contents](#-table-of-contents)
- [🏗 Frontend](#React JS, Ether.js)
  
 
- [🏗 Backend](#Solidity, Router Cross-Talk Library)
  - [`Initiating the Contract`](#Initiating-the-Contract)
  - [`Creating state variables and the constructor`](#Creating-state-variables-and-the-constructor)
  - [`Sending a message to the destination chain`](#Sending-a-message-to-the-destination-chain)
  - [`Handling a crosschain request`](#Handling-a-crosschain-request)
  - [`Handling the acknowledgement received from destination chain`](#Handling-the-acknowledgement-received-from-destination-chain)
  
# 🏗 Frontend


# 🏗 Backend
  
## `Initiating the Contract`

For initiating the smart contract named "CrossChainNFT", the contract imports four external contracts :-

1. **ICrossTalkApplication.sol**

2. **Utils.sol**

3. **IGateway.sol**

4. **ERC1155.sol**

The "ICrossTalkApplication.sol", "Utils.sol" and "IGateway.sol" contracts are imported from the "evm-gateway-contract/contracts" and "ERC1155.sol" from "openzeppelin/contracts/token".The "CrossChain" contract implements the "ICrossTalkApplication" and "ERC1155.sol" contract by inheriting from them. This means that the "CrossChainNFT" contract must have all the functions and variables defined in the "ICrossTalkApplication" contract. By importing and implementing these contracts, the "CrossChainNFT" contract will have access to their functionality and will be compatible with other contracts that follow the same standards.

```sh
//SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.8.0 <0.9.0;

import "evm-gateway-contract/contracts/ICrossTalkApplication.sol";
import "evm-gateway-contract/contracts/Utils.sol";
import "evm-gateway-contract/contracts/contracts/CrossTalkUtils.sol";
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";

contract CrossChainNFT is ERC1155, ICrossTalkApplication {
}
```
## `Creating State Variables and the Constructor`

The smart contract has the following state variables:

1. **admin** - an address variable which stores the address of the admin. This will be used for access control purposes.

2. **gatewayContract** - an address variable which holds the address of the gateway contract. This contract will route messages to the Router Chain.

3. **destGasLimit** - a uint64 variable which indicates the amount of gas required to execute the function that will handle cross-chain requests on the destination chain.

4. **ourContractOnChains** - a mapping which maps a chain type and chain ID to the address of NFT contracts deployed on different chains. This mapping will be used to set the address of the destination contract.

The smart contract also defines a struct TransferParams which includes the following parameters:

1. **nftIds** - an array of uint256 values which represent the IDs of the NFTs to be transferred.

2. **nftAmounts** - an array of uint256 values which represent the amounts of each NFT to be transferred.

3. **nftData** - a bytes variable which holds additional data related to the NFT transfer.

4. **recipient** - a bytes variable which holds the address of the recipient of the NFT transfer.

The constructor of the smart contract takes three parameters:

1. **uri** - a string which represents the URI for the NFTs being created.

2. **gatewayAddress** - an address variable which holds the address of the gateway contract.

3. **_destGasLimit** - a uint64 variable which indicates the amount of gas required to execute the function that will handle cross-chain requests on the destination chain.
4. 
The smart contract extends the ERC1155 standard and includes all the required functions such as balanceOf, safeTransferFrom, setApprovalForAll, and others.

```sh
address public admin;
address public gatewayContract;
uint64 public destGasLimit;
// chain type + chain id => address of our contract in bytes
mapping(uint64 => mapping(string => bytes)) public ourContractOnChains;

struct TransferParams {
    uint256[] nftIds;
    uint256[] nftAmounts;
    bytes nftData;
    bytes recipient;
}

constructor(
    string memory uri,
    address payable gatewayAddress, 
    uint64 _destGasLimit
) ERC1155(uri) {
    gatewayContract = gatewayAddress;
    destGasLimit = _destGasLimit;
    admin = msg.sender;
}
```

## `Setting up the Destination Contract on the Source Contract`

**setContractOnChain**:-

The given code defines a setter function setContractOnChain which allows the admin to set the address of a destination contract on the source chain and visa versa. The function takes three parameters:

chainType - a uint64 variable which represents the type of the chain (e.g. Ethereum, Binance Smart Chain, etc.)
chainId - a string which represents the ID of the chain where the contract is deployed.
contractAddress - an address variable which holds the address of the NFT contract deployed on the specified chain.
The function first checks whether the caller is the admin by comparing the msg.sender with the admin address variable. If the caller is not the admin, the function will revert with an error message "only admin".

If the caller is the admin, the function will convert the contractAddress to bytes using the toBytes function and store it in the ourContractOnChains mapping using the chainType and chainId as the keys.

```sh
function setContractOnChain(
    uint64 chainType, 
    string memory chainId, 
    address contractAddress
) external {
    require(msg.sender == admin, "only admin");
    ourContractOnChains[chainType][chainId] = toBytes(contractAddress);
}
```
## `Handling a crosschain request`

**handleRequestFromSource function:-**

The function is designed to handle a request that originates from a source chain , passes through a router chain, and arrives at the contract on a destination blockchain.

The function takes four parameters:

1. **srcContractAddress**: the address of the source contract that initiated the request.
 
2. **payload**: a byte array that contains the payload of the request.
 
3. **srcChainId**: a string that represents the ID of the source blockchain.

4. **srcChainType**: an unsigned 64-bit integer that represents the type of the source blockchain.
The function is marked as "external" and "override", meaning that it can be called from outside the contract and it overrides a function with the same name and signature in the contract it inherits from.

The function first checks that the caller of the function is the "gatewayContract" by using the "require" statement. If the caller is not the gateway contract, the function will stop executing and return an error.

The function then decodes the "payload" parameter into a string variable called "sampleStr". It checks if the string is empty by comparing its hash to the hash of an empty string using the "keccak256" function. If the string is empty, the function will stop executing and return a custom error message.

If the string is not empty, the function will set the value of a global variable called "greeting" to the value of the "sampleStr" variable.

Finally, the function will return a byte array that contains the "srcChainId" and "srcChainType" parameters encoded using the "abi.encode" function.

 ```sh
 function handleRequestFromSource(
  bytes memory srcContractAddress,
  bytes memory payload,
  string memory srcChainId,
  uint64 srcChainType
) external override returns (bytes memory) {
  require(msg.sender == gatewayContract);

  string memory sampleStr = abi.decode(payload, (string));

  if (
    keccak256(abi.encodePacked(sampleStr)) == keccak256(abi.encodePacked(""))
  ) {
    revert CustomError("String should not be empty");
  }
  greeting = sampleStr;
  return abi.encode(srcChainId, srcChainType);
}
```
 
## `Handling the acknowledgement received from destination chain`

**handleCrossTalkAck function:-**

This function handles the acknowledgement sent by the destination chain to the source chain after a successful cross-chain communication. The function takes three parameters: the event identifier, a boolean array of execution flags, and a byte array of execution data. It is an external view function and is marked as an override of a parent contract's function.

The function first checks that the event identifier passed in as the first parameter matches the lastEventIdentifier variable, which is a state variable tracking the most recent cross-chain communication event. If the event identifier does not match, the function will revert.

Next, the function decodes the first element of the execData array, which is assumed to be a byte array. The decoded bytes are then further decoded as a tuple of a string and a uint64, representing the chain ID and chain type of the source chain that initiated the cross-chain communication.

After decoding the execution data, the function emits two events. The first event is an **ExecutionStatus** event that emits the event identifier and the first element of the execFlags array as parameters. The second event is a **ReceivedSrcChainIdAndType** event that emits the chain type and chain ID of the source chain as parameters.

1. **if the execution was successful on the destination chain:**
    
    We will get [true] in execFlags and [abi.encode(abi.encode(sourceChainType, sourceChainId))] in execData as we sent this as return value in handleRequestFromSource function.
    
2. **If the execution failed on the destination chain:**
    
    We will get [false] in execFlags and [errorBytes] in execData where error bytes correspond to the error that was thrown on the destination chain contract

```sh
function handleCrossTalkAck(
    uint64 eventIdentifier,
    bool[] memory execFlags,
    bytes[] memory execData
  ) external view override {
    require(lastEventIdentifier == eventIdentifier);
		bytes memory _execData = abi.decode(execData[0], (bytes));

    (string memory chainID, uint64 chainType) = abi.decode(
      _execData,
      (string, uint64)
    );
		
    emit ExecutionStatus(eventIdentifier, execFlags[0]);
	
    emit ReceivedSrcChainIdAndType(chainType, chainID);
  }
  ```
