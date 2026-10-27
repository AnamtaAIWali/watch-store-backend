# Perfect Time - Backend (Node.js + Express + MongoDB)

REST API for the Perfect Time watch store. Designed for **Vercel serverless** (cached MongoDB connection, `api/index.js` entry, `vercel.json`).

## 1. Accounts you need
1. **MongoDB Atlas** - cluster + database user. In *Network Access* add `0.0.0.0/0` (Vercel uses changing IPs).
2. **Cloudinary** (free) - permanent image storage. From the dashboard copy *Cloud name, API Key, API Secret*.
3. **Vercel** account.

## 2. Environment variables
Copy `.env.example` to `.env` for local use. On Vercel add the same names in *Project > Settings > Environment Variables*:

| Name | Meaning |
|---|---|
| `MONGODB_URI` | Atlas connection string |
| `MONGODB_DB` | Database name (default `perfecttime`) |
| `JWT_SECRET` | Long random string (e.g. 40+ characters) |
| `ADMIN_EMAIL` / `ADMIN_PASSWORD` | Your admin login. The account is created automatically (password is hashed) the first time you log in. Change the password later in Admin > Website Settings. |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | Image storage |
| `FRONTEND_URL` | Your frontend URL(s), comma separated, no trailing slash. Add it after the frontend is deployed, then redeploy the backend. |
| `WHATSAPP_NUMBER` | Reference only; the live number is edited in Admin > Website Settings |

**Never commit `.env` and never paste secrets into code.**

## 3. Deploy to Vercel (backend first)
1. Unzip, push the `backend` folder to a GitHub repo (or use `npx vercel` in this folder).
2. Vercel > *Add New Project* > import it. Framework preset: **Other**. No build command needed.
3. Add all environment variables above, then **Deploy**.
4. Open `https://YOUR-BACKEND.vercel.app/` - you should see `{"ok":true,"service":"Perfect Time API"}`.
5. Open `https://YOUR-BACKEND.vercel.app/api/home` - should return JSON (this also creates the default settings and 6 category slots in MongoDB).
6. Copy the backend URL for the frontend.

Run locally: `npm install && npm run dev` (needs `.env`).

Notes: Vercel limits a request body to ~4.5 MB, so each uploaded image is limited to 4 MB (the API enforces this).

## 4. API reference
Public (JSON): 
- `GET /api/home` - settings, slides, categories, best selling, men, women
- `GET /api/settings`, `GET /api/categories`, `GET /api/categories/:slug`
- `GET /api/products?category=<slug>&section=men|women&q=<name>&best=true`
- `GET /api/products/by-ids?ids=a,b`, `GET /api/products/:slug`
- `POST /api/orders` `{customer, items:[{productId,quantity}], notes, idempotencyKey}` - prices/stock/delivery are recalculated on the server
- `GET /api/orders/:orderNumber?token=...` - confirmation data (token is returned at checkout)
- `GET /sitemap.xml`, `GET /robots.txt`

Admin (header `Authorization: Bearer <token>`):
- `POST /api/admin/login`, `GET /api/admin/me`, `PUT /api/admin/password`
- `POST /api/admin/upload?folder=products|categories|slider|branding` (multipart field `images`), `POST /api/admin/upload/delete`
- `GET /api/admin/dashboard`
- Products: `GET/POST /api/admin/products`, `GET/PUT/DELETE /api/admin/products/:id`, `PATCH .../:id/stock`, `PATCH .../:id/bestselling`, `PUT /api/admin/products/bestselling/order`
- Categories: `GET/POST /api/admin/categories`, `PUT/DELETE .../:id`, `PUT .../order`
- Slides: `GET/POST /api/admin/slides`, `PUT/DELETE .../:id`, `PUT .../order`
- Orders: `GET /api/admin/orders?status=&q=`, `GET .../by-ids?ids=`, `GET .../:id`, `PATCH .../:id/status`, `DELETE .../:id`
- `PUT /api/admin/settings`

## 5. Data model (MongoDB collections)
`products`, `categories`, `orders`, `admins`, `settings`, `slides`, `counters` (order numbers start at #1001).
