# Full Stack AI E-Commerce Website

An e-commerce project with a React storefront, a React admin dashboard, and a Node.js/Express API backed by PostgreSQL. The backend includes product search with Google Gemini, image uploads through Cloudinary, user authentication, orders, reviews, and a Stripe payment flow. **This repository is a work in progress:** several dashboard components and integrations need completion before the whole application works end to end.

## Tech stack

| Area | Technologies |
| --- | --- |
| Storefront and dashboard | React 19, Vite, Redux Toolkit, React Router, Tailwind CSS |
| Backend | Node.js, Express, PostgreSQL (`pg`), JWT, bcrypt |
| Integrations | Gemini API, Cloudinary, Stripe, Nodemailer |

## Project layout

The downloaded repository has one extra enclosing directory inside each app:

```text
FULL-STACK-ECOMMERCE-AI-BASED-WEB-APPLICATION-BACKEND-CODE-main/
  FULL-STACK-ECOMMERCE-AI-BASED-WEB-APPLICATION-BACKEND-CODE-main/  # API
ecommerce-frontend-template-main/
  ecommerce-frontend-template-main/                              # Storefront
ecommerce-dashboard-template-main/
  ecommerce-dashboard-template-main/                              # Admin dashboard
```

Run `npm` commands from the **inner directory** containing the relevant `package.json`.

## Features represented in the code

- Customer registration, login, profile updates and password reset using JWT cookies and email.
- Product listing and details, admin product management, product reviews and image uploads.
- Cart and order pages in the storefront; order creation and management API routes.
- AI assisted product filtering via the authenticated `POST /api/v1/product/ai-search` endpoint, which calls Gemini.
- Stripe payment intent creation and a payment webhook intended to update payment status and product stock.
- Admin routes for users, order management and dashboard statistics.

Some of these features are incomplete or unverified; see [Current limitations](#current-limitations).

## Local setup

Install Node.js (a modern version with built-in `fetch`, preferably Node 20+), npm, and PostgreSQL. Create a PostgreSQL database named `mern_ecommerce_store`, or change the backend database connection to match your database.

1. In the **inner backend directory**, run `npm install`. Create `config/config.env` locally with your own values:

   ```dotenv
   PORT=4000
   FRONTEND_URL=http://localhost:5173
   DASHBOARD_URL=http://localhost:5174

   JWT_SECRET_KEY=replace-with-a-long-random-secret
   JWT_EXPIRES_IN=30d
   COOKIE_EXPIRES_IN=30

   DB_USER=postgres
   DB_HOST=localhost
   DB_NAME=mern_ecommerce_store
   DB_PASSWORD=replace-with-your-database-password
   DB_PORT=5432

   GEMINI_API_KEY=replace-with-your-key
   CLOUDINARY_CLIENT_NAME=replace-with-your-cloud-name
   CLOUDINARY_CLIENT_API=replace-with-your-api-key
   CLOUDINARY_CLIENT_SECRET=replace-with-your-api-secret

   SMTP_SERVICE=gmail
   SMTP_HOST=smtp.gmail.com
   SMTP_PORT=465
   SMTP_MAIL=replace-with-your-email
   SMTP_PASSWORD=replace-with-your-app-password

   STRIPE_SECRET_KEY=replace-with-your-test-secret-key
   STRIPE_WEBHOOK_SECRET=replace-with-your-webhook-secret
   ```

   **Before starting the API, update `database/db.js`**: this version hard codes its PostgreSQL host, user, database, password and port. Replace those values with the corresponding `process.env.DB_*` values (and ensure dotenv is loaded before the database module is imported). Also replace the placeholder Stripe key in `utils/generatePaymentIntent.js` with `process.env.STRIPE_SECRET_KEY`. Setting these variables alone will not fix those two files in the current source.

2. Start the backend from that directory with `npm start` (`npm run dev` requires `nodemon`, which is not declared in this package). The backend listens on `http://localhost:4000`; `app.js` attempts to create its tables at startup.

3. In the **inner storefront directory**, run `npm install` then `npm run dev -- --port 5173`. Open `http://localhost:5173`. The storefront's development Axios configuration points to `http://localhost:4000/api/v1`.

4. In the **inner dashboard directory**, run `npm install` then `npm run dev -- --port 5174`. Open `http://localhost:5174`. Its app currently needs the fixes listed below before its protected dashboard route can render correctly.

Keep all three commands running in separate terminals. The Vite ports must match the backend's CORS origins. For payment testing, configure a Stripe test webhook to forward to `http://localhost:4000/api/v1/payment/webhook`.

## API routes

All routes below have the prefix `/api/v1`. Protected routes require a login cookie; admin routes also require the `Admin` role.

| Area | Routes |
| --- | --- |
| Auth | `POST /auth/register`, `POST /auth/login`, `GET /auth/me`, `GET /auth/logout`, `POST /auth/password/forgot`, `PUT /auth/password/reset/:token`, `PUT /auth/password/update`, `PUT /auth/profile/update` |
| Products | `GET /product`, `GET /product/singleProduct/:productId`, `PUT /product/post-new/review/:productId`, `DELETE /product/delete/review/:productId`, `POST /product/ai-search` |
| Admin products | `POST /product/admin/create`, `PUT /product/admin/update/:productId`, `DELETE /product/admin/delete/:productId` |
| Orders | `POST /order/new`, `GET /order/:orderId`, `GET /order/orders/me`, `GET /order/admin/getall`, `PUT /order/admin/update/:orderId`, `DELETE /order/admin/delete/:orderId` |
| Admin | `GET /admin/getallusers`, `DELETE /admin/delete/:id`, `GET /admin/fetch/dashboard-stats` |
| Payment | `POST /payment/webhook` (Stripe webhook) |

## Current limitations

- `database/db.js` contains a hard-coded local password and ignores the `DB_*` variables. Remove the password from the repository and rotate it if it was real and published.
- `utils/generatePaymentIntent.js` contains `PASTE_YOUR_STRIPE_SECRET_KEY`; payments cannot work until it is configured. The webhook implementation should be checked against Stripe test events before accepting real payments.
- The dashboard's `src/App.jsx` references `isAuthenticated`, `user`, and `renderDashboardContent` without definitions; `src/components/SideBar.jsx` currently renders an empty fragment. Several dashboard slices/components are placeholders, so the dashboard is not a finished admin interface.
- No working automated test suite is configured: the backend `npm test` script is a placeholder. The application has not been verified here with live PostgreSQL, Gemini, Cloudinary, SMTP or Stripe credentials.
- The storefront's production Axios base URL is `/`; production hosting needs API routing or a configurable base URL.

## Security

Never commit `config/config.env`, `.env`, API keys, database passwords, or `node_modules`. Add root level ignore rules that cover the **nested** backend config file, such as `**/config/config.env`, `**/.env`, `**/node_modules/`, and `**/uploads/`. The supplied archive contains a backend `config/config.env` file: if this repository has already been pushed publicly, removing the file from the latest commit does not erase earlier exposure. Rotate every real credential it contained and remove sensitive history if needed.
