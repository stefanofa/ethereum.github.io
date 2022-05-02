# Link utili sviluppo Ethereum

 - Ethereum development official documentation:
   [_https://ethereum.org/en/developers/docs/_](https://ethereum.org/en/developers/docs/)
 - EIP - Ethereum Improvement Proposals:
   [_https://eips.ethereum.org/all_](https://eips.ethereum.org/all)
  
### Ambienti di sviluppo
- Remix IDE: [_https://remix.ethereum.org/_](https://remix.ethereum.org/)
- Hardhat: [_https://hardhat.org/getting-started/_](https://hardhat.org/getting-started/)
- Openzeppelin Docs: [_https://docs.openzeppelin.com/contracts/4.x/_](https://docs.openzeppelin.com/contracts/4.x/)
- Openzeppelin Github: [_https://github.com/OpenZeppelin/openzeppelin-contracts_](https://github.com/OpenZeppelin/openzeppelin-contracts)

### Smart Contract sviluppati

#### EtherSplit.sol
```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract EtherSplit {

	address[] private receivers;
	address deployer;

	modifier  onlyDeployer() {
		require(msg.sender  == deployer, 'Only deployer can do this operation');
		_;
	}
	
	constructor () {
		deployer =  msg.sender;
	}
	 
	function addReceiver (address _receiver) public onlyDeployer {
		receivers.push(_receiver);
	}

	function addReceivers (address[] memory _receivers) public {
		for (uint i =  0; i < _receivers.length; ++i) {
			addReceiver(_receivers[i]);
		}
	}

	receive () external payable {
		uint share = msg.value / receivers.length;
		for (uint i =  0; i < receivers.length; ++i) {
			payable(receivers[i]).transfer(share);
		}
		
		// if the amount of ether sent is not divisible by the number of receivers,
		// 'share' will be rounded down, so there may be a small amount of unused ether left
		// that will be sent back to the sender of the transaction
		// example :
		// - value send: 7 wei
		// - number of receivers: 3
		// -> share = 7 / 3 = 2
		// -> total amount split = 3 * 2 = 6
		// -> leftover = 7 - 6 = 1

		uint leftover =  address(this).balance;
		payable(msg.sender).transfer(leftover);
	}

}
```

#### SporteamsToken.sol

Standard ERC20: [https://eips.ethereum.org/EIPS/eip-20](https://eips.ethereum.org/EIPS/eip-20)  
Openzeppelin documentation: [https://docs.openzeppelin.com/contracts/4.x/erc20](https://docs.openzeppelin.com/contracts/4.x/erc20)  
Openzeppelin Implementation: [https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/ERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/ERC20.sol)  

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma  solidity  ^0.8.0;

import  "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/ERC20.sol";
  
contract  SporteamsToken  is  ERC20 {

	uint8  private decimals;

	constructor(uint8 _decimals) ERC20("SporteamsToken", "SPT") {
	  decimals = _decimals;
	}

	function  decimals() public  view  virtual  override  returns (uint8) {
	  return decimals;
	}

	function  receiveOneSPT() external {
		_mint(msg.sender, 10  ** decimals);
	}

}
```

#### SporteamsNFT.sol

Standard ERC721: [https://eips.ethereum.org/EIPS/eip-721](https://eips.ethereum.org/EIPS/eip-721)  
Openzeppelin documentation: [https://docs.openzeppelin.com/contracts/4.x/erc721](https://docs.openzeppelin.com/contracts/4.x/erc721)  
Openzeppelin Implementation: [https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC721/ERC721.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC721/ERC721.sol)  

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC721/ERC721.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/utils/Counters.sol";

contract SporteamsNFT is ERC721 {
	using Counters for Counters.Counter;

	Counters.Counter private tokenIdTracker;
	mapping(uint256 => string)  private tokenURIs;
	
	constructor () ERC721("SporteamsCollection", "SPTNFT") {}

	function tokenURI(uint256 tokenId) public view virtual override  returns (string memory) {
		require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");
		return tokenURIs[tokenId];
	}

	function mintMyNFT(string memory _tokenURI, address _to) external {
		uint tokenId = tokenIdTracker.current();
		_mint(_to, tokenId);
		tokenURIs[tokenId] = _tokenURI;
		tokenIdTracker.increment();
	}

}
```
