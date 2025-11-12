# Hangry

Hangry is a multi-channel food ordering platform that ships with a customer web app, an operations/admin portal, a lightweight user-management console, a React Native (Expo) client, and an Express/MongoDB backend. It supports menu curation, carts, COD and Stripe checkout, and near-real-time order tracking.

## Repository Layout

| Path          | Description                                                     | Primary Tech                        |
| ------------- | --------------------------------------------------------------- | ----------------------------------- |
| `backend/`    | REST API, auth, menus, carts, orders, payments, file uploads    | Node.js, Express, MongoDB, Stripe   |
| `frontend/`   | Customer-facing Vite + React storefront                         | React 18, Vite, Axios               |
| `admin/`      | Operations dashboard for menus & orders                         | React 18, React Router, Toastify    |
| `users-admin/`| Lightweight console to search/update/remove platform users      | React 18, React Router              |
| `mobile/`     | Expo app with tabs for browse/cart/orders/profile               | Expo Router, React Native, Zustand  |
| `uploads/`    | Local image bucket served at `/images/*` when Cloudinary is off | Express static assets               |

## Feature Highlights

- **Unified backend (`backend/server.js`)** with JWT auth, cart persistence, menu CRUD, Stripe checkout (`/api/order/place`) and COD fallback (`/api/order/place-cod`), plus optional Cloudinary uploads.
- **Web storefront (`frontend/src`)** for browsing, filtering, adding to cart, authentication (`LoginPopup`), placing orders, and verifying payments via `/verify`.
- **Admin portal (`admin/src/pages`)** to add new dishes (with image validation), review menus, and manage fulfillment state on each order.
- **User-management console (`users-admin/src/pages/Users.jsx`)** for role toggling, activation/suspension, and deletion via search + filters.
- **Mobile client (`mobile/app/(tabs)`)** with native cart interactions, order placement, and profile/login flows backed by the same API (`mobile/src/api/client.ts`).
- **Media handling** via local `uploads/` or Cloudinary, exposed through `/images/<filename>` so all clients render menu photos consistently.

## Tech Stack

- **Backend:** Node.js 20+, Express 4, Mongoose 8, JWT, bcrypt, Multer, Stripe SDK, optional Cloudinary.
- **Web Clients:** React 18 + Vite 5/7, React Router 6, Axios, React Icons, React Toastify.
- **Mobile:** Expo SDK 54, React Native 0.81, Expo Router, React Query, Zustand, AsyncStorage.
- **Tooling:** Nodemon, ESLint, Vite, Expo CLI.

## Prerequisites

- Node.js >= 20 and npm >= 10.
- MongoDB Atlas or local Mongo instance.
- Stripe test keys (secret + publishable) for card checkout.
- (Optional) Cloudinary account for CDN-based image storage.
- Expo CLI & a simulator/device for the mobile app.

## Getting Started

