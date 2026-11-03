# Nestwell — Online Rental Property Management

Nestwell is a full-stack rental marketplace where tenants can discover and save homes, contact owners, and manage their rental search. Owners can publish listings and review incoming inquiries.

## Features

- Responsive property discovery with location, rent range, home type, sort, and pagination filters.
- Tenant and owner registration, JWT authentication, bcrypt password hashing, and protected role-specific screens.
- Property listing creation, editing, deletion, and local multi-image uploads for owners.
- Property detail pages with image gallery, amenities, owner profile, and tenant-to-owner inquiries.
- Owner dashboard with property management and received inquiries.
- Tenant favorites stored against the user account.
- MongoDB/Mongoose validation and indexes, request validation, centralized API errors, Helmet, CORS, and API rate limiting.
- Seed data: three demo accounts and 21 sample properties.

## Requirements

- Node.js 20 or newer and npm.
- MongoDB 6 or newer, local or hosted.

## Setup

1. Copy the project-level `.env.example` to `server/.env` (run `Copy-Item .env.example server/.env` from this directory), then set `MONGODB_URI` and a long random `JWT_SECRET`. The API loads its environment from the server working directory.
2. Install dependencies:

   ```powershell
   npm.cmd install
   npm.cmd install --prefix server
   npm.cmd install --prefix client
   ```

3. Start both the API and website together from this directory:

   ```powershell
   npm.cmd run dev
   ```

   Open `http://localhost:5173`. The website proxies API and image requests to the local server, so browser requests also work when opening the site from another device on the same network.

   To start the API and client separately instead:

   ```powershell
   cd server
   npm.cmd install
   npm.cmd run dev
   ```

   In a second terminal:

   ```powershell
   cd client
   npm.cmd install
   npm.cmd run dev
   ```

   The API listens on `http://localhost:5000`. Uploaded images are stored in `server/uploads` by default and served from `/uploads`. For a separately hosted API, set `VITE_API_URL` in `client/.env.local` to its full `/api` URL.

4. With `NODE_ENV=development`, the API automatically inserts 21 sample listings and demo users when the database has no properties. You can also seed manually from `server`:

   ```powershell
   npm.cmd run seed
   ```

   Demo accounts (all passwords are for local development only):

   | Role | Email | Password |
   | --- | --- | --- |
   | Owner | `olivia.owner@example.com` | `OwnerDemo123!` |
   | Owner | `noah.owner@example.com` | `OwnerDemo123!` |
   | Tenant | `ava.tenant@example.com` | `TenantDemo123!` |

   Seeding is repeatable, adds sample listings only when their titles do not already exist, and fills in photos for existing sample listings that do not yet have images. Change or remove these demo credentials before using any shared environment.

## Testing and production build

Run backend integration tests from `server`:

```powershell
npm.cmd test
```

Tests use `mongodb-memory-server` and do not require a separate MongoDB instance. Build the client from `client`:

```powershell
npm.cmd run build
```

For production, set `NODE_ENV=production`, use a managed MongoDB database and a strong secret, configure `CLIENT_URL` to the deployed client origin(s), and persist/serve the upload directory. Deploy the API and static client separately or configure a reverse proxy. Local disk image storage is intended for a single-instance deployment; use object storage for horizontally scaled deployments.

## API overview

All API responses use a `{ "success": true, "data": ... }` envelope. Errors use `{ "success": false, "message": "...", "errors": [...] }` where field details apply. Protected endpoints require `Authorization: Bearer <JWT>`.

### Authentication

| Method | Path | Access | Description |
| --- | --- | --- | --- |
| POST | `/api/auth/register` | Public | Create tenant or owner account (`name`, `email`, `password`, optional `role`, `phone`) |
| POST | `/api/auth/login` | Public | Authenticate with email and password |
| GET | `/api/auth/me` | Authenticated | Return the current user |

### Properties

| Method | Path | Access | Description |
| --- | --- | --- | --- |
| GET | `/api/properties` | Public | Search available homes; query: `location`, `minPrice`, `maxPrice`, `type`, `sort` (`newest`, `price-asc`, `price-desc`), `page`, `limit` (max 24) |
| GET | `/api/properties/:id` | Public | Property details and public owner information |
| GET | `/api/properties/my` | Owner | List the signed-in owner's properties |
| POST | `/api/properties` | Owner | Create a listing; accepts multipart form data and up to 8 `images` (JPEG/PNG/WebP, 5 MB each) |
| PUT | `/api/properties/:id` | Listing owner | Update listing fields and append uploaded images |
| DELETE | `/api/properties/:id` | Listing owner | Delete listing and its local images |

Listing fields: `title`, `description`, `location`, `price`, `type` (`Apartment`, `House`, `Villa`, `PG`, `Studio`), `bedrooms`, `bathrooms`, `area`, `amenities` (JSON array or comma-separated text), and `images` for uploads.

### Inquiries and favorites

| Method | Path | Access | Description |
| --- | --- | --- | --- |
| POST | `/api/inquiries` | Tenant | Contact the owner with `propertyId` and `message` |
| GET | `/api/inquiries/my` | Owner | List received inquiries, including tenant and property details |
| GET | `/api/favorites/my` | Tenant | List saved, available properties |
| POST | `/api/favorites/:propertyId` | Tenant | Save a property |
| DELETE | `/api/favorites/:propertyId` | Tenant | Remove a saved property |

`GET /api/health` is a public service health check.

## Screenshots

Screenshots can be added here after running the application locally.

- Home and filtered listings
- Property detail and contact form
- Owner dashboard and listing editor
- Tenant saved homes

## Project structure

```text
rental-system/
  client/                  # React, Vite, Tailwind CSS
    src/components/
    src/context/
    src/pages/
    src/services/
  server/                  # Express REST API and Mongoose
    src/config/
    src/controllers/
    src/middleware/
    src/models/
    src/routes/
    src/scripts/
    tests/
    uploads/
  .env.example
  README.md
```
