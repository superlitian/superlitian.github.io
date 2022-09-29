---
title: ERC20合约

description: 标准实现ERC20智能合约

slug: erc20

date: 2022-09-29

categories:
- Contract

tags:
- Solidity
---

### ERC20

```solidity
// SPDX-License-Identifier: SimPL-2.0

pragma solidity ^0.8.0;


contract Cafe20{

  using SafeMath for uint256;

  // creator
  address private _creator;

  // token name
  string private _name;

  // token symbol
  string private _symbol;

  // token totalSupply
  uint256 private _totalSupply;

  // address have how much token
  mapping (address => uint256) private balances;

  // owner approve spender spend how much token
  mapping (address => mapping (address => uint256)) private allowances;

  constructor(string memory name_, string memory symbol_) public {
    _creator = theSender();
    _name = name_;
    _symbol = symbol_;
  }

  /**
  * total tokens
  */
  function totalSupply() public view returns (uint256) {
    return _totalSupply;
  }

  /**
  * mints tokens, only creator can call this function
  */
  function mint(uint256 value) public returns (bool) {
    require(theSender() == _creator, "Cafe20: No permission");
    _totalSupply = _totalSupply.add(value);
    balances[_creator] = balances[_creator].add(value);
    return true;
  }

  // when the token transfer, must be call this function
  event Transfer(address indexed _from, address indexed _to, uint256 _value);

  // when the approval, must be call this function
  event Approval(address indexed _owner, address indexed _spender, uint256 _value);

  /**
  * return this token's name
  */
  function name() public view returns (string memory) {
    return _name;
  }

  function creator() public view returns (address) {
    return _creator;
  }

  /**
  * return this token's symbol
  */
  function symbol() public view returns (string memory) {
    return _symbol;
  }

  /**
  * return this token's number of decimal places
  */
  function decimals() public view virtual returns (uint8) {
    return 5;
  }

  /*
  * return the sender's balance
  */
  function balance() public view returns (uint256 balance) {
    return balances[theSender()];
  }


  /**
  * return the balance of this address
  */
  function balanceOf(address _owner) public view returns (uint256 balance) {
    return balances[_owner];
  }

  /**
  * transfer to address
  */
  function transfer(address _to, uint256 _value) public returns (bool success) {
    require(balances[theSender()] >= _value,"Cafe20: not enough balance");
    require(_to != address(0), "Cafe20: can not to the zero address transfer");
    balances[theSender()] = balances[theSender()].sub(_value);
    balances[_to] = balances[_to].add(_value);
    emit Transfer(theSender(), _to, _value);
    return true;
  }

  /**
  * transfer from address to other address
  */
  function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {

    if(theSender() == _from){
      return transfer(_to, _value);
    }

    uint256 tokens = allowance(_from, theSender());
    require(_from != address(0), "Cafe20: can not from the zero address transfer");
    require(_to != address(0), "Cafe20: can not to the zero address transfer");
    require(balances[_from] >= _value, "Cafe20: not enough tokens");

    require(tokens >= _value, "Cafe20: the allowance not enough tokens");

    // update balance
    balances[_from] = balances[_from].sub(_value);
    balances[_to] = balances[_to].add(_value);

    // update allowances
    allowances[_from][theSender()] = allowances[_from][theSender()].sub(_value);
    emit Transfer(_from, _to, _value);
    return true;
  }


  /**
  * approve the spender for msg.sender
  */
  function approve(address _spender, uint256 _value) public returns (bool success) {
    address _owner = theSender();
    require(_spender != address(0), "Cafe20: can not approve the zero address");
    require(balances[_owner] >= _value, "Cafe20: not enough tokens");
    allowances[_owner][_spender] = allowances[_owner][_spender].add(_value);
    emit Approval(_owner, _spender, _value);
    return true;
  }

  /*
  * query the amount which _spender is still allowed to withdraw from _owner
  */
  function allowance(address _owner, address _spender) public view returns (uint256 remaining) {
    return allowances[_owner][_spender];
  }

  /*
  * destroy the tokens
  */
  function destroyTokens(uint256 _value) public returns (bool success) {
    require(balances[theSender()] >= _value, "Cafe20: not enough tokens");
    balances[theSender()] = balances[theSender()].sub(_value);
    _totalSupply = _totalSupply.sub(_value);
    return true;
  }

  /*
  * msg.sender
  */
  function theSender() private view returns (address) {
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

