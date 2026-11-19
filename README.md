# Customer Ticket Support System (CTSS) — Core MERN Stack

An enterprise-grade Customer Ticket Support System built with MongoDB, Express.js, React, and Node.js featuring Role-Based Access Control (RBAC), real-time ticket lifecycles, and professional dashboards.

## Key Features
- **Role-Based Access Control (RBAC)**: Distinct dashboards and routes for Customers and Support Admins.
- **Customer Workspace**: Submit support tickets, review ticket history, filter by priority and category, and chat with agents in real-time threads.
- **Support Operations Console**: Overview metrics, category & priority distribution analytics, ticket assignment to agents, urgency triage, and internal staff-only notes.
- **Strict Data Privacy**: Internal notes and ticket ownership strictly isolated and hidden from customers.
- **Zero-Comments Clean Code**: Production code is 100% comment-free with complete documentation in the companion manual.
- **Local MongoDB**: Fully configured to connect with local MongoDB instances with automatic index and seed scripts.

## Quick Start

1. **Seed Local Database**:
   ```bash
   cd server
   npm run seed
   ```

2. **Start Backend Server**:
   ```bash
   cd server
   npm start
   ```

3. **Start Frontend Development Server**:
   ```bash
   cd client
   npm run dev
   ```

## Demo Credentials

- **Admin**: `admin@ctss.com` / `AdminPassword123`
- **Agent**: `sarah.agent@ctss.com` / `AgentPassword123`
- **Customer**: `customer@ctss.com` / `CustomerPassword123`
- **Customer 2**: `elena@techcorp.io` / `CustomerPassword123`

*(The login page includes convenient one-click demo account switcher buttons)*

## Complete Technical Documentation
Refer to [ARCHITECTURE_AND_CODE_GUIDE.md](ARCHITECTURE_AND_CODE_GUIDE.md) for complete details on every codeblock, data model, API contract, and workflow.
