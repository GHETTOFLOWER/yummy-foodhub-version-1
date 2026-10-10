# Yummy Food Hub

Yummy Food Hub is a full-stack food ordering app for a local food market. Customers can browse food items, add them to a cart, and place orders. Sellers can list their own dishes, track sales, and manage stock. Staff and admins can review orders, update inventory, and monitor platform activity.

This project is built with:

- Frontend: plain HTML, CSS, and JavaScript in `yummy-web/`
- Backend: Node.js + Express in `yummy-backend/server.js`
- Database: MySQL
- Authentication: session tokens stored in memory on the server

---

## Project structure

```text
yummy-food-hub-android/
├── README.md
├── yummy-backend/
│   ├── package.json
│   ├── server.js
│   └── node_modules/
└── yummy-web/
    ├── app.js
    ├── index.html
    └── styles.css
```

### What each folder does

- `yummy-web/`: the browser app that users interact with
- `yummy-backend/server.js`: the API layer that handles auth, menu, orders, stock, and dashboards
- `yummy-backend/package.json`: backend dependencies and start script

---

## Core app behavior

### 1) Customer experience

Customers can:

- create an account
- sign in as a customer
- browse the menu by category
- search dishes
- add items to a cart
- place an order
- view their own order history

When a customer places an order:

- if a dish has enough stock, the order line is approved immediately
- if the stock is too low, the order item stays pending until a seller, staff member, or admin approves it
- stock is only reduced when an item is approved

### 2) Seller experience

Any user-created dish is treated as a community listing with a seller attached to it. Sellers can:

- publish their own dishes with ingredients and preparation steps
- update the stock of their dishes
- review incoming orders that contain their dishes
- approve or reject pending items for their dishes
- see sales totals and seller earnings

The app records:

- commission amount for the platform/admin from approved user-owned dishes
- seller payout amount after commission

### 3) Staff and admin experience

Staff and admin users can:

- view the full inventory list
- update stock counts for any dish
- review all customer orders
- approve, reject, or complete pending order items
- access dashboards with sales and order summaries

Admin users have additional access to the full platform summary dashboard.

---

## User roles

| Role | Typical purpose |
| --- | --- |
| `customer` | browse, order, and view personal order history |
| `staff` | manage inventory and approve/reject orders |
| `admin` | full oversight of orders, sales, and platform dashboards |

Important note:

- Public registration always creates a `customer` account.
- Staff and admin accounts must be created first as a customer, then promoted manually in MySQL.

---

## Business rules in the app

The backend enforces several rules:

- a user can only view and manage their own orders
- a seller can only manage their own dish lines
- staff/admin can manage any order line
- stock is updated only when an approved item is processed
- order status is recalculated based on all order items as a group
- the app tracks commission and seller earnings separately

### Commission logic

For user-owned dishes that are approved:

- total dish value is calculated
- 5% commission is deducted
- remaining amount is credited to the seller

This is handled in the backend when an order item is approved.

---

## Database concept

The app expects a MySQL database named `yummy_food_hub`.

The main tables used by the backend are:

- `users`
- `menu_items`
- `recipe_ingredients`
- `orders`
- `order_items`

The server uses queries like these:

- user login and registration
- menu listing and ingredients lookup
- order creation and line-level approval logic
- seller dashboards and staff/admin reporting

---

## Local setup

### 1) Install MySQL

Use MySQL or XAMPP locally.

Typical XAMPP settings:

- host: `localhost`
- port: `3307`
- user: `root`
- password: empty by default, unless you set one

### 2) Create the database

Create the database before starting the app:

```sql
CREATE DATABASE yummy_food_hub;
USE yummy_food_hub;
```

### 3) Configure the backend

Edit `yummy-backend/server.js` to match your local database credentials if needed:

```js
const DB_CONFIG = {
  host: 'localhost',
  user: 'root',
  password: '',
  database: 'yummy_food_hub',
  port: 3307,
};
```

### 4) Install and run the server

