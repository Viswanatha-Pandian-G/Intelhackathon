Web Framework:
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Budget Management Tool</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>

<body class="bg-gray-100">
    <div class="flex flex-col items-center justify-center min-h-screen">
        <div class="bg-white p-8 rounded-lg shadow-lg max-w-md w-full">
            <h2 class="text-2xl font-semibold text-gray-800 mb-4">
                Budget Management Tool
            </h2>
            <form id="budgetForm">
                <label for="income" class="text-gray-700">Income:</label>
                <input type="number" id="income" name="income" class="w-full border border-green-500 rounded-lg p-2 mb-4" required>
                
                <label for="expenseAmount" class="text-gray-700">Expense Amount:</label>
                <input type="number" id="expenseAmount" name="expenseAmount" class="w-full border border-green-500 rounded-lg p-2 mb-4" min="0" required>
                
                <label for="expenseCategory" class="text-gray-700">Expense Category:</label>
                <select id="expenseCategory" name="expenseCategory" class="w-full border border-green-500 rounded-lg p-2 mb-4">
                    <option value="Groceries">Groceries</option>
                    <option value="Utilities">Utilities</option>
                    <option value="Transportation">Transportation</option>
                    <option value="Entertainment">Entertainment</option>
                    <option value="Other">Other</option>
                </select>
                
                <button type="button" id="addExpense" class="bg-blue-500 hover:bg-blue-600 text-white rounded-lg px-4 py-2 w-full mb-4">Add Expense</button>
                
                <div id="expenseList" class="mb-4"></div>
                
                <div class="flex justify-between items-center mb-4">
                    <span class="text-gray-700">Total Expenses:</span>
                    <span id="totalExpenses" class="text-green-600 font-semibold">$0.00</span>
                </div>
                
                <button type="submit" class="bg-green-500 hover:bg-green-600 text-white rounded-lg px-4 py-2 w-full">Calculate Savings</button>
            </form>
            <div id="savings" class="mt-4 text-gray-700"></div>
        </div>
    </div>

    <script>
        let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
        document.getElementById('income').value = localStorage.getItem('income') || '';

        document.getElementById('addExpense').addEventListener('click', function () {
            const expenseAmount = parseFloat(document.getElementById('expenseAmount').value);
            const expenseCategory = document.getElementById('expenseCategory').value;

            if (!isNaN(expenseAmount) && expenseAmount > 0) {
                expenses.push({ amount: expenseAmount, category: expenseCategory });
                localStorage.setItem('expenses', JSON.stringify(expenses));
                renderExpenseList();
            } else {
                alert('Please enter a valid expense amount.');
            }
        });

        function renderExpenseList() {
            const expenseListElement = document.getElementById('expenseList');
            expenseListElement.innerHTML = '';
            let totalExpenses = 0;

            expenses.forEach((expense, index) => {
                const expenseItem = document.createElement('div');
                expenseItem.className = "flex justify-between items-center mb-2";

                expenseItem.innerHTML = `
                    <span>${expense.category}: $${expense.amount.toFixed(2)}</span>
                    <div>
                        <button onclick="editExpense(${index})" class="bg-yellow-500 hover:bg-yellow-600 text-white px-2 py-1 rounded-lg">Edit</button>
                        <button onclick="removeExpense(${index})" class="bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded-lg">Remove</button>
                    </div>
                `;

                expenseListElement.appendChild(expenseItem);
                totalExpenses += expense.amount;
            });

            document.getElementById('totalExpenses').textContent = `$${totalExpenses.toFixed(2)}`;
        }

        function removeExpense(index) {
            expenses.splice(index, 1);
            localStorage.setItem('expenses', JSON.stringify(expenses));
            renderExpenseList();
        }

        function editExpense(index) {
            const newAmount = prompt("Enter new amount:", expenses[index].amount);
            const newCategory = prompt("Enter new category:", expenses[index].category);
            if (newAmount && newCategory && !isNaN(parseFloat(newAmount))) {
                expenses[index] = { amount: parseFloat(newAmount), category: newCategory };
                localStorage.setItem('expenses', JSON.stringify(expenses));
                renderExpenseList();
            } else {
                alert("Invalid input.");
            }
        }

        document.getElementById('budgetForm').addEventListener('submit', function (event) {
            event.preventDefault();
            const income = parseFloat(document.getElementById('income').value);
            const totalExpenses = expenses.reduce((acc, expense) => acc + expense.amount, 0);

            if (!isNaN(income) && income >= 0) {
                localStorage.setItem('income', income);
                const savings = income - totalExpenses;
                document.getElementById('savings').textContent = `Savings: $${savings.toFixed(2)}`;
            } else {
                document.getElementById('savings').textContent = "Please enter a valid income amount.";
            }
        });

        

        window.onload = renderExpenseList;
    </script>
</body>

</html>


python framework for GenAI:

from flask import Flask, render_template, request, jsonify
import numpy as np
from sklearn.linear_model import LinearRegression

app = Flask(__name__)

# Dummy historical data
historical_data = {
    "income": [3000, 3200, 3100, 2900, 3500],
    "expenses": [1200, 1300, 1250, 1100, 1400]
}

# Predict future expenses using Linear Regression
def predict_future_budget(income, total_expenses):
    X = np.array(historical_data['expenses']).reshape(-1, 1)
    y = np.array(historical_data['income'])

    # Train a simple linear regression model
    model = LinearRegression()
    model.fit(X, y)

    # Predict future expenses based on current total expenses
    future_expense = model.predict(np.array([[total_expenses]]))[0]
    future_savings = income - future_expense

    return future_expense, future_savings

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/calculate', methods=['POST'])
def calculate():
    income = float(request.form['income'])
    expenses = [float(e['amount']) for e in request.json['expenses']]

    total_expenses = sum(expenses)
    savings = income - total_expenses

    # Predict future expenses and savings
    future_expense, future_savings = predict_future_budget(income, total_expenses)

    return jsonify({
        'savings': savings,
        'future_expense': future_expense,
        'future_savings': future_savings
    })

if __name__ == "__main__":
    app.run(debug=True)