1. **Clone the repo**

   ```bash
   git clone https://github.com/TNTDanh/Software_Hangry.git
   cd Software_Hangry
Install dependencies (per workspace)

bash

cd backend && npm install
cd ../frontend && npm install
cd ../admin && npm install
cd ../users-admin && npm install
cd ../mobile && npm install
Environment Variables
Create the following files before running any services.

backend/.env

PORT=4000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/hangry
JWT_SECRET=<random-long-secret>
CLIENT_URL=http://localhost:5173,http://localhost:4173,http://localhost:5174
FRONTEND_URL=http://localhost:5173
STRIPE_SECRET_KEY=sk_test_xxx
CLOUDINARY_CLOUD_NAME=<optional>
CLOUDINARY_API_KEY=<optional>
CLOUDINARY_API_SECRET=<optional>
CLIENT_URL is a comma-separated list of origins allowed by CORS.
Leave Cloudinary keys empty to keep files in uploads/.
frontend/.env

VITE_API_URL=http://localhost:4000
The React app falls back to https://hangry-backend.onrender.com, so override it for local dev.

admin/.env (or .env.local)

VITE_API_URL=http://localhost:4000
users-admin/.env

VITE_API_URL=http://localhost:4000
mobile API base
Update both mobile/src/api/client.ts and mobile/src/until/image.ts so API_URL points at your backend (e.g., http://10.0.2.2:4000 for Android emulators). You can refactor these files to read from process.env.EXPO_PUBLIC_API_URL if you prefer.

Running the Apps
Service	Command	Notes
Backend API	cd backend && npm run server	Nodemon listens on PORT (default 4000).
Web storefront	cd frontend && npm run dev	Vite default port 5173.
Admin portal	cd admin && npm run dev	Requires VITE_API_URL.
User console	cd users-admin && npm run dev	Routes to /users by default.
Mobile app	cd mobile && npx expo start	Use npm run android / ios / web as needed.
Make sure backend is running before launching any client so API requests resolve.

API Reference (partial)
Method	Endpoint	Description	Auth Header (token)
POST	/api/user/register	Register user (name, email, password).	No
POST	/api/user/login	Login, returns JWT token.	No
GET	/api/user/list	List/search users (admin console).	Should be protected
POST	/api/user/update	Toggle roles or active flag.	Should be protected
POST	/api/user/remove	Delete user.	Should be protected
POST	/api/food/add	Add menu item with image (Multer or Cloudinary).	Should be admin
GET	/api/food/list	Public menu listing.	No
POST	/api/food/remove	Remove menu item (image cleaned up).	Admin
POST	/api/cart/add	Increment cart item count.	Yes
POST	/api/cart/remove	Decrement cart item count.	Yes
POST	/api/cart/get	Fetch persisted cart.	Yes
POST	/api/order/place	Stripe checkout, returns session_url.	Yes
POST	/api/order/place-cod	Place unpaid COD order.	Yes
POST	/api/order/verify	Called by frontend /verify screen post-checkout.	No
POST	/api/order/userorders	List current user orders.	Yes
GET	/api/order/list	Admin-facing order feed.	Admin
POST	/api/order/status	Update order status (Food Processing etc.).	Admin
Images uploaded via /api/food/add are available at /images/<filename> unless you supply Cloudinary keys (then the stored value is a https://res.cloudinary.com/... URL).

Stripe & Order Flow
Frontend/mobile posts items, amount, and address to /api/order/place.
Backend creates a Mongo order (default payment=false) and a Stripe Checkout session.
User is redirected to session_url. Stripe redirects back to ${FRONTEND_URL}/verify?....
Frontend calls /api/order/verify with orderId and success.
COD orders skip Stripe and call /api/order/place-cod.
Keep FRONTEND_URL aligned with the port that serves /verify, otherwise Stripe success/cancel pages will 404.

Assets & Uploads
Local images land in uploads/ and are statically served through app.use("/images", express.static("uploads")).
When Cloudinary env vars are present, files are pushed to foodfast/foods and only the secure URL is stored.
Admin UI will fall back to /header_img.png or the logo if an image fails to load (admin/src/pages/List/List.jsx).
Mobile Notes
Tabs live in mobile/app/(tabs) (index, cart, orders, myorders, profile).
State is managed with Zustand stores (mobile/lib/store/cart.ts, mobile/lib/store/auth.ts).
Axios client injects the JWT token header automatically (mobile/src/api/client.ts).
Update API_URL for local devices (iOS simulator can use http://localhost:4000, Android emulator needs http://10.0.2.2:4000).
Development Tips
Seed food documents manually or via Mongo shell so clients have items to render.
Keep the uploads/ folder writable by the backend server process.
CORS is strict: update CLIENT_URL whenever you introduce a new host/port.
Consider wrapping admin-only routes with middleware; currently they are open.
Contributing
Fork the repo.
Create a feature branch: git checkout -b feat/<feature-name>
Make changes + add tests if applicable.
Open a PR targeting dev.
License
MIT



Next steps: decide whether to replace `README.md` directly or adapt parts of this draft, then run through each service to confirm the env vars/ports are correct and that mobile clients point to the intended backend.
