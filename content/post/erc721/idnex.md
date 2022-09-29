---
title: ERC721合约

description: 标准实现ERC721智能合约

slug: erc721

date: 2022-09-29

categories:
- Contract

tags:
- Solidity
---
### ERC721

```solidity
// SPDX-License-Identifier: SimPL-2.0

pragma solidity ^0.8.10;

interface IERC721Receiver{
  function onERC721Received(
        address operator,
        address from,
        uint256 tokenId,
        bytes calldata data
    ) external returns (bytes4);
}

contract Cafe721 {
  using SafeMath for uint256;
  //creator
  address private contractCreator;

  // name
  string private _name;

  // symbol
  string private _symbol;


  // Token
  struct Cafe {
      address creator;
      address owner;
      address approved;
      bytes32 summary;
      string uri;
  }

  // all Cafe
  mapping (uint256 => Cafe) allCafe;

  // cafe's counts
  uint256 _totalCafe;

  // User
  struct User {
    uint256[] cafes;
    uint256 totalCafe;
  }

  // all user
  mapping (address => User) allUser;

  // operator approvals
  mapping (address => mapping (address => bool)) operatorApprovals;

  constructor(string memory name_, string memory symbol_){
    contractCreator = theSender();
    _name = name_;
    _symbol = symbol_;
  }

  event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);

  event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

  event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);


  /*
  * the address have how much tokens
  */
  function balanceOf(address _owner) external view returns (uint256 balance) {
    return allUser[_owner].totalCafe;
  }

  function cafesOf(address _owner) external view returns (uint256[] memory cafes){
    return allUser[_owner].cafes;
  }

  /*
  * the token's owner
  */
  function ownerOf(uint256 _tokenId) external view returns (address owner) {
    return allCafe[_tokenId].owner;
  }

  function creatorOf(uint256 _tokenId) external view returns (address creator) {
    return allCafe[_tokenId].creator;
  }



  // trans begin
  function transferFrom(address _from, address _to, uint256 _tokenId) public returns (bool success) {
    // checked token
    require(exists(_tokenId), "Cafe721: the cafe is not exists");
    // checked addr
    require(zeroAddr(_from), "Cafe721: can not from the zero address");
    require(zeroAddr(_to), "Cafe721: can not to the zero address");
    // checked permission
    require(isApprovedOrOwner(_tokenId), "Cafe721: no permission");

    // remove cafe
    remove(_from, _tokenId);

    // transfer cafe
    allCafe[_tokenId].owner = _to;
    allCafe[_tokenId].approved = address(0);

    // add cafe
    allUser[_to].cafes.push(_tokenId);
    allUser[_to].totalCafe = allUser[_to].totalCafe.add(1);

    emit Transfer(_from, _to, _tokenId);

    return true;
  }
  function safeTransferFrom(address _from, address _to, uint256 _tokenId) external virtual  returns (bool success) {
    return safeTransferFrom(_from, _to, _tokenId, "");
  }

  function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes memory _data) public virtual returns (bool success) {
    require(isApprovedOrOwner(_tokenId), "Cafe721: no permission");
    transferFrom(_from, _to, _tokenId);
    require(checkOnERC721Received(_from, _to, _tokenId, _data), "Cafe721: transfer to non ERC721Receiver implementer");
    return true;
  }

  // trans end


  function approve(address _approved, uint256 _tokenId) external returns (bool success) {
    require(allCafe[_tokenId].owner == theSender(), "Cafe721: no permission");
    require(_approved != theSender(), "Cafe721: can not approved for owner");
    allCafe[_tokenId].approved = _approved;
    emit Approval(theSender(), _approved, _tokenId);
    return true;
  }

  function setApprovalForAll(address _operator, bool _approved) external returns (bool success) {
    require(_operator != theSender(), "Cafe721: can not approved for owner");
    operatorApprovals[theSender()][_operator] = _approved;
    emit ApprovalForAll(theSender(), _operator, _approved);
    return true;
  }


  function getApproved(uint256 _tokenId) private view returns(address) {
    return allCafe[_tokenId].approved;
  }

  function isApprovedForAll(address _owner, address _operator) public view returns (bool) {
    return operatorApprovals[_owner][_operator];
  }

  function name() external view returns (string memory) {
    return _name;
  }

  function symbol() external view returns (string memory) {
    return _symbol;
  }

  function tokenURI(uint256 _tokenId) external view returns (string memory uri) {
    return allCafe[_tokenId].uri;
  }

  function tokenSummary(uint256 _tokenId) external view returns (bytes32 summary) {
    return allCafe[_tokenId].summary;
  }

  function totalSupply() external view returns (uint256 totalCafe) {
    return _totalCafe;
  }


  function exists(uint256 _tokenId) internal view returns (bool){
    return allCafe[_tokenId].owner != address(0);
  }

  function zeroAddr(address addr) private pure returns (bool) {
    return addr != address(0);
  }

  function remove(address _from, uint256 _tokenId) private {
    uint[] storage cafes = allUser[_from].cafes;
    for (uint i = 0; i < cafes.length - 1; i++){
      if(cafes[i] == _tokenId){
        cafes[i] = cafes[cafes.length + 1];
        break;
      }
    }
    delete cafes[cafes.length - 1];
    cafes.pop();
    allUser[_from].cafes = cafes;
    allUser[_from].totalCafe = allUser[_from].totalCafe.sub(1);
  }

    /*
    * address owner;
    * address approved;
    * bytes32 summary;
    * string uri; 
    */

  function mint(uint256 _tokenId, bytes32 _summary, string memory _uri) external {
    require(allCafe[_tokenId].creator == address(0), "Cafe721: the cafe is exists.");

    allCafe[_tokenId].creator = theSender();
    allCafe[_tokenId].owner = theSender();
    allCafe[_tokenId].approved = address(0);
    allCafe[_tokenId].summary = _summary;
    allCafe[_tokenId].uri = _uri;

    // User
    allUser[theSender()].cafes.push(_tokenId);
    allUser[theSender()].totalCafe = allUser[theSender()].totalCafe.add(1);

    _totalCafe = _totalCafe.add(1);
  }


  function isApprovedOrOwner(uint256 _tokenId) private view returns (bool) {
    require(exists(_tokenId), "Cafe721: the cafe is exists.");
    return (theSender() == allCafe[_tokenId].owner || isApprovedForAll(allCafe[_tokenId].owner, theSender()) || getApproved(_tokenId) == theSender());
  }

  function checkOnERC721Received(address from,address to,uint256 tokenId,bytes memory _data) private returns (bool) {
    if (to.code.length > 0) {
            try IERC721Receiver(to).onERC721Received(theSender(), from, tokenId, _data) returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
                if (reason.length == 0) {
                    revert("ERC721: transfer to non ERC721Receiver implementer");
                } else {
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }
        } else {
            return true;
        }
  }

  // destroyCafe
  function destroyCafe(uint256 _tokenId) external  returns (bool success) {
    require(allCafe[_tokenId].owner == theSender(), "Cafe721: no premission");
    remove(theSender(), _tokenId);
    delete allCafe[_tokenId];

    _totalCafe = _totalCafe.sub(1);
    return true;
  }


  function theSender() internal view returns (address) {
    return msg.sender;
  }
}

library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    // Gas optimization: this is cheaper than requiring 'a' not being zero, but the
    // benefit is lost if 'b' is also tested.
    // See: https://github.com/OpenZeppelin/openzeppelin-solidity/pull/522
    if (a == 0) {
      return 0;
    }

    uint256 c = a * b;
    require(c / a == b);

    return c;
  }

  /**
  * @dev Integer division of two numbers truncating the quotient, reverts on division by zero.
  */
  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b > 0); // Solidity only automatically asserts when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold

    return c;
  }

  /**
  * @dev Subtracts two numbers, reverts on overflow (i.e. if subtrahend is greater than minuend).
  */
  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b <= a);
    uint256 c = a - b;

    return c;
  }

  /**
  * @dev Adds two numbers, reverts on overflow.
  */
  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    require(c >= a);

    return c;
  }

  /**
  * @dev Divides two numbers and returns the remainder (unsigned integer modulo),
  * reverts when dividing by zero.
  */
  function mod(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b != 0);
    return a % b;
  }

}

```

