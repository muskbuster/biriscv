# Ciphertext Routing ISM

## Introduction

The Routing ISM allows usage of different ISMs based on the context of the message being received .\
In the context of Ciphertext Routing ISM. Based on the message type it checks whether it is a normal cross chain message or requires Ciphertext to be fetched from an offchain server and accordingly calls the Multisig ISM or the CCIP-read ISM.

## Usage

### Modification on sender contract

* The message being dispatched must be prepended with `uint8`before the message data.
* **To Use CCIP Read ISM** -The uint8 value should be zero (0) if CCIP-read ISM needs to be used to fetch ciphertext. followed by the hash of ciphertext needed to be fetched from offchain.\
  `abi.encode(uint8(0),Ciphertext_Hash, arbitrary data)`
* **To use normal message passing** - The uint8 value should be one (1).\
  `abi.encode(uint8(1), arbitrary data)`

### Modification on recipient Contract at inco

The logic necessary to execute any functionality other than CCIP needs to be written in the handle function. However if the CCIP ism is being used another handler called `handleWithCiphertext` needs to be written. as per the standard defined [here](./#inco-gentry-smart-contract).

so the code snippet will look something like this&#x20;

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.19;

import {IMailbox} from "@hyperlane-xyz/core/contracts/interfaces/IMailbox.sol";
import {IPostDispatchHook} from "@hyperlane-xyz/core/contracts/interfaces/hooks/IPostDispatchHook.sol";
import {IInterchainSecurityModule} from "@hyperlane-xyz/core/contracts/interfaces/IInterchainSecurityModule.sol";
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract CipherTextProcessor is OwnableUpgradeable{
    IInterchainSecurityModule public interchainSecurityModule;
        function setInterchainSecurityModule(address _module) public {
        interchainSecurityModule = IInterchainSecurityModule(_module);
    }
    event handled (bytes32 hash);
    address mailbox; // address of mailbox contract
    address public ISM;

    constructor(address _mailbox, address _interchainSecurityModule) {
        mailbox = _mailbox;
        ISM=_interchainSecurityModule;
        //Sets CCIP read ISM as ISM to be used
        setInterchainSecurityModule(_interchainSecurityModule);
        
    }

    // Modifier so that only mailbox can call particular functions
    modifier onlyMailbox() {
        require(
            msg.sender == mailbox,
            "Only mailbox can call this function !!!"
        );
        _;
    }
        modifier onlyISM() {
        require(
            msg.sender == ISM,
            "Only mailbox can call this function !!!"
        );
        _;
    }
 // Handle general message passing here i.e when uint8 value is 1
  function handle( uint32 _origin,
        bytes32 _sender,
        bytes memory _body)external onlyMailbox {
           (uint8 handler)=  abi.decode(_body, (uint8));
           if(handler==1)
           {
               // Logic to handle and decode general message passing
           
           }
           emit handled(committedHash);
           

  }

  //**************************************************************************

  //*************************************************************************
  // Necessary function to be implemented with no changes as it is  called by ISM with metadata
  function handleWithCiphertext( uint32 _origin,
        bytes32 _sender,
        bytes memory _message)external onlyISM {
  (bytes memory message,bytes memory ciphertext)=abi.decode(_message,(bytes , bytes));
   (bytes32 committedHash)=  abi.decode(message, (bytes32));
   //rest of function logic 
        }
  //**************************************************************************

}
```

