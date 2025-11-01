<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Smart Expense Tracker</title>

  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    body {
      font-family: "Poppins", sans-serif;
      background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
      color: #f0f0f0;
      margin: 0;
      padding: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    h1, h2, h3 {
      text-align: center;
    }

    .login-container, .tracker-container {
      background: rgba(255, 255, 255, 0.1);
      padding: 20px 30px;
      border-radius: 15px;
      margin-top: 50px;
      box-shadow: 0 0 10px rgba(0,0,0,0.4);
      width: 90%;
      max-width: 500px;
      backdrop-filter: blur(6px);
    }

    input, select, button {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border: none;
      border-radius: 8px;
      font-size: 16px;
    }

    input, select {
      background-color: #1c2e3b;
      color: white;
    }

    button {
      background-color: #00bcd4;
      color: white;
      cursor: pointer;
      font-weight: bold;
      transition: 0.3s;
    }

    button:hover {
      background-color: #0097a7;
    }

    table {
      width: 100%;
      margin-top: 15px;
      border-collapse: collapse;
    }

    th, td {
      border-bottom: 1px solid rgba(255, 255, 255, 0.2);
      text-align: left;
      padding: 8px;
    }

    canvas {
      background: rgba(255,255,255,0.05);
      border-radius: 10px;
      padding: 10px;
      margin-top: 20px;
    }

    .logout-btn {
      background-color: #e91e63;
      margin-top: 15px;
    }
  </style>
</head>

<body>
  <h1>💰 Smart Expense Tracker</h1>

  <!-- LOGIN PAGE -->
  <div class="login-container" id="login-page">
    <h2>Login</h2>
    <input type="text" id="username" placeholder="Enter your name" />
    <button onclick="login()">Login</button>
  </div>

  <!-- MAIN TRACKER PAGE -->
  <div class="tracker-container" id="tracker-page" style="display:none;">
    <h2 id="welcome"></h2>

    <h3>Add Expense</h3>
    <input type="number" id="amount" placeholder="Enter amount" />
    <input type="text" id="desc" placeholder="Enter description" />
    <select id="category">
      <option value="Food">🍔 Food</option>
      <option value="Transport">🚌 Transport</option>
      <option value="Bills">💡 Bills</option>
      <option value="Entertainment">🎮 Entertainment</option>
      <option value="Study">📚 Study</option>
      <option value="Others">🛍 Others</option>
    </select>
    <button onclick="addExpense()">Add Expense</button>

    <table id="expense-table">
      <thead>
        <tr>
          <th>Amount</th>
          <th>Category</th>
          <th>Description</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>

    <h3>Expense Charts</h3>
    <canvas id="pieChart" height="200"></canvas>
    <canvas id="barChart" height="200"></canvas>

    <button class="logout-btn" onclick="logout()">Logout</button>
  </div>

  <script>
    let expenses = JSON.parse(localStorage.getItem("expenses")) || [];
    let currentUser = localStorage.getItem("user") || "";

    // LOGIN
    function login() {
      const username = document.getElementById("username").value.trim();
      if (username === "") return alert("Please enter your name.");
      localStorage.setItem("user", username);
      currentUser = username;
      showTracker();
    }

    function logout() {
      localStorage.removeItem("user");
      document.getElementById("tracker-page").style.display = "none";
      document.getElementById("login-page").style.display = "block";
    }

    function showTracker() {
      document.getElementById("login-page").style.display = "none";
      document.getElementById("tracker-page").style.display = "block";
      document.getElementById("welcome").textContent = `Welcome, ${currentUser}!`;
      renderTable();
      renderCharts();
    }

    // ADD EXPENSE
    function addExpense() {
      const amount = parseFloat(document.getElementById("amount").value);
      const desc = document.getElementById("desc").value.trim();
      const category = document.getElementById("category").value;

      if (isNaN(amount) || amount <= 0 || desc === "") {
        alert("Please enter valid details!");
        return;
      }

      expenses.push({ amount, category, desc, user: currentUser });
      localStorage.setItem("expenses", JSON.stringify(expenses));
      renderTable();
      renderCharts();

      document.getElementById("amount").value = "";
      document.getElementById("desc").value = "";
    }

    // RENDER TABLE
    function renderTable() {
      const tbody = document.querySelector("#expense-table tbody");
      tbody.innerHTML = "";

      const userExpenses = expenses.filter(e => e.user === currentUser);

      userExpenses.forEach(exp => {
        const row = `<tr><td>₹${exp.amount}</td><td>${exp.category}</td><td>${exp.desc}</td></tr>`;
        tbody.insertAdjacentHTML("beforeend", row);
      });
    }

    // RENDER CHARTS
    function renderCharts() {
      const userExpenses = expenses.filter(e => e.user === currentUser);
      const categoryTotals = {};

      userExpenses.forEach(exp => {
        categoryTotals[exp.category] = (categoryTotals[exp.category] || 0) + exp.amount;
      });

      const categories = Object.keys(categoryTotals);
      const values = Object.values(categoryTotals);

      // PIE CHART
      const ctx1 = document.getElementById("pieChart").getContext("2d");
      new Chart(ctx1, {
        type: "pie",
        data: {
          labels: categories,
          datasets: [{
            data: values,
            backgroundColor: ["#f44336","#2196f3","#4caf50","#ff9800","#9c27b0","#00bcd4"],
          }]
        },
        options: { plugins: { legend: { labels: { color: 'white' } } } }
      });

      // BAR CHART
      const ctx2 = document.getElementById("barChart").getContext("2d");
      new Chart(ctx2, {
        type: "bar",
        data: {
          labels: categories,
          datasets: [{
            label: "Expense Breakdown (₹)",
            data: values,
            backgroundColor: "#03a9f4"
          }]
        },
        options: {
          scales: {
            x: { ticks: { color: 'white' } },
            y: { ticks: { color: 'white' } }
          },
          plugins: { legend: { labels: { color: 'white' } } }
        }
      });
    }

    // AUTO LOGIN
    if (currentUser) showTracker();
  </script>
</body>
</html>
# dailyexpense-tracker.in
