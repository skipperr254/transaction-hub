# Treatcode Technical Architecture

This document outlines the technical stack and architecture used to build the Treatcode application.

## 1. Frontend Architecture
The application is a Single Page Application (SPA) built for performance and interactivity.

*   **framework:** **React** (v18+) with **TypeScript**.
*   **Build Tool:** **Vite** - Used for extremely fast development server and optimized production builds.
*   **Routing:** **React Router DOM** - Handles client-side navigation (Dashboard, Auth, History pages) without page reloads.
*   **State Management & Data Fetching:**
    *   **React Query (@tanstack/react-query):** Manages server state, caching, and synchronization with Supabase.
    *   **Context API:** Used effectively for global UI states (like Toasts).

## 2. UI & Design System
The user interface is designed to be fully responsive "Mobile First" and accessible.

*   **Styling Engine:** **Tailwind CSS** - Utility-first CSS framework for rapid styling.
*   **Component Library:** **shadcn/ui** - A collection of re-usable components built using **Radix UI** primitives and Tailwind. This provides high-quality, accessible components (Cards, Sheets, Dialogs, Inputs) that we have customized.
*   **Icons:** **Lucide React** - Consistent icon set used throughout the app.
*   **Animations:** CSS native transitions and Tailwind animate classes `(animate-spin, animate-in)`.

## 3. Backend & Infrastructure (Supabase)
We utilize **Supabase** as a fully managed Backend-as-a-Service (BaaS), replacing the need for a traditional dedicated server.

*   **Database:** **PostgreSQL** - A robust relational database hosting our Users, Profiles, and Transaction data.
*   **Authentication:** **Supabase Auth** - Handles user sign-up, sign-in, and session management (JWT based).
*   **Real-time:** **Supabase Realtime** - Used to listen for database changes (e.g., when a deposit is confirmed) and update the UI instantly without refreshing.
*   **API/Logic:** We do not have a traditional Node.js/Python server. Instead, we use **Supabase Edge Functions** (running on Deno) for secure server-side logic:
    *   `create-billing-request`: Initiates payment flows.
    *   `complete-billing-request`: Finalizes payments.
    *   `manage-subscription`: Handles recurrence logic.
    *   `gocardless-webhook`: Secure listener for bank payment events.

## 4. Payment Integration
*   **Provider:** **GoCardless** - specialized in Direct Debit processing.
*   **Flow:** The frontend connects to Supabase Edge Functions, which interact securely with the GoCardless API to create mandates and payment requests.

## 5. Deployment
*   **Frontend Hosting:** The React application is built into static assets (HTML/CSS/JS) and is typically deployed to a global CDN provider like **Vercel** or **Netlify**. This ensures the site loads instantly from edge locations worldwide.
*   **Backend Hosting:** Hosting is managed entirely by Supabase on their cloud infrastructure (AWS/Fly.io).

---

### Summary Diagram

```mermaid
graph TD
    User[User Device] -->|Visits Website| CDN[Vercel/Netlify (Frontend)]
    User -->|Auth & Data| Supabase[Supabase (Database & Auth)]
    User -->|Payment Setup| Edge[Supabase Edge Functions]
    Edge -->|API Calls| GC[GoCardless API]
    GC -->|Webhooks| Edge
```