# Glass Shop

Node.js (Express) + PostgreSQL storefront with a Glassmorphism UI, customer accounts, an admin dashboard, and permanent product image uploads.

## Run locally

1. Install Node 18+ and PostgreSQL, then create a database:
   `createdb glass_shop`
2. `cp .env.example .env` and set `DATABASE_URL` and `JWT_SECRET`
   (generate one: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`).
3. `npm install`
4. `npm run migrate` creates the tables and the admin account from `ADMIN_USERNAME` / `ADMIN_PASSWORD`.
5. `npm start` and open http://localhost:3000. The admin dashboard is at /admin/ after you log in as the admin.

## Layout

- `db/schema.sql`: users, products, orders, order_items
- `src/db.js`: Postgres connection pool
- `src/middleware/auth.js`: JWT cookie auth, `requireAuth`, `requireAdmin`
- `src/routes/`: `auth` (signup, login, logout, me), `products` (public read, admin write + uploads), `orders`
- `public/`: storefront and Glassmorphism CSS (`css/style.css`)
- `admin/`: dashboard, only served to the admin
- `uploads/`: product photos, served at /uploads (keep this folder on a persistent disk or volume when deploying)

## API

| Method and path | Access | Purpose |
|---|---|---|
| POST /api/auth/signup, /login, /logout | public | Accounts (signup always creates a customer) |
| GET /api/auth/me | public | Current user or null |
| GET /api/products, /api/products/:id | public | Browse |
| POST /api/products, PUT /api/products/:id | admin | Create or edit (multipart form, field `image`) |
| PATCH /api/products/:id/sold-out | admin | Toggle sold out |
| DELETE /api/products/:id | admin | Delete product and its image |
| POST /api/orders | customer | Place an order `{items:[{product_id, quantity}]}`; prices come from the database |
| GET /api/orders/mine | customer | Own orders |
| GET /api/orders | admin | Pending orders (`?status=packed` for history) |
| PATCH /api/orders/:id/packed | admin | Mark packed |

## Before you deploy

- Change `ADMIN_PASSWORD` in `.env` to something new before the first `npm run migrate` on a live server. It was shared in plain text, so treat it as public. To change it later, update `users.password_hash` with a new bcrypt hash.
- Set `NODE_ENV=production`, serve over HTTPS (the login cookie is then marked Secure), and set `DATABASE_SSL=true` if your host requires it.
- The storefront does not have a cart yet. The orders API and tables are ready for one.

## Deploy on Render

1. Push this folder to a GitHub repo.
2. Render Dashboard: New > Postgres. Copy the **Internal Database URL**.
3. New > Web Service, connect the repo, then set:
   - Build command: `npm install`
   - Start command: `npm run migrate && npm start` (the migration is safe to repeat and never overwrites the admin password)
4. Environment variables: `DATABASE_URL` (internal URL), `JWT_SECRET`, `NODE_ENV=production`, `ADMIN_USERNAME`, `ADMIN_PASSWORD` (choose a new one), `UPLOAD_DIR`.
   If you use the External Database URL instead, also set `DATABASE_SSL=true`.
5. Product photos need a persistent disk. Render's filesystem is wiped on every deploy and restart, and free web services cannot attach a disk.
   Use a paid web service, add a Disk mounted at `/var/data`, and set `UPLOAD_DIR=/var/data/uploads`.
