# Yummy Food Hub

A browser-based food ordering app built with plain HTML, CSS, and JavaScript. The Express API serves the frontend and connects it to MySQL.

## Features

- Customer registration and login
- Browse and search menu categories, add items to a cart, and place orders
- Menu priced in Ugandan shillings (UGX), with local favorites including Rolex, Luwombo, and Matoke
- Users can publish searchable recipes with ingredients, preparation steps, prices, and portion availability
- Menu clearly separates user-created Community dishes from staff/admin-created Hub menu dishes
- Available orders are approved automatically; unavailable requests wait for admin review without reserving stock
- View order history and refresh approval statuses
- Sellers can see who ordered their dishes, approve/reject their own pending items, and track net earnings
- Admin dashboard for all orders, approved sales, 5% user-dish commission, and availability
- Staff/admin order queue to review, approve, or reject pending orders
- Staff/admin inventory dashboard with stock updates
- Responsive layouts for desktop and mobile

## Project structure

```
yummy-web/             HTML, CSS, and JavaScript frontend
yummy-backend/         Express REST API and MySQL integration
  server.js
  schema.sql           Database tables, ingredient lists, and sample menu items
```

## Run locally

1. Start MySQL (for example, through XAMPP). Import `yummy-backend/schema.sql` using phpMyAdmin or MySQL Workbench. It creates the `yummy_food_hub` database and UGX-priced sample menu. Existing USD-scale menu and order amounts are converted once at approximately UGX 3,800 per USD.
2. In `yummy-backend/server.js`, set `DB_CONFIG.password` to your MySQL password if needed. XAMPP is configured to use MySQL port `3307`.
3. Install and start the server:

   ```powershell
   cd yummy-backend
   npm install
   npm start
   ```

4. Open [http://localhost:3107](http://localhost:3107). The health endpoint is [http://localhost:3107/api/health](http://localhost:3107/api/health). XAMPP Apache uses `http://localhost:8088` (HTTPS: `https://localhost:4443`), and phpMyAdmin is at `http://localhost:8088/phpmyadmin`.

Create a customer account in the app to browse and place orders. To create an admin or staff account, register that account as a customer first, then promote it from phpMyAdmin's SQL tab. Replace the email with the account you registered:

```sql
UPDATE users SET role = 'admin' WHERE email = 'admin@yummy.com';
```

For staff access, use `role = 'staff'` instead. Public registration only creates customer accounts. Sign out and back in after changing a role, then choose **Staff / Admin** to access orders and inventory.

Seller earnings and the admin's 5% commission are recorded when a user-owned dish is approved. The app tracks these balances but does not transfer money; connect a payment provider to enable payouts.

## Troubleshooting

- If registration or login fails, check that MySQL is running and `DB_CONFIG` matches the local database credentials.
- If the menu cannot load, confirm `npm start` is still running and visit `/api/health` to verify the server is reachable.
- PowerShell may block `npm` scripts on some Windows setups. Use Command Prompt or enable script execution for the current user if your environment permits it.
