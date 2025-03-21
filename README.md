# my-project-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Online Banking System</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; }
        input, button { margin: 10px; padding: 10px; }
        #dashboard, #transaction-history { display: none; }
    </style>
</head>
<body>
    <h1>WELCOME TO SA-EED IBN UMAR BANKING SOFTWARE</h1>
    <h2>Banking System</h2>

    <!-- Authentication Section -->
    <div id="auth">
        <h2>Register</h2>
        <input type="text" id="reg-username" placeholder="Username">
        <input type="password" id="reg-password" placeholder="Password">
        <button onclick="register()">Register</button>

        <h2>Login</h2>
        <input type="text" id="login-username" placeholder="Username">
        <input type="password" id="login-password" placeholder="Password">
        <button onclick="login()">Login</button>
    </div>

    <!-- Dashboard -->
    <div id="dashboard">
        <h2>Dashboard</h2>
        <p id="balance">Balance: $0.00</p>

        <h3>Deposit</h3>
        <input type="number" id="deposit-amount" placeholder="Amount">
        <button onclick="deposit()">Deposit</button>

        <h3>Withdraw</h3>
        <input type="number" id="withdraw-amount" placeholder="Amount">
        <button onclick="withdraw()">Withdraw</button>

        <h3>Transfer</h3>
        <input type="text" id="transfer-recipient" placeholder="Recipient Username">
        <input type="number" id="transfer-amount" placeholder="Amount">
        <button onclick="transfer()">Transfer</button>

        <button onclick="getTransactionHistory()">View Transactions</button>
        <button onclick="logout()">Logout</button>
    </div>

    <!-- Transaction History -->
    <div id="transaction-history">
        <h2>Transaction History</h2>
        <table border="1">
            <tr><th>Type</th><th>Amount</th><th>Date</th></tr>
            <tbody id="transactions"></tbody>
        </table>
    </div>

    <script>
        let token = "";

        function register() {
            const username = document.getElementById("reg-username").value;
            const password = document.getElementById("reg-password").value;

            if (!username || !password) {
                alert("Please enter a username and password.");
                return;
            }

            fetch("http://127.0.0.1:5000/register", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ username, password })
            })
            .then(res => res.json())
            .then(data => {
                alert(data.message || data.error);
            })
            .catch(err => console.error("Error:", err));
        }

        function login() {
            const username = document.getElementById("login-username").value;
            const password = document.getElementById("login-password").value;

            if (!username || !password) {
                alert("Please enter both username and password.");
                return;
            }

            fetch("http://127.0.0.1:5000/login", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ username, password })
            })
            .then(res => res.json())
            .then(data => {
                if (data.token) {
                    token = `Bearer ${data.token}`;
                    document.getElementById("auth").style.display = "none";
                    document.getElementById("dashboard").style.display = "block";
                    getBalance();
                } else {
                    alert(data.error || "Login failed.");
                }
            })
            .catch(err => console.error("Error:", err));
        }

        function getBalance() {
            fetch("http://127.0.0.1:5000/balance", {
                method: "GET",
                headers: { "Authorization": token }
            })
            .then(res => res.json())
            .then(data => {
                if (data.balance !== undefined) {
                    document.getElementById("balance").innerText = `Balance: $${data.balance}`;
                } else {
                    alert(data.error || "Failed to retrieve balance.");
                }
            })
            .catch(err => console.error("Error:", err));
        }

        function deposit() {
            const amount = document.getElementById("deposit-amount").value;

            if (!amount || amount <= 0) {
                alert("Please enter a valid deposit amount.");
                return;
            }

            fetch("http://127.0.0.1:5000/deposit", {
                method: "POST",
                headers: { "Content-Type": "application/json", "Authorization": token },
                body: JSON.stringify({ amount: parseFloat(amount) })
            })
            .then(res => res.json())
            .then(data => {
                alert(data.message || data.error);
                getBalance();
            })
            .catch(err => console.error("Error:", err));
        }

        function withdraw() {
            const amount = document.getElementById("withdraw-amount").value;

            if (!amount || amount <= 0) {
                alert("Please enter a valid withdrawal amount.");
                return;
            }

            fetch("http://127.0.0.1:5000/withdraw", {
                method: "POST",
                headers: { "Content-Type": "application/json", "Authorization": token },
                body: JSON.stringify({ amount: parseFloat(amount) })
            })
            .then(res => res.json())
            .then(data => {
                alert(data.message || data.error);
                getBalance();
            })
            .catch(err => console.error("Error:", err));
        }

        function transfer() {
            const recipient = document.getElementById("transfer-recipient").value;
            const amount = document.getElementById("transfer-amount").value;

            if (!recipient || !amount || amount <= 0) {
                alert("Please enter a valid recipient and transfer amount.");
                return;
            }

            fetch("http://127.0.0.1:5000/transfer", {
                method: "POST",
                headers: { "Content-Type": "application/json", "Authorization": token },
                body: JSON.stringify({ recipient, amount: parseFloat(amount) })
            })
            .then(res => res.json())
            .then(data => {
                alert(data.message || data.error);
                getBalance();
            })
            .catch(err => console.error("Error:", err));
        }

        function getTransactionHistory() {
            fetch("http://127.0.0.1:5000/transactions", {
                method: "GET",
                headers: { "Authorization": token }
            })
            .then(res => res.json())
            .then(data => {
                document.getElementById("transactions").innerHTML = "";
                data.transactions.forEach(tran => {
                    document.getElementById("transactions").innerHTML += `<tr><td>${tran.type}</td><td>$${tran.amount}</td><td>${tran.date}</td></tr>`;
                });
            })
            .catch(err => console.error("Error:", err));
        }

        function logout() {
            token = "";
            document.getElementById("auth").style.display = "block";
            document.getElementById("dashboard").style.display = "none";
        }
    </script>
</body>
</html>
