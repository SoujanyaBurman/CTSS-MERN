# Customer Ticket Support System (CTSS)
## Master Architecture, Code Explanation & API Manual

This document provides a comprehensive, exhaustive explanation of the **Customer Ticket Support System (CTSS)** codebase, architecture, role-based access control (RBAC), database schemas, and API contracts. In accordance with the project guidelines, all production source code files are kept completely comment-free, with this guide serving as the centralized source of truth.

---

## Table of Contents
1. [System Architectural Overview](#1-system-architectural-overview)
2. [Project Directory Layout](#2-project-directory-layout)
3. [Database Models & Persistence](#3-database-models--persistence)
4. [Backend Codeblock Walkthrough](#4-backend-codeblock-walkthrough)
   - [Server Entry & Database Configuration](#server-entry--database-configuration)
   - [Authentication & Role Middleware](#authentication--role-middleware)
   - [Centralized Error Handler](#centralized-error-handler)
   - [Auth Controller](#auth-controller)
   - [Ticket Controller](#ticket-controller)
   - [Admin Controller](#admin-controller)
   - [Routes Architecture](#routes-architecture)
   - [Database Seeder](#database-seeder)
5. [Frontend Codeblock Walkthrough](#5-frontend-codeblock-walkthrough)
   - [Application Shell & Routing System](#application-shell--routing-system)
   - [API Interceptors & Axios Client](#api-interceptors--axios-client)
   - [Authentication & Toast Contexts](#authentication--toast-contexts)
   - [Common UI Component Library](#common-ui-component-library)
   - [Ticket Component Library](#ticket-component-library)
   - [Customer Dashboard & Workflows](#customer-dashboard--workflows)
   - [Admin Operations & Incident Console](#admin-operations--incident-console)
6. [Complete REST API Specification](#6-complete-rest-api-specification)
7. [Getting Started & Operational Verification](#7-getting-started--operational-verification)

---

## 1. System Architectural Overview

CTSS is architected around a client-server REST design using the **Core MERN Stack**:
- **MongoDB**: Schema-based document store with local persistence (`mongodb://127.0.0.1:27017/ctss_db`).
- **Express.js & Node.js**: Modular backend handling business logic, authentication guards, and data filtering.
- **React 18 & Vite**: Fast client runtime utilizing single-page client-side routing and reactive UI components.
- **Tailwind CSS & Lucide Icons**: Modern dashboard styling with utility-first classes, color-coded priority alerts, and accessible interactive widgets.

### Core Architectural Principles
1. **Stateless JWT Authentication**: Authentication is enforced via JSON Web Tokens signed with a HMAC SHA-256 secret. The client stores the token in `localStorage` and injects it via an Axios request interceptor as `Authorization: Bearer <token>`.
2. **Strict Data Isolation & Privacy**: Customers are strictly constrained to tickets where `customer == req.user._id`. Furthermore, the backend strips all `internalNotes` subdocuments before serializing tickets to customer callers.
3. **Atomic Sequential Ticket IDs**: Instead of exposing raw MongoDB ObjectIDs to end users, tickets are automatically assigned sequential codes (e.g., `TC-1001`, `TC-1002`) powered by an atomic increment on a dedicated `Counter` model.
4. **Clean Code Standard**: Zero comments in production source code. Every variable, function, and component is named declaratively.

---

## 2. Project Directory Layout

```
CTSS-MERN/
├── ARCHITECTURE_AND_CODE_GUIDE.md     # Master documentation
├── package.json                       # Root script orchestrator
├── server/
│   ├── .env                           # Server environment configurations
│   ├── .env.example                   # Environment configuration template
│   ├── package.json                   # Server dependencies and scripts
│   ├── server.js                      # Express application entry point
│   ├── config/
│   │   └── db.js                      # MongoDB connection manager
│   ├── models/
│   │   ├── User.js                    # User account schema with bcrypt hooks
│   │   ├── Counter.js                 # Atomic ticket counter schema
│   │   └── Ticket.js                  # Ticket lifecycle and thread schema
│   ├── middleware/
│   │   ├── authMiddleware.js          # JWT verification & session hydration
│   │   ├── roleMiddleware.js          # RBAC authorization guard
│   │   └── errorHandler.js            # Standardized JSON error response handler
│   ├── controllers/
│   │   ├── authController.js          # Registration, login, profile management
│   │   ├── ticketController.js        # Ticket CRUD, reply threads, customer status
│   │   └── adminController.js         # Metrics aggregation, triage, staff notes, users
│   ├── routes/
│   │   ├── authRoutes.js              # /api/auth endpoints
│   │   ├── ticketRoutes.js            # /api/tickets endpoints
│   │   └── adminRoutes.js             # /api/admin endpoints
│   └── seed/
│       └── seedData.js                # Database seeder with mock data
│
└── client/
    ├── index.html                     # HTML root document
    ├── package.json                   # Client dependencies and build scripts
    ├── postcss.config.js              # PostCSS plugins (Tailwind, Autoprefixer)
    ├── tailwind.config.js             # Tailwind design tokens and theme extensions
    ├── vite.config.js                 # Vite bundler configuration with backend proxy
    └── src/
        ├── App.jsx                    # Root layout, routing and provider tree
        ├── index.css                  # Tailwind imports and custom scrollbar styles
        ├── main.jsx                   # React DOM root entry point
        ├── api/
        │   └── axiosInstance.js       # Pre-configured Axios instance with interceptors
        ├── context/
        │   ├── AuthContext.jsx        # User state, login, registration, session checks
        │   └── ToastContext.jsx       # Global notification toast alerts
        ├── components/
        │   ├── common/
        │   │   ├── Badge.jsx          # Color-coded status and priority tags
        │   │   ├── Loading.jsx        # Skeletons, spinners, and page loading states
        │   │   ├── Modal.jsx          # Accessible dialog component
        │   │   ├── Navbar.jsx         # Sticky top navigation with user profile badge
        │   │   ├── ProtectedRoute.jsx # RBAC route guard component
        │   │   └── Sidebar.jsx        # Responsive navigation drawer for dashboards
        │   ├── stats/
        │   │   └── StatCard.jsx       # Metric cards with icon badges and subtitles
        │   └── tickets/
        │       ├── InternalNotes.jsx  # Admin-only confidential staff collaboration panel
        │       ├── TicketFilter.jsx   # Multi-field search, status, and priority toolbar
        │       ├── TicketTable.jsx    # Responsive ticket list table with detail links
        │       └── TicketThread.jsx   # Interactive message thread with reply composer
        └── pages/
            ├── NotFound.jsx           # 404 Error page
            ├── auth/
            │   ├── Login.jsx          # Login view with 1-click demo role switchers
            │   └── Register.jsx       # Customer self-registration form
            ├── customer/
            │   ├── CustomerDashboard.jsx # Overview metrics and recent activity
            │   ├── CreateTicket.jsx      # Ticket submission form
            │   ├── MyTickets.jsx         # Searchable customer ticket history
            │   └── TicketDetail.jsx      # Interactive ticket conversation and closing
            └── admin/
                ├── AdminDashboard.jsx    # Real-time analytics, SLA alerts, category charts
                ├── TicketQueue.jsx       # Global ticket queue with multi-filter triage
                ├── AdminTicketDetail.jsx # Management console: status, agent assignment, notes
                └── UserManagement.jsx    # Registered user and agent directory
```

---

## 3. Database Models & Persistence

### 3.1. `User` Model (`server/models/User.js`)
Stores authentication credentials, personal details, and RBAC roles.
- **Fields**:
  - `name` (String, required): Full name of the user.
  - `email` (String, required, unique, lowercase): Primary identifier.
  - `password` (String, required, minlength 6, `select: false`): Bcrypt salted hash.
  - `role` (String, enum: `['customer', 'admin']`, default: `'customer'`): Defines system access permissions.
  - `department` (String, default: `'General'`): Team or organizational unit (e.g. `'Technical Operations'`, `'Billing & Accounts'`).
  - `avatar` (String): Profile image URL.
- **Hooks & Methods**:
  - `pre('save')`: Hashes the password using `bcrypt.genSalt(10)` whenever the field is modified.
  - `matchPassword(enteredPassword)`: Compares an incoming plain-text string with the stored hash using `bcrypt.compare`.

### 3.2. `Counter` Model (`server/models/Counter.js`)
Maintains sequential integer sequences for generating human-readable IDs.
- **Fields**:
  - `_id` (String): The sequence key (e.g., `'ticketId'`).
  - `seq` (Number, default: `1000`): The last issued integer sequence.

### 3.3. `Ticket` Model (`server/models/Ticket.js`)
Encapsulates support inquiries, message histories, status transitions, and administrative data.
- **Fields**:
  - `ticketId` (String, unique, indexed): Formatted identifier (e.g. `"TC-1001"`).
  - `title` (String, required): Summary of the reported issue.
  - `description` (String, required): Detailed initial explanation.
  - `category` (String, enum: `['Technical', 'Billing', 'General Inquiry', 'Feature Request', 'Account']`).
  - `priority` (String, enum: `['Low', 'Medium', 'High', 'Urgent']`, default: `'Medium'`).
  - `status` (String, enum: `['Open', 'In Progress', 'Resolved', 'Closed']`, default: `'Open'`).
  - `customer` (ObjectId, ref: `'User'`, required, indexed): Requester reference.
  - `assignedTo` (ObjectId, ref: `'User'`, default: null): Assigned support specialist.
  - `messages` (Array of Subdocuments):
    - `sender`: ObjectId referencing the user.
    - `senderRole`: String (`'customer'` or `'admin'`).
    - `senderName`: String snapshot for fast display.
    - `message`: Text content.
    - `createdAt`: Date timestamp.
  - `internalNotes` (Array of Subdocuments, admin-only):
    - `author`: ObjectId referencing the staff user.
    - `authorName`: String snapshot.
    - `note`: Confidential staff note.
    - `createdAt`: Date timestamp.
  - `resolvedAt`, `closedAt`: Timestamps populated on resolution or closure.
- **Hooks**:
  - `pre('save')`: If `ticketId` is not yet assigned, queries `Counter.findByIdAndUpdate({ _id: 'ticketId' }, { $inc: { seq: 1 } }, { new: true, upsert: true })` and formats the sequence as `TC-<seq>`.

---

## 4. Backend Codeblock Walkthrough

### Server Entry & Database Configuration

#### `server/config/db.js`
Initializes Mongoose connection using the `MONGODB_URI` environment variable, falling back to local MongoDB (`mongodb://127.0.0.1:27017/ctss_db`). Outputs connection host info on success and terminates the process with code 1 if a connection error occurs.

#### `server/server.js`
1. Loads environment variables with `dotenv.config()`.
2. Connects to MongoDB via `connectDB()`.
3. Registers standard middleware:
   - `cors()` for cross-origin resource sharing.
   - `express.json()` and `express.urlencoded()` for request body parsing.
4. Mounts health check endpoint `GET /api/health` returning server status and server time.
5. Mounts route routers:
   - `/api/auth` -> `authRoutes`
   - `/api/tickets` -> `ticketRoutes`
   - `/api/admin` -> `adminRoutes`
6. Registers 404 fallback handler for unmatched API routes.
7. Registers `errorHandler` middleware.
8. Binds HTTP server to `process.env.PORT` (default: 5000).

### Authentication & Role Middleware

#### `server/middleware/authMiddleware.js`
- Extracts the authorization header and parses `Bearer <token>`.
- Returns `401 Unauthorized` if the token is missing.
- Verifies token validity using `jwt.verify(token, JWT_SECRET)`.
- Queries `User.findById(decoded.id).select('-password')` and attaches the user document to `req.user`.
- Validates that the user still exists in the database.

#### `server/middleware/roleMiddleware.js`
- Higher-order function `authorize(...roles)` returning an Express middleware function.
- Evaluates `roles.includes(req.user.role)`.
- Rejects requests with `403 Forbidden` if permissions do not match.

#### `server/middleware/errorHandler.js`
- Captures thrown errors and sends consistent JSON `{ success: false, message: err.message, stack: ... }`.
- Automatically suppresses stack traces when `NODE_ENV === 'production'`.

### Auth Controller (`server/controllers/authController.js`)
- `generateToken(id)`: Produces a signed JWT containing `{ id }` valid for 30 days.
- `register`: Checks for duplicate emails, creates the user, generates a JWT, and returns user profile details.
- `login`: Fetches user including hidden password field (`.select('+password')`), executes `user.matchPassword(password)`, generates a token, and returns user details.
- `getMe`: Returns the currently authenticated user document (`req.user`).
- `updateProfile`: Modifies the user's name, department, or avatar and persists the change.

### Ticket Controller (`server/controllers/ticketController.js`)
- `getTickets`:
  - Enforces role isolation: If `req.user.role === 'customer'`, automatically sets `filter.customer = req.user._id`.
  - Supports query filters: `status`, `priority`, `category`, and regex search across `title`, `description`, and `ticketId`.
  - Populates referenced customer and assigned agent fields.
  - Strips `internalNotes` from all ticket documents if the requester is a customer.
- `createTicket`:
  - Validates `title` and `description`.
  - Sets `customer: req.user._id`.
  - Seeds the first entry in `messages` with the customer's issue description to inaugurate the thread.
- `getTicketById`:
  - Finds ticket by ID and populates references.
  - Verifies customer ownership: Returns 403 if a customer attempts to view another customer's ticket.
  - Sanitizes the payload by stripping `internalNotes` for customer users.
- `addMessage`:
  - Appends a new message subdocument with sender details, role, and timestamp.
  - If a customer replies to a `Resolved` ticket, automatically transitions status back to `In Progress`.
- `updateTicketStatus`:
  - Restricts customer transitions to closing (`Closed`) or reopening (`Open`).
  - Permits administrators to transition across all states (`Open`, `In Progress`, `Resolved`, `Closed`).
  - Sets `resolvedAt` or `closedAt` timestamps when entering respective terminal states.

### Admin Controller (`server/controllers/adminController.js`)
- `getMetrics`:
  - Computes counts for total, open, in-progress, resolved, closed, urgent, and unassigned tickets.
  - Executes MongoDB aggregation pipelines (`$group`) to produce category and priority distribution breakdowns.
  - Fetches the 6 most recent incidents.
- `assignTicket`:
  - Assigns or unassigns ticket to a designated support agent (`assignedTo`).
  - If ticket was `Open`, automatically updates status to `In Progress`.
- `updatePriority`:
  - Changes ticket urgency level (`Low`, `Medium`, `High`, `Urgent`).
- `addInternalNote`:
  - Appends private staff notes to `internalNotes` array.
- `getUsers`:
  - Returns directory of users with their associated ticket counts.
- `getAgents`:
  - Returns list of staff members with `role === 'admin'`.

### Routes Architecture
- `server/routes/authRoutes.js`: Maps registration, login, profile retrieval, and updates.
- `server/routes/ticketRoutes.js`: Protected routes for ticket listing, ticket creation, message posting, and status modification.
- `server/routes/adminRoutes.js`: Protected routes guarded with `authorize('admin')` for metrics, assignments, priorities, internal notes, and user directories.

### Database Seeder (`server/seed/seedData.js`)
- Clears `User`, `Ticket`, and `Counter` collections.
- Initializes counter at 1000.
- Creates demo accounts:
  - Admin: `admin@ctss.com` / `AdminPassword123`
  - Agent: `sarah.agent@ctss.com` / `AgentPassword123`
  - Customer: `customer@ctss.com` / `CustomerPassword123`
  - Customer: `elena@techcorp.io` / `CustomerPassword123`
- Seeds four realistic tickets with complete message threads, category tags, priority levels, and internal notes.

---

## 5. Frontend Codeblock Walkthrough

### Application Shell & Routing System

#### `client/src/App.jsx`
- Sets up `BrowserRouter`, `AuthProvider`, and `ToastProvider`.
- `AppLayout`: Standard application shell rendering the `Navbar`, `Sidebar`, and main `<Outlet />`.
- `RootRedirect`: Inspects authenticated state and role, automatically navigating the user to `/admin/dashboard`, `/customer/dashboard`, or `/login`.
- Declarative routes:
  - Public: `/login`, `/register`.
  - Customer: `/customer/*` protected by `<ProtectedRoute allowedRoles={['customer']}>`.
  - Admin: `/admin/*` protected by `<ProtectedRoute allowedRoles={['admin']}>`.
  - Fallback: `*` -> `<NotFound />`.

### API Interceptors & Axios Client (`client/src/api/axiosInstance.js`)
- Creates an Axios instance with base URL `/api`.
- Request Interceptor: Retrieves `ctss_token` from `localStorage` and injects `Authorization: Bearer <token>`.
- Response Interceptor: Listens for `401 Unauthorized` responses on authenticated requests, clears stored credentials, and redirects to `/login`.

### Authentication & Toast Contexts

#### `client/src/context/AuthContext.jsx`
- Maintains `user`, `token`, and `loading` states.
- On initialization, checks stored credentials and verifies validity against `GET /api/auth/me`.
- Provides `login`, `register`, and `logout` actions.
- Exposes convenience flags: `isAuthenticated`, `isAdmin`, and `isCustomer`.

#### `client/src/context/ToastContext.jsx`
- Manages an array of notification toasts with auto-dismiss timeouts (default: 4000ms).
- Renders an animated floating alert stack in the bottom-right viewport with distinct color themes for `success`, `error`, and `info`.

### Common UI Component Library

#### `client/src/components/common/Badge.jsx`
- Renders color-coded pill tags with matching rings and background shades.
- Statuses: Open (Sky blue), In Progress (Indigo), Resolved (Emerald), Closed (Slate).
- Priorities: Urgent (Rose), High (Orange), Medium (Amber), Low (Slate).

#### `client/src/components/common/Loading.jsx`
- `Spinner`: SVG loader icon with smooth spin animation.
- `PageLoading`: Centered full-height loader indicator.
- `Skeleton`: Shimmering placeholder box for content loading states.

#### `client/src/components/common/ProtectedRoute.jsx`
- Validates user session and role before rendering child route elements.
- Redirects unauthenticated users to `/login` preserving target return URL.
- Redirects unauthorized roles to their respective dashboards.

#### `client/src/components/common/Navbar.jsx`
- Top navigation bar featuring system branding, user avatar, role indicator badge, and sign-out button.
- Includes mobile menu toggle button for smaller screens.

#### `client/src/components/common/Sidebar.jsx`
- Sidebar with role-adaptive navigation links (`NavLink` with active states).
- Supports responsive drawer overlay on mobile viewports.

### Ticket Component Library

#### `client/src/components/tickets/TicketFilter.jsx`
- Provides real-time search input and dropdown selectors for Status, Priority, and Category.
- Features a quick-reset button to clear all active filters.

#### `client/src/components/tickets/TicketTable.jsx`
- Renders tabular list of tickets displaying ID, title, category, priority badge, status badge, assigned agent, update date, and action link.
- Displays responsive fallback view when zero tickets match filters.

#### `client/src/components/tickets/TicketThread.jsx`
- Chronological message conversation component.
- Visually distinguishes customer messages from support specialist messages using alternate alignments, badges, and color tokens.
- Includes a reply text area and submit button with loading state.

#### `client/src/components/tickets/InternalNotes.jsx`
- Admin-only collaboration panel styled in soft amber tones.
- Lists confidential staff notes with author attribution and timestamp.
- Allows staff to append new private notes without customer visibility.

### Customer Dashboard & Workflows

#### `client/src/pages/customer/CustomerDashboard.jsx`
- Computes and displays four summary metric cards: Total Tickets, Open Tickets, In Progress, and Resolved.
- Displays primary action button to raise a new ticket.
- Presents a table of the 5 most recent tickets.

#### `client/src/pages/customer/CreateTicket.jsx`
- Form with fields for title, category, priority, and detailed issue description.
- Validates input, submits to `POST /api/tickets`, displays toast notification, and navigates to the newly generated ticket detail view.

#### `client/src/pages/customer/MyTickets.jsx`
- Displays all tickets submitted by the authenticated customer.
- Integrated with `TicketFilter` and debounced search refetching.

#### `client/src/pages/customer/TicketDetail.jsx`
- Shows complete ticket details, assigned support engineer, status, and priority badges.
- Displays full message thread via `TicketThread`.
- Provides one-click action to close or reopen the ticket.

### Admin Operations & Incident Console

#### `client/src/pages/admin/AdminDashboard.jsx`
- Operational console displaying aggregated metrics: Total Incidents, Open Queue, Urgent Priority items, and Resolved cases.
- Progress bars illustrating ticket distribution by Category and Priority.
- Quick view of recent incidents with direct navigation to the full queue.

#### `client/src/pages/admin/TicketQueue.jsx`
- System-wide ticket queue displaying all tickets across all customers.
- Equipped with multi-field search and filters for triage.

#### `client/src/pages/admin/AdminTicketDetail.jsx`
- Specialist command center for an individual incident:
  - Updates ticket status (`Open`, `In Progress`, `Resolved`, `Closed`).
  - Adjusts ticket priority (`Low`, `Medium`, `High`, `Urgent`).
  - Assigns/reassigns ticket to available support agents.
  - Replies directly to the customer in the shared thread.
  - Appends private internal notes in `InternalNotes`.
  - Requester profile card with customer contact details.

#### `client/src/pages/admin/UserManagement.jsx`
- Directory of all registered users and agents.
- Displays role badge, department, registration date, and active ticket allocations.
- Includes live search filtering by name, email, or department.

---

## 6. Complete REST API Specification

### Authentication Endpoints (`/api/auth`)

| Method | Endpoint | Access | Description | Request Body |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/register` | Public | Register customer account | `{ name, email, password, role? }` |
| `POST` | `/api/auth/login` | Public | Authenticate user & issue JWT | `{ email, password }` |
| `GET` | `/api/auth/me` | Bearer JWT | Retrieve current user profile | None |
| `PUT` | `/api/auth/profile` | Bearer JWT | Update user profile details | `{ name?, department?, avatar? }` |

### Ticket Endpoints (`/api/tickets`)

| Method | Endpoint | Access | Description | Request Body |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/api/tickets` | Bearer JWT | List tickets (filtered by role) | Query: `?status=&priority=&category=&search=` |
| `POST` | `/api/tickets` | Bearer JWT | Create new support ticket | `{ title, description, category, priority }` |
| `GET` | `/api/tickets/:id` | Bearer JWT | Get single ticket with thread | None |
| `POST` | `/api/tickets/:id/messages` | Bearer JWT | Post reply to ticket thread | `{ message }` |
| `PATCH` | `/api/tickets/:id/status` | Bearer JWT | Update status (Open/Closed for Customer, any for Admin) | `{ status }` |

### Admin Endpoints (`/api/admin`)

| Method | Endpoint | Access | Description | Request Body |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/api/admin/metrics` | Admin JWT | Aggregate operational metrics & stats | None |
| `PATCH` | `/api/admin/tickets/:id/assign` | Admin JWT | Assign or reassign ticket to agent | `{ assignedTo }` (User ID or null) |
| `PATCH` | `/api/admin/tickets/:id/priority` | Admin JWT | Change ticket urgency | `{ priority }` (`Low` \| `Medium` \| `High` \| `Urgent`) |
| `POST` | `/api/admin/tickets/:id/notes` | Admin JWT | Append confidential internal staff note | `{ note }` |
| `GET` | `/api/admin/users` | Admin JWT | Directory of users with ticket counts | None |
| `GET` | `/api/admin/agents` | Admin JWT | List of staff members with admin role | None |

---

## 7. Getting Started & Operational Verification

### 7.1. Prerequisites
- **Node.js**: v18.0.0 or higher.
- **MongoDB**: Local MongoDB Server running on `127.0.0.1:27017`.

### 7.2. Installation & Database Seeding

1. **Install Server Dependencies**:
   ```bash
   cd server
   npm install
   ```

2. **Install Client Dependencies**:
   ```bash
   cd ../client
   npm install
   ```

3. **Seed Database with Demo Accounts and Sample Incidents**:
   ```bash
   cd ../server
   npm run seed
   ```

### 7.3. Running the Application

To launch both backend and frontend servers:

- **Terminal 1 (Backend API)**:
  ```bash
  cd server
  npm start
  ```
  *Server listens on `http://localhost:5000`.*

- **Terminal 2 (Frontend Client)**:
  ```bash
  cd client
  npm run dev
  ```
  *Client serves on `http://localhost:5173`.*

Or execute from the project root:
```bash
npm run server
npm run client
```

### 7.4. Demo Credentials for Immediate Testing

| Role | Email | Password | Primary Dashboard |
| :--- | :--- | :--- | :--- |
| **System Admin** | `admin@ctss.com` | `AdminPassword123` | `/admin/dashboard` |
| **Support Agent** | `sarah.agent@ctss.com` | `AgentPassword123` | `/admin/dashboard` |
| **Customer 1** | `customer@ctss.com` | `CustomerPassword123` | `/customer/dashboard` |
| **Customer 2** | `elena@techcorp.io` | `CustomerPassword123` | `/customer/dashboard` |

*Note: On the login screen (`/login`), three one-click buttons (`Admin`, `Agent`, `Customer`) automatically populate credentials for testing.*

