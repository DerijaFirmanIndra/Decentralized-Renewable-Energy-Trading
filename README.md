# Decentralized-Renewable-Energy-Trading
Build a WEB3-based platform for trading renewable energy credits, promoting the adoption of clean energy sources.
from flask import Flask, jsonify, request
from web3 import Web3
from solcx import compile_source

# Simulated smart contract code for issuing and trading renewable energy credits
smart_contract_code = """
pragma solidity ^0.5.0;

contract RenewableEnergyCredits {
    address public owner;
    mapping(address => uint256) public balances;

    constructor() public {
        owner = msg.sender;
    }

    function issueCredits(address to, uint256 amount) public {
        require(msg.sender == owner, "Only the owner can issue credits");
        balances[to] += amount;
    }

    function transferCredits(address to, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
"""

# Compile the smart contract
compiled_sol = compile_source(smart_contract_code, output_values=["abi", "bin"])
contract_interface = compiled_sol['<stdin>:RenewableEnergyCredits']

# Connect to local Ethereum node
w3 = Web3(Web3.EthereumTesterProvider())
if not w3.isConnected():
    print("Failed to connect to Ethereum node.")
    exit()

# Deploy the contract
RenewableEnergyCredits = w3.eth.contract(abi=contract_interface['abi'], bytecode=contract_interface['bin'])
tx_hash = RenewableEnergyCredits.constructor().transact()
tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash)
contract = w3.eth.contract(address=tx_receipt.contractAddress, abi=contract_interface['abi'])

# Flask web server for interacting with the contract
app = Flask(__name__)

@app.route('/issue_credits', methods=['POST'])
def issue_credits():
    address = request.json['address']
    amount = request.json['amount']
    tx_hash = contract.functions.issueCredits(address, amount).transact()
    tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash)
    return jsonify({'status': 'success', 'tx_receipt': tx_receipt.hex()}), 200

@app.route('/transfer_credits', methods=['POST'])
def transfer_credits():
    from_address = request.json['from_address']
    to_address = request.json['to_address']
    amount = request.json['amount']
    tx_hash = contract.functions.transferCredits(to_address, amount).transact({'from': from_address})
    tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash)
    return jsonify({'status': 'success', 'tx_receipt': tx_receipt.hex()}), 200

if __name__ == '__main__':
    app.run(debug=True)
