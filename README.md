# Project Title
MyToken Smart Contract

# Overview
MyToken is a custom Ethereum smart contract that implements a fungible token with additional functionalities like pausability and blacklisting. It allows the creation, transfer, and management of tokens while ensuring certain restrictions can be enforced by the contract owner.

# Features
Basic Token Details: Name, symbol, decimals, and total supply.
Token Transfer and Allowance: Standard token transfer functions and allowances.
Minting and Burning: Functions to mint new tokens and burn existing ones.
Pausability: Ability to pause and unpause the contract to halt operations temporarily.
Blacklisting: Mechanism to prevent certain addresses from participating in token transfers.
Ownership Management: Functions to manage contract ownership.

# code
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyToken {
    string public name;
    string public symbol;
    uint8 public decimals;
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    mapping(address => bool) public blacklisted;
    bool public paused = false;
    address public owner;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event Paused();
    event Unpaused();
    event Blacklisted(address indexed account);
    event Unblacklisted(address indexed account);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can call this function.");
        _;
    }
    modifier whenNotPaused() {
        require(!paused, "Token transfer while paused.");
        _;
    }
    modifier notBlacklisted(address account) {
        require(!blacklisted[account], "Address is blacklisted.");
        _;
    }

    constructor(string memory _name, string memory _symbol, uint8 _decimals, uint256 _initialSupply) {
        name = _name;
        symbol = _symbol;
        decimals = _decimals;
        totalSupply = _initialSupply * 10**uint256(decimals);
        balanceOf[msg.sender] = totalSupply;
        owner = msg.sender;
    }

    function transfer(address _to, uint256 _value) external whenNotPaused notBlacklisted(msg.sender) notBlacklisted(_to) returns (bool) {
        require(_to != address(0) && balanceOf[msg.sender] >= _value, "Invalid recipient or insufficient balance.");
        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value) external whenNotPaused notBlacklisted(msg.sender) returns (bool) {
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) external whenNotPaused notBlacklisted(_from) notBlacklisted(_to) returns (bool) {
        require(_to != address(0) && balanceOf[_from] >= _value && allowance[_from][msg.sender] >= _value, "Invalid recipient, insufficient balance, or insufficient allowance.");
        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;
        emit Transfer(_from, _to, _value);
        return true;
    }

    function mint(address _to, uint256 _value) external onlyOwner whenNotPaused notBlacklisted(_to) returns (bool) {
        require(_to != address(0), "Invalid recipient.");
        totalSupply += _value;
        balanceOf[_to] += _value;
        emit Transfer(address(0), _to, _value);
        return true;
    }

    function burn(uint256 _value) external whenNotPaused notBlacklisted(msg.sender) returns (bool) {
        require(balanceOf[msg.sender] >= _value, "Insufficient balance.");
        balanceOf[msg.sender] -= _value;
        totalSupply -= _value;
        emit Transfer(msg.sender, address(0), _value);
        return true;
    }

    function pause() external onlyOwner {
        paused = true;
        emit Paused();
    }

    function unpause() external onlyOwner {
        paused = false;
        emit Unpaused();
    }
  function blacklist(address account) external onlyOwner {
        blacklisted[account] = true;
        emit Blacklisted(account);
    }

    function unblacklist(address account) external onlyOwner {
        blacklisted[account] = false;
        emit Unblacklisted(account);
    }
}

# Authors
Atul Mishra
- [atulmishra4709](https://github.com/atulmishra4709)

# License

This project is licensed under the MIT License - see the LICENSE file for details.

