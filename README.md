# eWay Marketplace DBMS

A MySQL-based e-commerce application with CLI interface, supporting user management, product listings, transactions, and shopping features.

## Overview
This project is a command-line interface (CLI) e-commerce platform that allows users to buy and sell products, manage their accounts, and track transactions. Built with Python and MySQL, it demonstrates fundamental database operations, transaction management, and user authentication.

## Features
### User Management
- Account creation and authentication
- Profile viewing and editing
- Wallet balance management

### Product Management
- Browse products by category
- Filter products based on various criteria
- Add new products for sale
- Update product information

### Transaction System
- Purchase products securely
- View transaction history
- Automatic inventory management
- Wallet balance updates

### Prime Membership
- Premium user status upgrade
- Special benefits for high-spending customers

## Database Structure
The database consists of the following tables:
- **User**: Stores user account information
- **Category**: Classifies products into categories
- **Product**: Contains product listings and details
- **Payments**: Records all transaction data
- **Sell_Buy_Mapping**: Maps transaction types (buy/sell)

## Installation
### Prerequisites
- Python 3.x
- MySQL Server
- `mysql-connector-python` package

### Setup
Clone the repository:

```bash
git clone https://github.com/yourusername/e-commerce-platform.git
cd e-commerce-platform
```

Install required Python packages:

```bash
pip install mysql-connector-python
```

Set up the database:

```bash
mysql -u username -p < setup.sql
```

Update database connection details in `DBMS_Code.py`:

```python
cnx = mysql.connector.connect(
    user="your_username", 
    password="your_password", 
    database="DBMS_SQL"
)
```

## Usage
Run the application:

```bash
python DBMS_Code.py
```

### Main Menu Options
- **List all available products**: View all products with their details
- **User Signup/Login**: Create a new account or log in to an existing one
- **Exit**: Close the application

### User Menu Options
After login, users can:
- View available products
- View previous transactions
- View account balance
- Buy Product
- Put product up for sale
- Upgrade customer status
- View account details
- Update account details
- Add money to wallet
- Logout

## Transaction Examples
### Conflicting Transactions
The system prevents common transaction conflicts:

#### Negative Wallet Balance
```sql
-- Attempt to purchase when balance is insufficient
START TRANSACTION;
SELECT wallet INTO @current_balance FROM User WHERE user_id = 4;
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id) 
VALUES ('2024-04-20', 49999, 0, 4, 1);
COMMIT;
```

#### Price Change During Transaction
```sql
-- Attempt to purchase at old price after price update
START TRANSACTION;
UPDATE Product SET price = 50000 WHERE product_id = 1;
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id)
VALUES ('2024-04-20', 49999, 0, 1, 1);
COMMIT;
```

#### Out of Stock Scenario
```sql
-- Attempt to purchase same product multiple times until stock is depleted
START TRANSACTION;
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id)
VALUES ('2024-04-20', 49999, 0, 1, 1);
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id)
VALUES ('2024-04-20', 49999, 0, 1, 1);
COMMIT;
```

### Non-Conflicting Transactions
Examples of successful concurrent transactions:

#### Different users' independent actions
```sql
START TRANSACTION;
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id)
VALUES ('2024-04-20', 18999, 0, 1, 7);
UPDATE User SET wallet = wallet + 5000 WHERE user_id = 5;
COMMIT;
```

#### Complete seller/buyer transaction
```sql
-- User lists a product, another buys it, seller receives payment
INSERT INTO Product (name, price, description, quantity, avl_status, seller_id, category_id, user_id)
VALUES ('Pixel 6a', '24999', 'Best of Google', '3', true, 19, '1', 1);
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id)
VALUES ('2024-04-20', 24999, 0, 6, 14);
INSERT INTO Payments (Payment_Date, amount, sellbuy, user_id, product_id)
VALUES ('2024-04-20', 24999, 1, 1, 14);
```

## Database Triggers
The system uses triggers to maintain data integrity:
- `update_wallet_after_payment`: Updates user wallet after a payment
- `prevent_negative_balance`: Prevents transactions that would cause negative balance
- `handle_price_change`: Ensures payment amount matches product price
- `decrement_product_quantity`: Reduces product quantity after purchase
- `check_product_availability`: Prevents purchase of out-of-stock items

## Sample Queries
### Products per category
```sql
SELECT c.category_id, c.fname AS category_name, COUNT(p.product_id) AS product_count
FROM Category c
LEFT JOIN Product p ON c.category_id = p.category_id
GROUP BY c.category_id, c.fname;
```

### Most valuable customers
```sql
SELECT u.user_id, u.first_name, u.last_name, SUM(p.amount) AS total_payments
FROM User u
LEFT JOIN Payments p ON u.user_id = p.user_id
GROUP BY u.user_id, u.first_name, u.last_name
ORDER BY total_payments DESC;
```

### Products that need restocking
```sql
SELECT * FROM Product WHERE quantity < 10
ORDER BY quantity;
```
