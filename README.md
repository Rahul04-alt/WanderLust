# WanderLust

A full-stack Airbnb-inspired web application for discovering, listing, and reviewing travel accommodations. Users can browse properties across various categories, interact with an embedded map, post reviews, and manage their own listings.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Environment Variables](#environment-variables)
  - [Installation](#installation)
  - [Seeding the Database](#seeding-the-database)
  - [Running the App](#running-the-app)
- [Routes Overview](#routes-overview)
- [Authentication & Authorization](#authentication--authorization)
- [Image Uploads](#image-uploads)
- [Maps & Geocoding](#maps--geocoding)

---

## Features

- **Browse Listings** — View all available property listings on the home page.
- **Category Filters** — Filter listings by category: Rooms, Iconic Cities, Trending, Mountains, Castles, Amazing Pools, Camping, Farm, Arctic, Beach, Boat, and Ski-in/out.
- **Search** — Search for listings by keyword.
- **Listing Detail Page** — View full details of a listing including title, description, price (INR), category, location, country, hosted-by info, an interactive Mapbox map, and all user reviews.
- **Create / Edit / Delete Listings** — Authenticated users can create new listings with image upload; only the listing owner can edit or delete them.
- **Reserve a Listing** — Authenticated users can reserve a listing.
- **Reviews** — Logged-in users can leave a star rating (1–5) and a comment on any listing. Only the review author can delete their own review.
- **User Authentication** — Signup, login, and logout using a username/email/password strategy (Passport.js local strategy).
- **Flash Messages** — Success and error notifications on all user actions.
- **Responsive UI** — Bootstrap-based layout that works across desktop and mobile.
- **Session Persistence** — Sessions stored in MongoDB via `connect-mongo`.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js (v22) |
| Web Framework | Express.js |
| Database | MongoDB (via Mongoose) |
| Templating | EJS + ejs-mate (layout support) |
| Authentication | Passport.js (Local Strategy, passport-local-mongoose) |
| Session Store | express-session + connect-mongo |
| Image Storage | Cloudinary + multer-storage-cloudinary |
| File Uploads | Multer |
| Maps | Mapbox SDK (Forward Geocoding + Map GL JS) |
| Validation | Joi |
| Env Config | dotenv |
| HTTP Method Override | method-override |
| Flash Notifications | connect-flash |

---

## Project Structure

```
WanderLust/
├── app.js                  # Entry point — Express app setup, routes, error handling
├── cloudConfig.js          # Cloudinary & Multer storage configuration
├── middlewares.js          # isLoggedIn, isOwner, isReviewAuthor, validateListing, validateReview
├── schemaValidation.js     # Joi schemas for listing and review validation
├── package.json
│
├── controllers/
│   ├── listings.js         # Listing CRUD, geocoding, filter, search, reserve
│   ├── reviews.js          # Create & delete reviews
│   └── users.js            # Signup, login, logout
│
├── models/
│   ├── listing.js          # Listing schema (title, description, image, price, location, country, category, geometry, owner, reviews)
│   ├── review.js           # Review schema (rating, comment, author)
│   └── user.js             # User schema (passport-local-mongoose)
│
├── routes/
│   ├── listing.js          # /listings routes
│   ├── review.js           # /listings/:id/reviews routes
│   └── user.js             # /signup, /login, /logout routes
│
├── views/
│   ├── layouts/boilerplate.ejs
│   ├── includes/           # navbar, footer, flash partials
│   ├── listings/           # index, show, new, edit, error views
│   └── users/              # signup, login views
│
├── public/
│   ├── css/                # style.css, rating.css
│   └── js/                 # map.js (Mapbox GL), script.js
│
├── init/
│   ├── data.js             # Seed data
│   └── index.js            # DB seeding script
│
└── utils/
    ├── ExpressError.js     # Custom error class
    └── wrapAsync.js        # Async error wrapper
```

---

## Getting Started

### Prerequisites

- **Node.js** v22+
- **MongoDB** (local instance or MongoDB Atlas)
- A **Cloudinary** account (free tier works)
- A **Mapbox** account with an access token (free tier works)

### Environment Variables

Create a `.env` file in the root of the project with the following keys:

```env
# MongoDB Atlas connection string
ATLASDB_URL=mongodb+srv://<username>:<password>@cluster.mongodb.net/wanderlust

# Session secret (any random strong string)
SECRET=your_session_secret_here

# Cloudinary credentials
CLOUD_NAME=your_cloudinary_cloud_name
CLOUD_API_KEY=your_cloudinary_api_key
CLOUD_API_SECRET=your_cloudinary_api_secret

# Mapbox access token
MAP_TOKEN=your_mapbox_public_token
```

> The app only loads `.env` when `NODE_ENV` is not `"production"`. In production, set these as actual environment variables.

### Installation

```bash
# Clone the repository
git clone https://github.com/Rahul04-alt/WanderLust.git
cd WanderLust

# Install dependencies
npm install
```

### Seeding the Database

To populate the database with sample listings:

```bash
node init/index.js
```

### Running the App

```bash
# Development
node app.js

# Or with nodemon (auto-restart on file changes)
npx nodemon app.js
```

The server starts on **http://localhost:8080** by default. Opening the root URL `/` redirects to `/listings`.

---

## Routes Overview

### Listings

| Method | Route | Description | Auth Required |
|---|---|---|---|
| GET | `/listings` | Browse all listings | No |
| GET | `/listings/new` | New listing form | Yes |
| POST | `/listings` | Create a new listing | Yes |
| GET | `/listings/filter/:id` | Filter by category | No |
| GET | `/listings/search` | Search listings | No |
| GET | `/listings/:id` | View listing details | No |
| GET | `/listings/:id/edit` | Edit listing form | Yes (Owner only) |
| PUT | `/listings/:id` | Update a listing | Yes (Owner only) |
| DELETE | `/listings/:id` | Delete a listing | Yes (Owner only) |
| GET | `/listings/:id/reservelisting` | Reserve a listing | Yes |

### Reviews

| Method | Route | Description | Auth Required |
|---|---|---|---|
| POST | `/listings/:id/reviews` | Add a review | Yes |
| DELETE | `/listings/:id/reviews/:reviewId` | Delete a review | Yes (Author only) |

### Users

| Method | Route | Description |
|---|---|---|
| GET | `/signup` | Signup form |
| POST | `/signup` | Register a new user |
| GET | `/login` | Login form |
| POST | `/login` | Authenticate user |
| GET | `/logout` | Log out current user |

---

## Authentication & Authorization

- Powered by **Passport.js** with the `passport-local` strategy and `passport-local-mongoose` for easy user management.
- Sessions are persisted in MongoDB using **connect-mongo** and encrypted with the `SECRET` environment variable.
- The `isLoggedIn` middleware guards all protected routes, saving the originally requested URL for post-login redirect.
- The `isOwner` middleware ensures only the listing's creator can edit or delete it.
- The `isReviewAuthor` middleware ensures only the review's author can delete their review.

---

## Image Uploads

Images are uploaded via **Multer** directly to **Cloudinary** using `multer-storage-cloudinary`. Files are stored in the `wanderlust_DEV` folder on Cloudinary. Accepted formats: `png`, `jpg`, `jpeg`, `webp`. On the edit page, the existing image is displayed at a reduced resolution (`w_250,h_160`) using Cloudinary's on-the-fly transformation URL.

---

## Maps & Geocoding

When a listing is created or updated, the location and country are forward-geocoded using the **Mapbox Geocoding API** to obtain GeoJSON `Point` coordinates. These are stored in the listing's `geometry` field. On the listing detail page, **Mapbox GL JS** renders an interactive map centred on the listing's coordinates.
WanderLust | Holidays Rentals, Cabins, Beach Houses &amp; More
