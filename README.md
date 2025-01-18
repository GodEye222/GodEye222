from flask import Flask, request, jsonify
from flask_socketio import SocketIO
from web3 import Web3
import random

# Flask app setup
app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret!'
socketio = SocketIO(app)

# Blockchain setup (Ethereum)
infura_url = "https://goerli.infura.io/v3/YOUR_INFURA_PROJECT_ID"
web3 = Web3(Web3.HTTPProvider(infura_url))
contract_address = "YOUR_CONTRACT_ADDRESS"
private_key = "YOUR_PRIVATE_KEY"
account_address = "YOUR_WALLET_ADDRESS"

# Dummy ABI (Replace with actual smart contract ABI)
contract_abi = """
[
  // Your contract ABI goes here
]
"""

# Load the smart contract
contract = web3.eth.contract(address=contract_address, abi=contract_abi)

@app.route('/deposit', methods=['POST'])
def deposit():
    data = request.json
    user_address = data.get("user_address")
    amount = data.get("amount")  # In ETH
    
    if not web3.isAddress(user_address):
        return jsonify({"error": "Invalid address"}), 400
    
    try:
        nonce = web3.eth.get_transaction_count(account_address)
        tx = {
            'nonce': nonce,
            'to': contract_address,
            'value': web3.toWei(amount, 'ether'),
            'gas': 2000000,
            'gasPrice': web3.toWei('50', 'gwei')
        }
        signed_tx = web3.eth.account.sign_transaction(tx, private_key)
        tx_hash = web3.eth.send_raw_transaction(signed_tx.rawTransaction)
        return jsonify({"message": "Deposit successful", "tx_hash": tx_hash.hex()})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/play', methods=['POST'])
def play_game():
    data = request.json
    user_address = data.get("user_address")
    
    if not web3.isAddress(user_address):
        return jsonify({"error": "Invalid address"}), 400

    # Game logic: simple coin flip
    result = random.choice(["win", "lose"])
    if result == "win":
        reward = web3.toWei(0.01, 'ether')  # Fixed reward
        try:
            nonce = web3.eth.get_transaction_count(account_address)
            tx = {
                'nonce': nonce,
                'to': user_address,
                'value': reward,
                'gas': 2000000,
                'gasPrice': web3.toWei('50', 'gwei')
            }
            signed_tx = web3.eth.account.sign_transaction(tx, private_key)
            tx_hash = web3.eth.send_raw_transaction(signed_tx.rawTransaction)
            return jsonify({"message": "You won!", "reward": web3.fromWei(reward, 'ether'), "tx_hash": tx_hash.hex()})
        except Exception as e:
            return jsonify({"error": str(e)}), 500
    else:
        return jsonify({"message": "You lost!"})

if __name__ == '__main__':
    socketio.run(app, debug=True)
