from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

# Simulated user database
users = {
    "admin": {"password": "1234", "role": "admin"},
    "user": {"password": "abcd", "role": "user"}
}

# Authentication decorator
def authenticate(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = request.authorization
        if not auth or not check_credentials(auth.username, auth.password):
            return jsonify({"message": "Authentication required"}), 401
        return f(*args, **kwargs)
    return decorated

# Authorization decorator
def authorize(role):
    def decorator(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            auth = request.authorization
            if users[auth.username]["role"] != role:
                return jsonify({"message": "Unauthorized"}), 403
            return f(*args, **kwargs)
        return decorated
    return decorator

# Helper function to check credentials
def check_credentials(username, password):
    user = users.get(username)
    return user and user["password"] == password

@app.route('/admin', methods=['GET'])
@authenticate
@authorize("admin")
def admin_route():
    return jsonify({"message": "Welcome, admin!"})

@app.route('/user', methods=['GET'])
@authenticate
@authorize("user")
def user_route():
    return jsonify({"message": "Welcome, user!"})

if __name__ == '__main__':
    app.run(debug=True)
