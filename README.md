from flask import Flask, request, jsonify

app = Flask(_name_)

# Sample data structures to store users and links
users = {}
links = {}
admin_id = "admin"
admin_phone = "0111277771"
users[admin_id] = {"balance": 0, "links": [], "phone": admin_phone, "funds": 0}

@app.route('/create_user', methods=['POST'])
def create_user():
    user_id = request.json.get("user_id")
    initial_funds = request.json.get("funds", 0)
    if user_id in users:
        return jsonify({"error": "User already exists"}), 400

    users[user_id] = {"balance": 0, "links": [], "funds": initial_funds}
    return jsonify({"message": "User created successfully", "user_id": user_id}), 201

@app.route('/create_link', methods=['POST'])
def create_link():
    parent_id = request.json.get("parent_id")
    if parent_id not in users:
        return jsonify({"error": "Parent not found"}), 404

    child_id = str(len(users) + 1)
    users[child_id] = {"balance": 0, "links": [], "funds": 0}
    links[child_id] = {"parent": parent_id, "link": f"{request.host_url}join/{child_id}"}
    users[parent_id]["links"].append(child_id)

    return jsonify({"child_id": child_id, "link": links[child_id]["link"]}), 201

@app.route('/join/<child_id>', methods=['POST'])
def join(child_id):
    user_id = request.json.get("user_id")
    if child_id not in users:
        return jsonify({"error": "Link not found"}), 404

    if user_id not in users:
        return jsonify({"error": "User not found"}), 404

    # Simulate payment
    payment = 100  # Simulated amount in KSH

    if users[user_id]["funds"] < payment:
        return jsonify({"error": "Insufficient funds"}), 400

    parent_id = links[child_id]["parent"]
    users[admin_id]["balance"] += payment / 2
    users[parent_id]["balance"] += payment / 2
    users[user_id]["funds"] -= payment

    # Simulate payment distribution
    admin_payment = payment / 2
    referrer_payment = payment / 2
    admin_phone = users[admin_id]["phone"]

    # Here you would integrate with M-Pesa API to handle the actual transaction
    # For example purposes, we'll just print the payment distribution
    print(f"Simulated payment: {admin_payment} KSH to admin at {admin_phone}")
    print(f"Simulated payment: {referrer_payment} KSH to referrer with ID {parent_id}")

    return jsonify({"message": "Joined successfully", "admin_balance": users[admin_id]["balance"], "parent_balance": users[parent_id]["balance"]}), 200

@app.route('/balance/<user_id>', methods=['GET'])
def balance(user_id):
    if user_id not in users:
        return jsonify({"error": "User not found"}), 404

    return jsonify({"user_id": user_id, "balance": users[user_id]["balance"], "funds": users[user_id]["funds"]}), 200

if _name_ == '_main_':