From the project root:

```powershell
cd yummy-backend
npm install
npm start
```

The server starts on:

```text
http://localhost:3107
```

Check health:

```text
http://localhost:3107/api/health
```

---

## Frontend access

The Express server serves the static frontend from `yummy-web/`, so the browser app is opened through the same origin as the backend.

Open:

```text
http://localhost:3107
```

## Deploy online with Render

The repository includes a `render.yaml` Blueprint for deploying the Express server and browser app together. The frontend uses relative API URLs, and the server reads its listening port and MySQL connection details from environment variables.

1. Create a managed MySQL database with a provider that allows connections from Render. Create the `yummy_food_hub` database and load the required tables and menu data. **This repository does not include a schema or seed script**, so create/import these before relying on the deployed app.
2. Push this repository to GitHub.
3. In Render, create a new Blueprint and select this repository. Review the service settings from `render.yaml`.
4. Enter the database connection values when prompted: `DB_HOST`, `DB_PORT`, `DB_USER`, and `DB_PASSWORD`. Keep credentials in Render's environment settings, not in source files. The database provider may require additional network-access or TLS configuration.
5. Deploy, then open the service URL and verify `/api/health` responds. Use the same URL for the web app.

The app keeps login sessions in server memory. Restarting or redeploying the service signs users out, and running multiple service instances is not supported reliably without moving sessions to shared storage.

---

## Important routes in the API

The backend exposes routes such as:

- `GET /api/health` — checks if the server is responding
- `POST /register` — register a new customer account
- `POST /login` — sign in and get a bearer token
- `POST /logout` — end the session
- `GET /menu` — fetch all menu items with ingredients
- `POST /order` — place an order for one or more dishes
- `GET /orders/:userId` — view a customer’s order history
- `POST /recipes` — publish a community dish as a seller
- `GET /recipes/user/:userId` — view dishes owned by a user
- `GET /inventory` — view stock inventory for staff/admin
- `PUT /inventory/:id` — update a dish’s stock quantity
- `GET /staff/orders` — review the full order queue
- `GET /admin/dashboard` — view admin totals and recent orders
- `GET /seller/orders` — view sales tied to a seller
- `GET /seller/dashboard` — view seller earnings and stats

---

## Promote a user to staff or admin

Create the account through the app first, then change the role in MySQL:

```sql
UPDATE users SET role = 'admin' WHERE email = 'admin@yummy.com';
```

For a staff account:

```sql
UPDATE users SET role = 'staff' WHERE email = 'staff@yummy.com';
```

Then sign out and sign in again, and choose the Staff / Admin mode in the UI.

---

## Typical user flow

1. User registers or logs in.
2. User browses menu items and searches by category.
3. User adds food to cart.
4. User places an order.
5. If stock is available, the item is approved immediately.
6. If stock is low, the item remains pending and requires review.
7. Seller or staff/admin approves or rejects the dish line.
8. Admin dashboard updates totals, commissions, and sales.

---

## Troubleshooting

### Server fails to start

- confirm MySQL is running
- confirm the database exists and the credentials match
- confirm the port is correct
- run `npm install` in `yummy-backend`

### Login or registration fails

- ensure `users` table exists
- verify the `password_hash` column is being populated correctly
- confirm the app is talking to the right MySQL instance

### Menu does not load

- confirm the backend is running at `http://localhost:3107`
- check `http://localhost:3107/api/health`
- confirm your MySQL database has menu data or seed data available

### Access denied

- customer accounts are limited to customer actions
- seller permissions depend on dish ownership
- staff/admin roles must be granted manually in the database

---

## Notes

This project is intentionally lightweight and uses plain frontend code rather than a larger framework. The app logic is centered around a small Express API and database-driven role management, which keeps the project easy to understand and modify.

If you want to extend it, the natural next upgrades are:

- add payment processing
- add image uploads for dishes
- add admin user management
- add full order notifications
- add database seed scripts for sample menu items
