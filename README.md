# 🚀 Eleven's Deployment Hub - Technical Documentation

> **A comprehensive full-stack deployment platform similar to Vercel/Netlify**  
> Version: 1.0.0 | Last Updated: December 7, 2025

---

## 📑 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Technology Stack](#technology-stack)
3. [Database Schema & Models](#database-schema--models)
4. [Authentication System](#authentication-system)
5. [Project Management](#project-management)
6. [Deployment Pipeline](#deployment-pipeline)
7. [Frontend Architecture](#frontend-architecture)
8. [API Reference](#api-reference)
9. [Real-time Features](#real-time-features)
10. [Security Implementation](#security-implementation)
11. [File Storage System](#file-storage-system)
12. [Build & Worker System](#build--worker-system)

---

## 🏗️ Architecture Overview

### System Architecture

The platform follows a **5-tier architecture** designed for scalability, maintainability, and separation of concerns:

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT TIER                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │   React 19 (Vite 7) + Tailwind CSS 4                    │   │
│  │   - Zustand (State Management)                           │   │
│  │   - TanStack Query (Server State)                        │   │
│  │   - React Router v7 (Navigation)                         │   │
│  │   - Socket.IO Client (Real-time)                         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              ↕ HTTP/WebSocket
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION TIER                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │   Node.js 20 + Express.js                               │   │
│  │   ┌────────────┬────────────────┬─────────────────┐     │   │
│  │   │  Task 1    │   Task 2       │   Task 3        │     │   │
│  │   │  Auth      │   Projects     │   Deployments   │     │   │
│  │   │  System    │   & Files      │   & Builds      │     │   │
│  │   └────────────┴────────────────┴─────────────────┘     │   │
│  │   ┌────────────────────────────────────────────────┐     │   │
│  │   │  Middleware Layer                              │     │   │
│  │   │  - JWT Auth                                    │     │   │
│  │   │  - Role-based Access Control                   │     │   │
│  │   │  - Rate Limiting                               │     │   │
│  │   │  - Request Validation                          │     │   │
│  │   └────────────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                        DATA TIER                                 │
│  ┌──────────────┬───────────────┬───────────────┬──────────┐   │
│  │  MongoDB     │  Redis        │  MinIO/S3     │  Nginx   │   │
│  │  (Primary    │  (Queue &     │  (File        │  (Proxy  │   │
│  │   Database)  │   Cache)      │   Storage)    │   Server)│   │
│  └──────────────┴───────────────┴───────────────┴──────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                     WORKER TIER                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │   Bull Queue Workers                                     │   │
│  │   - Build Worker (Docker-based builds)                   │   │
│  │   - Deployment Worker                                    │   │
│  │   - Health Check Worker                                  │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### **Client Tier**
The presentation layer where users interact with the application. Built with modern React, it handles all UI rendering, user input, and real-time updates. The client communicates with the backend exclusively through REST APIs and WebSocket connections, never directly accessing the database or file storage.

#### **Application Tier**
The business logic layer that processes all requests. It's divided into three major task modules (Authentication, Projects, Deployments), each with its own controllers, services, and routes. This tier validates all incoming data, enforces business rules, manages permissions, and orchestrates operations across different services. The middleware layer acts as a security checkpoint, ensuring every request is authenticated, authorized, and valid before reaching the core business logic.

#### **Data Tier**
The persistence layer with four specialized storage systems:
- **MongoDB**: Stores all structured data (users, projects, deployments, logs)
- **Redis**: Manages job queues and caching for performance
- **MinIO/S3**: Handles all file storage (source code, built assets, user uploads)
- **Nginx**: Serves deployed applications to end-users with optimal caching and compression

#### **Worker Tier**
The background processing layer that handles computationally expensive tasks asynchronously. Workers pull jobs from Redis queues and execute them independently from the main API server. This prevents long-running build processes from blocking user requests and enables horizontal scaling by running multiple worker instances.

### Design Patterns & Architectural Principles

#### 1. **MVC (Model-View-Controller) Pattern**
The backend strictly separates data models (MongoDB schemas), business logic (controllers/services), and presentation (API responses). This makes the code more maintainable because changes to one layer don't cascade to others.

**Why we use it**: When you need to change how authentication works, you only modify the auth controller without touching the User model or frontend code.

#### 2. **Repository Pattern**
All database operations are abstracted through Mongoose models, which act as repositories. Instead of writing raw MongoDB queries throughout the codebase, we use model methods like `User.findOne()` or `Project.create()`.

**Why we use it**: If we ever switch from MongoDB to PostgreSQL, we only need to update the model layer, not every controller that accesses data.

#### 3. **Middleware Pattern (Chain of Responsibility)**
HTTP requests pass through a series of middleware functions before reaching the controller. Each middleware performs one specific task (authentication, logging, validation) and either passes the request forward or terminates it with an error.

**Why we use it**: Security checks are centralized. Every protected route automatically gets authentication and authorization checks without repetitive code.

#### 4. **Observer Pattern**
Socket.IO implements this pattern for real-time updates. The server (subject) maintains a list of connected clients (observers) and notifies them when events occur (deployment status changes, new build logs).

**Why we use it**: Users see build progress in real-time without polling the server every second, reducing server load and improving user experience.

#### 5. **Queue Pattern (Producer-Consumer)**
The main API server produces jobs (deployment requests) and adds them to Redis queues. Worker processes consume these jobs and execute them asynchronously. This decouples job creation from job execution.

**Why we use it**: The API server remains responsive even when 100 users trigger deployments simultaneously. Jobs are processed in order by available workers without blocking the API.

#### 6. **Factory Pattern**
Build configurations are generated dynamically based on the project's framework. The system detects whether it's a React, Vue, or static site and creates the appropriate build commands, output directories, and environment setup.

**Why we use it**: Adding support for a new framework (like Svelte) only requires adding a new configuration object, not rewriting the entire build system.

---

## 🛠️ Technology Stack

### Why These Technologies?

Our technology choices are based on industry standards, performance requirements, and developer experience. Each component serves a specific purpose in the deployment pipeline.

### Backend Technologies - Deep Dive

#### **Core Framework: Express.js + Node.js 20**
Express is a minimal, fast web framework that gives us complete control over the HTTP layer. We chose Node.js 20 for several reasons:
- **Event-driven architecture**: Perfect for I/O-heavy operations like file uploads and deployment monitoring
- **Single language**: JavaScript across frontend and backend reduces context switching
- **Rich ecosystem**: npm has packages for everything we need
- **Stream support**: Efficiently handles large file uploads and log streaming
- **Latest LTS**: Node 20 provides performance improvements and long-term support

#### **Database: MongoDB with Mongoose**
MongoDB is a NoSQL document database that stores data in flexible JSON-like documents. We chose it because:
- **Schema flexibility**: Projects can have varying structures (React vs Node.js deployments)
- **Horizontal scalability**: Easy to shard data across multiple servers as we grow
- **Rich querying**: Supports complex queries, aggregations, and full-text search
- **Mongoose ODM**: Provides schema validation, middleware hooks, and cleaner syntax
- **Document-oriented**: Perfect for nested data like build logs within deployments

**Why not PostgreSQL?**: While Postgres is excellent, our data model has varying structures (different frameworks need different configs) which suits MongoDB's flexibility better.

#### **Authentication: JWT (JSON Web Tokens)**
JWTs are stateless tokens that contain user information cryptographically signed by the server. We use them because:
- **Stateless**: No need to store session data in the database for every request
- **Scalable**: Multiple API servers can verify tokens without shared session storage
- **Mobile-friendly**: Works seamlessly with mobile apps and SPAs
- **Distributed systems**: Microservices can independently verify tokens
- **Standard**: Industry-standard with libraries in every language

**Our JWT Strategy**:
- **Access tokens** (15 minutes): Short-lived for security, used for API requests
- **Refresh tokens** (7 days): Long-lived, stored hashed in database, used to get new access tokens
- **Token rotation**: New refresh token issued on each refresh to detect token theft

#### **File Storage: MinIO (S3-Compatible)**
MinIO provides object storage compatible with Amazon S3 APIs. We chose it because:
- **Self-hosted**: We control our data without AWS costs
- **S3-compatible**: Can easily migrate to AWS S3 if needed
- **Performance**: Optimized for high-throughput file operations
- **Scalability**: Supports clustering and distributed storage
- **Cost-effective**: No per-GB storage fees for self-hosting

**Storage Strategy**:
- Source code stored in MinIO for durability
- Built assets packaged as ZIP files
- User avatars and project thumbnails cached with CDN headers
- Separate buckets for projects, avatars, and templates

#### **Queue System: Bull + Redis**
Bull is a Redis-based queue for Node.js that handles background jobs. We use queues because:
- **Async processing**: Builds can take 5-10 minutes; queues prevent API timeout
- **Retry logic**: Failed builds automatically retry with exponential backoff
- **Priority queues**: Paid users' deployments can jump the queue
- **Monitoring**: Bull provides a dashboard to track job progress
- **Horizontal scaling**: Multiple workers can process jobs simultaneously

**Job Flow**:
1. User triggers deployment → API creates job → Job added to Redis queue
2. Worker picks up job → Processes build → Updates database → Emits Socket.IO events
3. If fails: Job retries up to 3 times with increasing delays

#### **Real-time: Socket.IO**
Socket.IO enables bidirectional, event-based communication between client and server. We use it for:
- **Build logs**: Stream logs to frontend as they're generated
- **Deployment status**: Instant status updates (queued → building → deploying → success)
- **Notifications**: Real-time alerts for completed deployments
- **Collaborative features**: Multiple team members see updates simultaneously

**Why not Server-Sent Events (SSE)?**: Socket.IO provides bidirectional communication, automatic reconnection, and fallback to HTTP long-polling if WebSockets fail.

#### **Security: Helmet + bcrypt + Rate Limiting**
- **Helmet**: Adds 15+ HTTP security headers (XSS protection, CSP, HSTS)
- **bcrypt**: Industry-standard password hashing with configurable work factor (we use 12 rounds)
- **express-rate-limit**: Prevents brute-force attacks by limiting login attempts
- **Input validation**: Mongoose schemas + Zod prevent injection attacks

### Frontend Technologies - Deep Dive

#### **Framework: React 19 with Vite 7**
React is the most popular UI library for building interactive interfaces. We use the latest version because:
- **Component-based**: Reusable UI components reduce code duplication
- **Virtual DOM**: Efficient updates only re-render changed elements
- **Ecosystem**: Massive library of third-party components
- **Server Components**: React 19 enables faster initial page loads
- **Vite**: Lightning-fast dev server with instant HMR (Hot Module Replacement)

**Why not Next.js?**: Next.js is great for SEO-heavy sites, but our dashboard is authenticated and doesn't need server-side rendering. Vite gives us faster development without the overhead.

#### **State Management: Zustand + TanStack Query**
We use a hybrid approach:
- **Zustand**: Global state (user authentication, theme) - lightweight alternative to Redux
- **TanStack Query**: Server state (projects, deployments) - handles caching, refetching, optimistic updates
- **Component state**: Local UI state (form inputs, modals) - stays in component

**Why not just Redux?**: Redux has too much boilerplate for simple apps. Zustand gives us global state with 10x less code.

**Why TanStack Query?**: It automatically handles the complexity of server state:
- Caches API responses to reduce network requests
- Refetches stale data in the background
- Provides loading/error states out of the box
- Optimistic updates for better UX
- Automatic retries on failure

#### **Styling: Tailwind CSS 4**
Tailwind is a utility-first CSS framework that provides low-level classes. We chose it because:
- **Rapid development**: Build UIs without writing custom CSS
- **Consistency**: Design system built into class names
- **Performance**: Purges unused CSS in production (tiny bundles)
- **Responsive**: Mobile-first with intuitive breakpoint syntax
- **Customizable**: Easy to extend with custom colors, spacing, etc.

**Component Libraries**: We use shadcn/ui patterns (unstyled components + Tailwind) for flexibility without the bloat of full UI libraries like Material-UI.

#### **Forms: React Hook Form + Zod**
React Hook Form manages form state with minimal re-renders. Zod provides TypeScript-first schema validation. Together they:
- **Performance**: Only re-render changed fields
- **Validation**: Schema-based validation with great error messages
- **Type safety**: Zod schemas generate TypeScript types
- **Developer experience**: Less boilerplate than Formik
- **Bundle size**: Lightweight (45KB vs 150KB for Formik)

### Infrastructure & DevOps

#### **Nginx**
Nginx acts as a reverse proxy and web server for deployed applications. We use it because:
- **Performance**: Handles 10,000+ concurrent connections efficiently
- **Static file serving**: Optimized for serving built React apps
- **Gzip compression**: Reduces bandwidth usage by 70-80%
- **Caching**: Browser caching for static assets (CSS, JS, images)
- **SPA support**: try_files directive handles client-side routing
- **SSL termination**: Can add Let's Encrypt certificates easily

**Dynamic configuration**: Each deployment gets its own Nginx server block auto-generated with proper caching rules, compression, and security headers.

#### **Docker (Planned)**
Currently builds run directly on the host machine. Docker will:
- **Isolation**: Each build runs in a sandboxed container
- **Consistency**: Same environment locally, staging, production
- **Security**: Malicious code can't access host system
- **Dependency management**: Each framework gets its own container with required tools
- **Resource limits**: Prevent one build from consuming all CPU/RAM

#### **Bull Dashboard**
Visual monitoring for background jobs showing:
- Active builds in progress
- Completed builds with timing metrics
- Failed builds with error details
- Queue depth and processing rate
- Worker utilization across instances

---

## 🗄️ Database Schema & Models

### MongoDB Schema Design Philosophy

MongoDB allows flexible, JSON-like documents that can evolve over time. Unlike traditional relational databases with rigid table structures, MongoDB documents can have optional fields and nested objects. This is perfect for deployment platforms where different project types have different requirements.

### 1. User Model - The Foundation of Authentication

**Purpose**: Manages user accounts, authentication credentials, and security features.

#### **Core Identity Fields**
- `username`: Unique identifier for login and display. Must be 3-30 characters to prevent abuse (too short = collision, too long = spam).
- `email`: Primary contact and login method. Validated with regex to ensure format correctness. Stored lowercase to prevent duplicate accounts (user@email.com vs USER@EMAIL.COM).
- `password`: Never stored in plaintext. Hashed with bcrypt using 12 salt rounds, which takes ~200ms to hash (slow enough to prevent brute-force, fast enough for good UX).

#### **Email Verification System**
Why do we require email verification?
1. **Prevent spam accounts**: Bots can't verify real emails
2. **Password recovery**: We need a verified email to reset passwords
3. **Communication**: Ensure we can reach users for important notifications
4. **Trust**: Users with verified emails are more likely to be legitimate

**How it works**:
- On signup, we generate a cryptographically random token (impossible to guess)
- Token is valid for 24 hours to prevent stale links
- Email contains link: `https://app.com/verify-email/{token}`
- User clicks → Server validates token → Sets `isEmailVerified: true`
- Until verified, users can't access full features (can't deploy, limited uploads)

#### **Password Reset Flow**
Similar to email verification but with stricter security:
- Token valid for only 1 hour (shorter window to prevent abuse)
- One-time use: Token deleted after successful reset
- All active sessions invalidated: Forces re-login on all devices
- Password requirements enforced: Minimum 8 characters, complexity rules

#### **Account Security Features**

**Login Attempt Tracking**:
Prevents brute-force attacks by tracking failed logins:
- After 5 failed attempts: Account locked for 1 hour
- `lockUntil` field stores unlock timestamp
- Automatic unlock after timeout (no admin intervention needed)
- Successful login resets attempt counter

**Why 5 attempts?**: Balance between security and user experience. Users legitimately forget passwords but 5 attempts is enough for brute-force prevention.

**Session Management**:
- `lastLogin`: Track user activity patterns
- `apiKey`: For programmatic access (CLI tools, webhooks)
- `isActive`: Soft delete without losing historical data

#### **Security Methods Explained**

**comparePassword()**: 
Instead of storing passwords, we store bcrypt hashes. When a user logs in:
1. User submits plaintext password
2. We hash it with bcrypt
3. Compare hash with stored hash
4. Match = login success, No match = wrong password

This is one-way: You can't reverse a hash to get the original password.

**isLocked()**:
Checks if current time < lockUntil time. Simple but effective at preventing automated attacks during lockout period.

**incLoginAttempts()**:
Increments failed login counter. After 5 attempts, sets `lockUntil` to 1 hour from now. Uses atomic MongoDB operations to prevent race conditions (two failed logins at the same time).

---

### 2. Project Model - Deployable Applications

**Purpose**: Represents a user's project (website, app, API) that can be deployed to a live URL.

#### **Subdomain System**
Every project gets a unique subdomain for accessing the deployed application.

**Format**: `project-name.local` (e.g., `my-portfolio.local`, `react-app.local`)

**Why .local?**: 
- Development TLD that doesn't conflict with real domains
- Can be accessed on local network
- Easy to configure in Windows hosts file
- Production would use real domains like `.vercel.app`

**Subdomain Validation Rules**:
- Lowercase only (prevents DNS case-sensitivity issues)
- Alphanumeric + hyphens (DNS-safe characters)
- Must start/end with alphanumeric (no hanging hyphens)
- Unique across all projects (enforced by MongoDB unique index)

**Example Flow**:
- User creates "My React App"
- System generates: `my-react-app.local`
- Nginx serves files from `/deployments/my-react-app.local/`
- Users access at `http://my-react-app.local`

#### **Framework Detection**
The `framework` field determines how the project is built and deployed:

- **react/vue/angular**: Frontend frameworks needing npm build
- **next/nuxt**: Full-stack frameworks with server-side rendering
- **static**: Plain HTML/CSS/JS (no build needed)
- **node/express**: Backend APIs (different deployment strategy)
- **other**: Custom build process

**Why track framework?**: Each framework has different:
- Build commands (`npm run build` vs `next build`)
- Output directories (`build/` vs `dist/` vs `.next/`)
- Environment setup (Node version, package manager)
- Deployment strategy (static files vs Node server)

#### **Ownership & Permissions**
- `userId`: Who created the project (always has full access)
- `teamId`: If project belongs to a team, all team members get access
- Combined with TeamMember roles to enforce permissions

**Permission Flow**:
1. User makes API request to edit project
2. Middleware checks: Is user the owner OR a team member with edit rights?
3. If no: Return 403 Forbidden
4. If yes: Proceed with operation

#### **Project Statistics**
- `totalFiles`: Count of uploaded files (updated on each upload/delete)
- `totalSize`: Bytes used by project (enforces storage quota)
- `viewCount`: How many times deployed URL was accessed
- `lastDeployedAt`: When last successful deployment completed

**Why track these?**: 
- Show users their storage usage
- Identify popular projects for analytics
- Detect inactive projects for cleanup

#### **Status Lifecycle**
- `active`: Normal state, can deploy
- `archived`: Hidden from main list, can't deploy (preserve history)
- `draft`: Not yet deployed, still configuring

**Favorites System**:
`isFavorite` flag lets users bookmark important projects for quick access. UI shows starred projects at top of list.

#### **Database Indexes for Performance**

MongoDB indexes speed up queries by creating sorted data structures:

- **userId index**: Fast lookup of "all projects by this user" (most common query)
- **teamId index**: Fast lookup of "all team projects" for collaboration features
- **subdomain unique index**: Enforces uniqueness AND enables fast DNS lookups

Without indexes, MongoDB would scan every document. With 100,000 projects, a query could take seconds. Indexes reduce this to milliseconds.

---

### 3. Deployment Model - Build History & Tracking

**Purpose**: Records every deployment attempt, whether successful or failed, with complete logs and metrics.

#### **Deployment Lifecycle States**

```
queued → building → deploying → success
                              ↓
                            failed
```

- **queued**: Job created, waiting for worker
- **building**: Worker is running npm install/build
- **deploying**: Copying built files to Nginx directory
- **success**: Live and accessible at deployment URL
- **failed**: Error occurred, logs contain details
- **cancelled**: User manually stopped deployment

**Why track state?**: Users see real-time progress. Backend knows which stage failed for debugging.

#### **Build Logs System**

Each log entry has:
- `message`: What happened ("Installing dependencies...", "Build complete!")
- `type`: Severity level (info/warning/error/success/command)
- `timestamp`: Exact time for troubleshooting

**Log Types Explained**:
- `info`: Normal progress updates
- `warning`: Non-critical issues (deprecated packages)
- `error`: Build failures (syntax errors, missing dependencies)
- `success`: Major milestones (build complete, deployment live)
- `command`: Shell commands executed (`npm install`, `npm run build`)

**Real-time Log Streaming**:
As build progresses, worker adds logs to database AND emits Socket.IO events. Frontend receives logs instantly and displays them in a terminal-style interface.

#### **Build Configuration**
These fields control HOW the project is built:

- `buildCommand`: What to run to build the project (`npm run build`)
- `installCommand`: How to install dependencies (`npm install`)
- `outputDir`: Where built files are located (`build/` or `dist/`)
- `nodeVersion`: Which Node.js version to use (`20.x`)

**Framework Defaults**: When creating a deployment, system auto-fills these based on project framework. User can override for custom setups.

#### **Performance Metrics**
- `buildDuration`: Seconds from start to finish (helps identify slow builds)
- `buildSize`: Bytes of final built assets (compressed)
- `startedAt`: When build worker picked up job
- `completedAt`: When final status set (success/failed)

**Why measure these?**: 
- Identify performance bottlenecks
- Charge enterprise users based on build minutes
- Detect infinite loop builds (timeout after 10 minutes)
- Compare build times across deployments to track regressions

#### **Error Tracking**
When builds fail, we capture:
- `error.message`: Human-readable description
- `error.stack`: Full stack trace for debugging
- `error.code`: Error category (DEPENDENCY_FAILED, BUILD_TIMEOUT, etc.)

**Categorizing Errors**: Makes it easier to provide helpful error messages. Instead of showing raw error, we detect common issues:
- "Missing package.json" → Suggest creating one
- "Command not found: npm" → Node.js not installed
- "Port already in use" → Another build is running

#### **Rollback System**
- `rollbackFrom`: Reference to previous deployment being reverted
- `isActive`: Only one deployment per project is active at a time

**Rollback Flow**:
1. User clicks "Rollback to v3"
2. System creates new deployment copying v3's configuration
3. Re-runs build with v3's code
4. On success: Deactivates current deployment, activates new one
5. Nginx automatically serves new version

**Why not just switch pointers?**: Re-building ensures environment consistency. Old builds might not work with newer Node versions or dependencies.

---

## 🔐 Authentication System

### Understanding JWT (JSON Web Tokens)

JWT is a standard for creating tokens that assert claims. Think of it like a digital passport that proves who you are without needing to check with a central authority every time.

**Why JWT instead of traditional sessions?**

**Traditional Session-based Auth (Old Way)**:
1. User logs in → Server creates session ID → Stores in database → Returns cookie with session ID
2. Every request → Server checks database for session ID → Validates → Proceeds
3. Logout → Delete session from database

**Problems**:
- Database hit on EVERY request (slow)
- Doesn't scale horizontally (sessions tied to one server)
- Complex with load balancers (need sticky sessions)

**JWT-based Auth (Our Approach)**:
1. User logs in → Server creates JWT with user info → Signs it with secret key → Returns token
2. Every request → Server verifies signature (no database) → Trusts claims in token → Proceeds
3. Logout → Client deletes token (server doesn't track it)

**Benefits**:
- **Stateless**: Server doesn't store anything, verifies signature mathematically
- **Scalable**: Any API server can verify tokens without shared session storage
- **Distributed**: Microservices can independently verify tokens with the same secret key
- **Mobile-friendly**: Works seamlessly with mobile apps (no cookies needed)
- **Cross-origin**: Easier to handle than session cookies with CORS

#### Two-Token Strategy

We use **two types of tokens** to balance security and user experience:

**Access Token (15 minutes)**:
- Short-lived for security
- Used for every API request
- If stolen, attacker only has 15 minutes of access
- Sent in Authorization header: `Bearer {token}`

**Refresh Token (7 days)**:
- Long-lived for convenience
- Used only to get new access tokens
- Stored hashed in database for revocation
- Rotates on each use to detect theft

**Why two tokens?**: If we only used long-lived access tokens, stolen tokens would be valid for days. If we only used short-lived tokens, users would have to re-login every 15 minutes. Two tokens give us both security and convenience.

**Token Rotation Explained**:
When a refresh token is used, we immediately issue a new refresh token and invalidate the old one. If someone steals your refresh token and uses it, you'll get logged out when you try to refresh, alerting you to the breach.

#### Authentication Middleware Flow

Every protected route passes through middleware that verifies the user's identity:

**Step 1: Extract Token**
```
Request comes in with header:
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Middleware extracts the token after "Bearer "
```

**Step 2: Verify Signature**
```
JWT consists of three parts: header.payload.signature

Server uses JWT_SECRET to verify the signature:
- If signature matches: Token hasn't been tampered with
- If signature fails: Return 401 Unauthorized
```

**Step 3: Check Expiration**
```
Payload contains exp (expiration timestamp):
- If current time < exp: Token is valid
- If current time > exp: Return 401 (token expired)
```

**Step 4: Attach User Object**
```
Token payload contains user info:
{ sub: userId, email: "user@example.com", role: "user" }

Middleware queries database for full user object
Attaches to req.user for controllers to use
```

**Step 5: Proceed or Block**
```
If all checks pass:
  next() → Request proceeds to controller

If any check fails:
  return res.status(401).json({ error: 'Unauthorized' })
```

#### Registration Flow with Email Verification

**Why require email verification?**
- **Spam prevention**: Bots can't verify real email addresses
- **Password recovery**: We need a verified email to send reset links
- **Communication channel**: Ensures we can reach users for important updates
- **Trust signal**: Verified users are more likely to be legitimate

**The Complete Flow**:

**1. User Submits Signup Form**
```
Frontend sends:
{
  username: "johndoe",
  email: "john@example.com",
  password: "SecurePass123!"
}
```

**2. Server Validates Input**
```
- Username: 3-30 characters, alphanumeric + underscores
- Email: Valid email format, not already registered
- Password: Minimum 8 characters, at least one uppercase, one lowercase, one number
```

**3. Check for Existing Users**
```
const existingUser = await User.findOne({
  $or: [{ email }, { username }]
});

If exists: Return 409 Conflict "Email/username already taken"
```

**4. Hash Password**
```
bcrypt hashes password with 12 salt rounds (~200ms per hash)

This is intentionally slow to prevent brute-force attacks
Attacker would need 200ms × 10,000 attempts = 33 minutes for 10K passwords
```

**5. Generate Verification Token**
```
const token = crypto.randomBytes(32).toString('hex');

Cryptographically secure random token (impossible to guess)
Valid for 24 hours (enough time to check email, not too long to be insecure)
```

**6. Send Verification Email**
```
Email contains link: https://app.com/verify-email/{token}

Email service (Nodemailer) sends via SMTP
Template includes branding, instructions, and expiry warning
```

**7. Save User to Database**
```
User saved with:
- isEmailVerified: false
- emailVerificationToken: hashed token
- emailVerificationExpires: 24 hours from now

User can't access full features until verified
```

**8. User Clicks Verification Link**
```
Browser navigates to /verify-email/{token}
Server validates token hasn't expired
Sets isEmailVerified: true
Clears verification token
Redirects to login with success message
```

#### Password Reset Flow - Maximum Security

Password reset is a security-critical operation because it bypasses authentication. We implement multiple safeguards:

**Request Reset (User Forgot Password)**:

**1. User Enters Email**
```
POST /api/auth/forgot-password
{ email: "john@example.com" }
```

**2. Server Finds User**
```
const user = await User.findOne({ email });

If user doesn't exist: Still return success (prevent email enumeration attacks)
"If account exists, reset email sent"
```

**3. Generate Reset Token**
```
const resetToken = crypto.randomBytes(32).toString('hex');

Token valid for only 1 hour (shorter than email verification)
Why? Password reset is more sensitive - stolen token gives account access
```

**4. Send Reset Email**
```
Email contains:
- Reset link: https://app.com/reset-password/{token}
- Expiry time: "This link expires in 1 hour"
- Security warning: "If you didn't request this, ignore this email"
```

**5. Store Hashed Token**
```
user.passwordResetToken = hash(resetToken);
user.passwordResetExpires = Date.now() + 3600000; // 1 hour

We hash the token before storing so database breach doesn't expose reset tokens
```

**Reset Password (User Submits New Password)**:

**1. User Clicks Link & Submits Form**
```
GET /reset-password/{token} → Shows form
POST /api/auth/reset-password/{token}
{ newPassword: "NewSecurePass456!" }
```

**2. Validate Token**
```
const user = await User.findOne({
  passwordResetToken: hash(token),
  passwordResetExpires: { $gt: Date.now() } // Not expired
});

If invalid/expired: Return 400 "Invalid or expired reset token"
```

**3. Enforce Password Requirements**
```
- Different from old password (prevent reuse)
- Meets complexity requirements (8+ chars, mixed case, numbers)
- Not in common password list (password123, qwerty, etc.)
```

**4. Hash New Password**
```
user.password = await bcrypt.hash(newPassword, 12);

bcrypt automatically generates new salt (old password can't be reversed)
```

**5. Clear Reset Token**
```
user.passwordResetToken = undefined;
user.passwordResetExpires = undefined;

One-time use: Token immediately invalidated
Even if attacker intercepts token, it's useless after first use
```

**6. Invalidate All Sessions**
```
await Session.deleteMany({ userId: user._id });

Force re-login on all devices (desktop, mobile, tablets)
Why? If account was compromised, attacker's sessions are also terminated
```

**7. Send Confirmation Email**
```
"Your password was successfully reset"
"If this wasn't you, contact support immediately"

Alerts legitimate user if someone else reset their password
```

#### Account Locking & Brute-Force Prevention

**The Attack Vector**:
Without protection, attackers can try thousands of password combinations per second against your login endpoint.

**Our Defense - Progressive Lockout**:

**Failed Login Counter**:
```
User fails login → loginAttempts++

After 5 failed attempts:
  lockUntil = Date.now() + 3600000 // Lock for 1 hour
  
Successful login:
  loginAttempts = 0
  lockUntil = undefined
```

**Why 5 Attempts?**:
- 3 attempts: Too strict, users legitimately forget passwords
- 10 attempts: Too lenient, gives attackers more chances
- 5 attempts: Good balance - enough for typos, stops automated attacks

**Why 1 Hour?**:
- Long enough to deter brute-force (1 hour per 5 attempts = ~17 years for 1M passwords)
- Short enough that legitimate users aren't permanently locked out
- Automatically unlocks without admin intervention

**Checking Lock Status**:
```
Before checking password, verify lock:

if (user.isLocked()) {
  const minutesLeft = Math.ceil((user.lockUntil - Date.now()) / 60000);
  return res.status(423).json({
    error: `Account locked. Try again in ${minutesLeft} minutes`
  });
}
```

**Atomic Operations Prevent Race Conditions**:
```
Multiple failed logins at exact same time could corrupt counter

MongoDB atomic operation:
await User.updateOne(
  { _id: userId },
  { $inc: { loginAttempts: 1 } } // Atomic increment
);

Guarantees correct count even with simultaneous requests
```

#### Complete Login Flow Diagram

```
┌─────────────┐
│   CLIENT    │  User enters email + password
└──────┬──────┘
       │ 1. POST /api/auth/login
       │    { email, password }
       ↓
┌─────────────────────────────────────────┐
│    SERVER - Auth Controller             │
│  ┌───────────────────────────────────┐  │
│  │ 1. Find user by email             │  │
│  │    if (!user) return 401          │  │
│  └───────────────┬───────────────────┘  │
│                  ↓                       │
│  ┌───────────────────────────────────┐  │
│  │ 2. Check if account locked        │  │
│  │    if (locked) return 423         │  │
│  └───────────────┬───────────────────┘  │
│                  ↓                       │
│  ┌───────────────────────────────────┐  │
│  │ 3. Compare password with bcrypt   │  │
│  │    const match = await            │  │
│  │    user.comparePassword(password) │  │
│  └───────────────┬───────────────────┘  │
│                  ↓                       │
│         ┌────────┴────────┐             │
│         │                 │             │
│    No Match          Yes Match          │
│         │                 │             │
│    ┌────▼─────┐      ┌───▼──────────┐  │
│    │ Increment│      │Reset attempts│  │
│    │ attempts │      │Generate JWT  │  │
│    │ Lock if 5│      │Create session│  │
│    │Return 401│      │Update lastLog│  │
│    └──────────┘      └───┬──────────┘  │
└────────────────────────────┼────────────┘
                             │ 2. Response
                             │    {
                             │      accessToken: "eyJ...",
                             │      refreshToken: "abc...",
                             │      user: { id, email, username }
                             │    }
                             ↓
┌──────────────────────────────────────────┐
│   CLIENT - Store Tokens                  │
│  localStorage.setItem('accessToken', ...) │
│  localStorage.setItem('refreshToken', ...)│
│  useAuthStore.setAuth(user, tokens)      │
└──────┬───────────────────────────────────┘
       │ 3. Navigate to /dashboard
       │
       │ 4. Subsequent API requests
       │    Authorization: Bearer {accessToken}
       ↓
┌──────────────────────────────────────────┐
│    SERVER - Middleware Chain             │
│  ┌────────────────────────────────────┐  │
│  │ authenticateUser()                 │  │
│  │ 1. Extract token from header       │  │
│  │ 2. Verify JWT signature            │  │
│  │ 3. Check expiration                │  │
│  │ 4. Load user from DB               │  │
│  │ 5. Attach to req.user              │  │
│  └────────┬───────────────────────────┘  │
│           ↓                               │
│  ┌────────────────────────────────────┐  │
│  │ Controller (e.g., getProjects)     │  │
│  │ Can access req.user.id             │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

#### Token Refresh Mechanism

When an access token expires (every 15 minutes), the frontend automatically requests a new one:

```
┌─────────────┐
│   CLIENT    │  Making API request
└──────┬──────┘
       │ GET /api/projects
       │ Authorization: Bearer {expired_token}
       ↓
┌─────────────────┐
│    SERVER       │
│  Middleware     │  Detects expired token
│  Returns 401    │
└────────┬────────┘
         │ 401 Unauthorized
         ↓
┌─────────────────────────────────────────┐
│   CLIENT - Axios Response Interceptor   │
│  if (error.status === 401 && !retried)  │
│    1. Get refreshToken from localStorage│
│    2. POST /api/auth/refresh            │
│       { refreshToken }                  │
└──────┬──────────────────────────────────┘
       │
       ↓
┌─────────────────────────────────────────┐
│    SERVER - Refresh Endpoint            │
│  1. Validate refresh token              │
│  2. Check expiration (7 days)           │
│  3. Generate NEW access token (15 min)  │
│  4. Generate NEW refresh token (7 days) │
│  5. Invalidate old refresh token        │
│  6. Return both tokens                  │
└──────┬──────────────────────────────────┘
       │ { accessToken: "new...", refreshToken: "new..." }
       ↓
┌─────────────────────────────────────────┐
│   CLIENT - Update Tokens                │
│  1. Save new tokens to localStorage     │
│  2. Retry original failed request       │
│     with new accessToken                │
└──────┬──────────────────────────────────┘
       │ GET /api/projects (retry)
       │ Authorization: Bearer {new_token}
       ↓
┌─────────────────┐
│    SERVER       │
│  Request        │  Success - returns data
│  succeeds       │
└─────────────────┘
```

**Why Token Rotation?**
If someone steals your refresh token and uses it before you do, they get new tokens and yours become invalid. When you try to refresh, it fails, alerting you to the breach. This is called **automatic breach detection**.

### Token Structure Anatomy

JWT tokens consist of three parts separated by dots: `header.payload.signature`

**Access Token Decoded** (Short-lived: 15 minutes):
```
Header (algorithm + type):
{
  "alg": "HS256",     // HMAC SHA-256 algorithm
  "typ": "JWT"        // Token type
}

Payload (claims about the user):
{
  "sub": "6932be8dc2b07fc45336d4e2",  // Subject (user ID)
  "email": "user@example.com",       // User email
  "role": "user",                     // User role
  "iat": 1733606400,                  // Issued at timestamp
  "exp": 1733607300                   // Expires in 15 minutes
}

Signature (proves token hasn't been tampered):
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  JWT_SECRET
)
```

**Why these fields?**:
- `sub` (subject): Standard JWT claim for user identifier
- `iat` (issued at): Track when token was created
- `exp` (expiration): Automatic expiration enforcement
- `role`: Quick access control without database lookup

**Refresh Token** (Long-lived: 7 days):
Similar structure but stored hashed in database for revocation. Contains `type: "refresh"` to prevent misuse as access token.

### Authentication Middleware Chain

Every protected route passes through a middleware pipeline that validates requests before reaching controllers:

#### **1. authenticateUser Middleware**

**Purpose**: Verifies the JWT and loads user information

**How it works**:
```
Request arrives →
1. Check Authorization header exists
   If missing: return 401 "No token provided"

2. Extract token from "Bearer {token}" format
   Split header by space, take second part

3. Verify JWT signature with JWT_SECRET
   jwt.verify(token, process.env.JWT_SECRET)
   If invalid: return 401 "Invalid token"
   If expired: return 401 "Token expired"

4. Extract user ID from token payload (sub claim)

5. Query database for complete user object
   const user = await User.findById(tokenPayload.sub)
   If user deleted: return 401 "User not found"

6. Check if email is verified (for sensitive operations)
   If not verified: return 403 "Email verification required"

7. Attach user to request object
   req.user = user

8. Call next() to proceed to next middleware/controller
```

**Why query database if JWT is stateless?**: JWT contains basic info (ID, email), but we need current user state (is account locked? is email still verified? did user change role?). The token might be valid but the account could be disabled.

#### **2. checkTeamAccess Middleware**

**Purpose**: Enforces team-based permissions for collaborative projects

**How it works**:
```
Request for project operation →
1. Get project ID from request (params or body)

2. Load project from database
   const project = await Project.findById(projectId)

3. Check if user is project owner
   if (project.userId.equals(req.user._id)) {
     // Owner has full access
     return next()
   }

4. Check if project belongs to a team
   if (!project.teamId) {
     // Personal project, user not owner
     return 403 "Access denied"
   }

5. Load user's team membership
   const membership = await TeamMember.findOne({
     userId: req.user._id,
     teamId: project.teamId
   })

6. Validate membership exists
   if (!membership) {
     return 403 "Not a team member"
   }

7. Check required permission for operation
   Operation: DELETE → requires 'admin' role
   Operation: DEPLOY → requires 'deploy' permission
   Operation: EDIT   → requires 'edit' permission
   Operation: VIEW   → requires 'view' permission

8. If permission granted: next()
   If permission denied: 403 "Insufficient permissions"
```

**Permission Hierarchy**:
- **Admin**: Full access (edit, deploy, delete, manage members)
- **Developer**: Can edit code and deploy
- **Viewer**: Read-only access, can't modify

**Example**: Team member with "viewer" role tries to delete project:
```
Request: DELETE /api/projects/123
→ authenticateUser: ✓ Valid token
→ checkTeamAccess: 
    - User is team member ✓
    - Operation requires 'admin' permission
    - User role is 'viewer' ✗
    - Return 403 "Insufficient permissions"
```

---

## 📦 Project Management

### What is a Project?

A **project** in our system represents a deployable application - whether it's a React website, Vue app, static site, or Node.js API. Each project has its own:
- Unique subdomain for accessing the deployed application
- File storage for source code
- Build configuration based on framework type
- Deployment history tracking every version
- Environment variables for configuration
- Team collaboration features (if owned by a team)

Think of a project as a container that holds everything needed to build and deploy your application to a live URL.

### Project Lifecycle

Every project goes through five stages from creation to production:

```
CREATE → UPLOAD → BUILD → DEPLOY → MONITOR
  ↓        ↓        ↓        ↓         ↓
Setup   Add Code  Compile  Go Live  Track
```

**CREATE**: Set up project with name, framework, visibility
**UPLOAD**: Add source code files (single, multiple, or ZIP)
**BUILD**: Install dependencies, compile code, generate assets
**DEPLOY**: Copy built files to Nginx, configure subdomain
**MONITOR**: Track deployment status, view logs, check analytics

### 1. Project Creation

**Endpoint**: `POST /api/projects`

```javascript
// Request
{
  name: "My React App",
  description: "Portfolio website",
  framework: "react",
  visibility: "private",
  tags: ["react", "portfolio"],
  repositoryUrl: "https://github.com/user/repo" // optional
}

// Process
1. Validate project name (unique per user)
2. Generate subdomain: sanitize(name) + ".local"
3. Create Project document
4. Create ProjectSettings with framework defaults
5. Create project directory: /uploads/{projectId}/
6. Return project with subdomain

// Response
{
  message: "Project created successfully",
  project: {
    _id: "6932be8dc2b07fc45336d4e2",
    name: "My React App",
    subdomain: "my-react-app.local",
    deploymentUrl: "http://my-react-app.local",
    framework: "react",
    ...
  }
}
```

### 2. File Upload System

**Three Upload Methods**:

#### a) Single File Upload
**Endpoint**: `POST /api/projects/:id/files/upload`

```javascript
// Multipart form data
FormData: {
  file: File,
  path: "src/App.jsx" // optional relative path
}

// Process
1. Multer receives file
2. Validate: max 50MB per file
3. Store temporarily in /temp/
4. Upload to MinIO/S3
5. Create ProjectFile record
6. Update project totalFiles & totalSize
7. Delete temp file
```

#### b) Multiple Files Upload
**Endpoint**: `POST /api/projects/:id/files/upload-multiple`

```javascript
// Multipart form data
FormData: {
  files: [File, File, File]
}

// Process
1. Accept up to 50 files per request
2. Process each file independently
3. Parallel upload to MinIO
4. Bulk insert ProjectFile records
```

#### c) ZIP Upload with Auto-extraction
**Endpoint**: `POST /api/projects/:id/files/upload-zip`

```javascript
// Multipart form data
FormData: {
  zipfile: File // .zip file
}

// Process
1. Receive ZIP (max 100MB)
2. Extract to temp directory
3. Validate folder structure
4. Upload each file to MinIO
5. Create ProjectFile records for all
6. Detect framework if package.json found
7. Update project settings
8. Clean up temp files
```

**ZIP Extraction Logic** (`utils/zipExtractor.js`):
```javascript
1. Unzip file
2. Detect root folder structure:
   - If single top folder → use as root
   - If multiple items → treat as root
3. Walk directory tree
4. Skip: node_modules, .git, dist, build
5. Extract files preserving structure
6. Return file list with metadata
```

### 3. File Tree Structure

**ProjectFile Model**:
```javascript
{
  projectId: ObjectId,
  fileName: "App.jsx",
  filePath: "src/components/App.jsx", // full path
  fileSize: 1024,
  mimeType: "text/javascript",
  s3Key: "projects/6932be8d.../src/components/App.jsx",
  s3Bucket: "projects",
  isPublic: false,
  version: 1
}
```

**File Tree API Response**:
```javascript
// GET /api/projects/:id/files
{
  files: [
    {
      _id: "...",
      fileName: "package.json",
      filePath: "package.json",
      fileSize: 2048,
      type: "file"
    },
    {
      fileName: "src",
      type: "folder",
      children: [
        { fileName: "App.jsx", filePath: "src/App.jsx", ... }
      ]
    }
  ],
  totalFiles: 15,
  totalSize: 125440
}
```

### 4. Storage Quota System

**User Limits**:
- Free Tier: 500 MB
- Pro Tier: 10 GB
- Enterprise: Unlimited

**Quota Checking** (`utils/storageQuota.js`):

```javascript
class StorageQuota {
  async checkQuota(userId, fileSize) {
    // 1. Get user's current usage
    const usage = await this.getUserStorage(userId);
    
    // 2. Get user's plan limit
    const limit = user.plan === 'free' ? 500 * 1024 * 1024 : 
                  user.plan === 'pro' ? 10 * 1024 * 1024 * 1024 : Infinity;
    
    // 3. Check if adding file exceeds limit
    if (usage + fileSize > limit) {
      throw new Error('Storage quota exceeded');
    }
    
    return true;
  }
  
  async getUserStorage(userId) {
    // Aggregate all ProjectFile sizes for user's projects
    const result = await ProjectFile.aggregate([
      { $match: { userId: ObjectId(userId) } },
      { $group: { _id: null, total: { $sum: '$fileSize' } } }
    ]);
    return result[0]?.total || 0;
  }
}
```

### 5. Template System

**Template Model**:
```javascript
{
  name: "React Starter",
  description: "Basic React app with routing",
  category: "frontend",
  framework: "react",
  thumbnail: "https://...",
  sourceUrl: "https://github.com/...",
  deployCount: 150,
  rating: 4.5,
  tags: ["react", "starter", "tailwind"],
  featured: true,
  files: [...] // Pre-configured file structure
}
```

**Deploy from Template Flow**:
```
1. User selects template
2. POST /api/templates/:id/deploy
3. Create new project
4. Clone template files to project
5. Replace placeholders (project name, etc.)
6. Auto-trigger first deployment
7. Return project ID
```

---

## 🚢 Deployment Pipeline

### Complete Deployment Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT TRIGGER                             │
│  User clicks "Deploy" → POST /api/deployments/deploy/:projectId  │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                             ↓
┌──────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT CONTROLLER                          │
│  1. Create Deployment record (status: 'queued')                  │
│  2. Get project files from database                              │
│  3. Load environment variables                                   │
│  4. Create build configuration                                   │
│  5. Add job to Bull queue                                        │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                             ↓
┌──────────────────────────────────────────────────────────────────┐
│                      BULL QUEUE (Redis)                           │
│  Job: { deploymentId, projectId, files, config, envVars }       │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                             ↓
┌──────────────────────────────────────────────────────────────────┐
│                      BUILD WORKER                                 │
│  1. Update status: 'building'                                    │
│  2. Create temp build directory                                  │
│  3. Download files from MinIO/S3                                 │
│  4. Create .env file with variables                              │
│  5. Detect package.json → run npm install                        │
│  6. Run build command (npm run build)                            │
│  7. Find build output directory                                  │
│  8. Create ZIP of built files                                    │
│  9. Upload to MinIO                                              │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                             ↓
┌──────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT PHASE                               │
│  1. Update status: 'deploying'                                   │
│  2. Create deployment directory: /public/deployments/{subdomain} │
│  3. Extract ZIP to deployment directory                          │
│  4. Generate Nginx configuration                                 │
│  5. Reload Nginx                                                 │
│  6. Update status: 'success'                                     │
│  7. Set isActive: true                                           │
│  8. Deactivate previous deployments                              │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                             ↓
┌──────────────────────────────────────────────────────────────────┐
│                    POST-DEPLOYMENT                                │
│  1. Health check: GET http://{subdomain}/                        │
│  2. Update project.lastDeployedAt                                │
│  3. Emit Socket.IO event: deployment-complete                    │
│  4. Send notification to user                                    │
│  5. Clean up temp files                                          │
└──────────────────────────────────────────────────────────────────┘
```

### Build Worker Deep Dive

**File**: `backend/task3-deployment/workers/buildWorker.js`

```javascript
// 1. JOB RECEIPT
buildQueue.process(async (job, done) => {
  const { deploymentId, projectId, files, config, envVars } = job.data;
  
  // 2. SETUP
  const buildPath = `/builds/${deploymentId}`;
  await fs.mkdir(buildPath, { recursive: true });
  
  // 3. FILE PREPARATION
  for (const file of files) {
    // Download from MinIO
    const fileData = await minioManager.downloadFile(file.s3Key);
    // Write to build directory
    await fs.writeFile(`${buildPath}/${file.filePath}`, fileData);
  }
  
  // 4. ENVIRONMENT VARIABLES
  const envContent = Object.entries(envVars)
    .map(([key, val]) => `${key}=${val}`)
    .join('\n');
  await fs.writeFile(`${buildPath}/.env`, envContent);
  
  // 5. FRAMEWORK DETECTION
  const hasPackageJson = await fileExists(`${buildPath}/package.json`);
  if (!hasPackageJson && framework !== 'static') {
    await createBasicPackageJson(buildPath);
  }
  
  // 6. DEPENDENCY INSTALLATION
  if (config.installCommand) {
    await execCommand(config.installCommand, buildPath);
    // Typically: npm install or yarn install
  }
  
  // 7. BUILD EXECUTION
  if (config.buildCommand) {
    await execCommand(config.buildCommand, buildPath);
    // Typically: npm run build or yarn build
  }
  
  // 8. OUTPUT DETECTION (Smart Logic)
  const outputDir = await findBuildOutput(buildPath, config.outputDir);
  // Searches: dist, build, out, .next, public, www, output
  
  // 9. ZIP CREATION
  const zipPath = `${buildPath}/deployment.zip`;
  await createZip(outputDir, zipPath);
  
  // 10. UPLOAD TO STORAGE
  const s3Key = await minioManager.uploadFile(zipPath, `deployments/${deploymentId}.zip`);
  
  // 11. CLEANUP
  await fs.rm(buildPath, { recursive: true });
  
  done(null, { s3Key, outputDir });
});
```

### Nginx Configuration

**Dynamic Nginx Config Generation** (`utils/nginxConfig.js`):

```nginx
# Auto-generated for subdomain: my-app.local

server {
    listen 80;
    server_name my-app.local;
    
    # Root directory
    root C:/path/to/backend/public/deployments/my-app.local;
    index index.html index.htm;
    
    # SPA fallback
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Static files caching
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
}
```

**Windows Nginx Management** (`utils/windowsNginxManager.js`):
```javascript
class WindowsNginxManager {
  async reloadConfig() {
    // Write config file
    await fs.writeFile(
      'C:/innodev/nginx/conf/sites-enabled/project.conf',
      configContent
    );
    
    // Test configuration
    await exec('cd C:\\innodev\\nginx && nginx.exe -t');
    
    // Reload Nginx
    await exec('cd C:\\innodev\\nginx && nginx.exe -s reload');
  }
  
  async initialize() {
    // Start Nginx if not running
    const isRunning = await this.isNginxRunning();
    if (!isRunning) {
      await exec('cd C:\\innodev\\nginx && start nginx.exe');
    }
  }
}
```

### Framework-Specific Build Commands

```javascript
const BUILD_CONFIGS = {
  react: {
    installCommand: 'npm install',
    buildCommand: 'npm run build',
    outputDir: 'build',
    devCommand: 'npm run dev'
  },
  nextjs: {
    installCommand: 'npm install',
    buildCommand: 'npm run build',
    outputDir: '.next',
    devCommand: 'npm run dev',
    serverSide: true
  },
  vue: {
    installCommand: 'npm install',
    buildCommand: 'npm run build',
    outputDir: 'dist',
    devCommand: 'npm run serve'
  },
  static: {
    installCommand: null,
    buildCommand: null,
    outputDir: '.',
    devCommand: null
  },
  node: {
    installCommand: 'npm install',
    buildCommand: null,
    outputDir: '.',
    startCommand: 'node index.js',
    serverSide: true
  }
};
```

### Real-time Build Logs

**Socket.IO Implementation**:

```javascript
// Backend - Emit logs during build
io.to(`deployment-${deploymentId}`).emit('build-log', {
  message: '📦 Installing dependencies...',
  type: 'info',
  timestamp: new Date()
});

// Frontend - Listen to logs
socket.on('build-log', (log) => {
  setLogs(prev => [...prev, log]);
});
```

---

## 🎨 Frontend Architecture

### State Management Strategy

```
┌─────────────────────────────────────────────────────────┐
│                   STATE LAYERS                          │
├─────────────────────────────────────────────────────────┤
│  1. Global State (Zustand + Persist)                   │
│     - Authentication (user, tokens)                     │
│     - Theme preferences                                 │
│     └─ Persisted to localStorage                        │
├─────────────────────────────────────────────────────────┤
│  2. Server State (TanStack Query)                       │
│     - Projects list                                     │
│     - Deployments                                       │
│     - User profile                                      │
│     └─ Cached with auto-refetch                         │
├─────────────────────────────────────────────────────────┤
│  3. Component State (useState/useReducer)               │
│     - Form inputs                                       │
│     - Modal visibility                                  │
│     - Local UI state                                    │
└─────────────────────────────────────────────────────────┘
```

### Zustand Store - Authentication

**File**: `frontend/src/store/authStore.js`

```javascript
const useAuthStore = create(
  persist(
    (set) => ({
      // State
      user: null,
      accessToken: null,
      refreshToken: null,
      isAuthenticated: false,
      
      // Actions
      setAuth: (user, accessToken, refreshToken) => {
        set({ user, accessToken, refreshToken, isAuthenticated: true });
        localStorage.setItem('accessToken', accessToken);
        localStorage.setItem('refreshToken', refreshToken);
      },
      
      logout: () => {
        set({
          user: null,
          accessToken: null,
          refreshToken: null,
          isAuthenticated: false
        });
        localStorage.clear();
      },
      
      updateTokens: (accessToken, refreshToken) => {
        set({ accessToken, refreshToken });
        localStorage.setItem('accessToken', accessToken);
      }
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        isAuthenticated: state.isAuthenticated
      })
    }
  )
);
```

### React Router Configuration

```javascript
// frontend/src/App.jsx
<Router>
  <Routes>
    {/* Public Routes */}
    <Route path="/login" element={
      <PublicRoute><Login /></PublicRoute>
    } />
    <Route path="/signup" element={
      <PublicRoute><Signup /></PublicRoute>
    } />
    
    {/* Protected Routes */}
    <Route path="/dashboard" element={
      <ProtectedRoute><Dashboard /></ProtectedRoute>
    } />
    <Route path="/projects" element={
      <ProtectedRoute><Projects /></ProtectedRoute>
    } />
    <Route path="/projects/:id" element={
      <ProtectedRoute><ProjectDetail /></ProtectedRoute>
    } />
    
    {/* Redirects */}
    <Route path="/" element={<Navigate to="/dashboard" />} />
    <Route path="*" element={<Navigate to="/dashboard" />} />
  </Routes>
</Router>
```

### Route Guards

**ProtectedRoute Component**:
```javascript
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuthStore();
  const location = useLocation();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}
```

**PublicRoute Component**:
```javascript
function PublicRoute({ children }) {
  const { isAuthenticated } = useAuthStore();
  
  if (isAuthenticated) {
    return <Navigate to="/dashboard" replace />;
  }
  
  return children;
}
```

### API Service Layer

**Base API Client** (`frontend/src/lib/api.js`):

```javascript
const api = axios.create({
  baseURL: 'http://localhost:5000/api',
  headers: { 'Content-Type': 'application/json' }
});

// Request interceptor
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor - Token refresh
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      
      const refreshToken = localStorage.getItem('refreshToken');
      const response = await axios.post('/api/auth/refresh', { refreshToken });
      
      localStorage.setItem('accessToken', response.data.accessToken);
      return api(error.config);
    }
    return Promise.reject(error);
  }
);
```

**Service Module Example** (`frontend/src/services/projectService.js`):

```javascript
const projectService = {
  async getProjects(params) {
    const response = await api.get('/projects', { params });
    return response.data;
  },
  
  async createProject(data) {
    const response = await api.post('/projects', data);
    return response.data;
  },
  
  async uploadFile(projectId, file) {
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await api.post(
      `/projects/${projectId}/files/upload`,
      formData,
      { headers: { 'Content-Type': 'multipart/form-data' } }
    );
    return response.data;
  }
};
```

### Component Structure

```
src/
├── components/
│   ├── ui/                 # Shadcn-style base components
│   │   ├── button.jsx
│   │   ├── card.jsx
│   │   ├── input.jsx
│   │   └── select.jsx
│   │
│   ├── projects/           # Project-specific components
│   │   ├── CreateProjectModal.jsx
│   │   ├── ProjectCard.jsx
│   │   └── EnvVarsManager.jsx
│   │
│   ├── files/              # File management components
│   │   ├── FileUploader.jsx
│   │   ├── FileTree.jsx
│   │   └── FilePreviewModal.jsx
│   │
│   └── storage/
│       └── StorageQuota.jsx
│
├── pages/                  # Full page components
│   ├── Login.jsx
│   ├── Dashboard.jsx
│   ├── Projects.jsx
│   └── ProjectDetail.jsx
│
├── services/               # API service modules
├── store/                  # Zustand stores
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities
└── routes/                 # Route guards
```

---

## 📡 API Reference

### Authentication Endpoints

```
POST   /api/auth/signup
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
POST   /api/auth/forgot-password
POST   /api/auth/reset-password/:token
GET    /api/auth/verify-email/:token
```

### Project Endpoints

```
GET    /api/projects
POST   /api/projects
GET    /api/projects/:id
PUT    /api/projects/:id
DELETE /api/projects/:id
POST   /api/projects/:id/duplicate
POST   /api/projects/:id/archive
POST   /api/projects/:id/favorite
GET    /api/projects/search
```

### File Management Endpoints

```
POST   /api/projects/:id/files/upload
POST   /api/projects/:id/files/upload-multiple
POST   /api/projects/:id/files/upload-zip
GET    /api/projects/:id/files
GET    /api/projects/:id/files/:fileId
DELETE /api/projects/:id/files/:fileId
GET    /api/projects/:id/files/:fileId/download
```

### Deployment Endpoints

```
POST   /api/deployments/deploy/:projectId
GET    /api/deployments/project/:projectId
GET    /api/deployments/:id
GET    /api/deployments/:id/logs
POST   /api/deployments/:id/rollback
POST   /api/deployments/:id/cancel
GET    /api/deployments/stats/:projectId
```

### Storage & Quota

```
GET    /api/storage/quota
GET    /api/storage/analytics
```

---

## 🔒 Security Implementation

### 1. Password Security
- **Hashing**: bcryptjs with 12 salt rounds
- **Validation**: Minimum 8 characters, complexity rules
- **Reset tokens**: Cryptographically secure, 1-hour expiry

### 2. JWT Security
- **Access tokens**: Short-lived (15 minutes)
- **Refresh tokens**: Hashed in database, 7-day expiry
- **Token rotation**: New refresh token on refresh

### 3. Rate Limiting

```javascript
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later'
});

app.use('/api/auth/login', authLimiter);
```

### 4. Account Locking
- 5 failed login attempts → 1-hour lock
- Automatically unlocks after timeout
- Admin can manually unlock

### 5. Input Validation
- Zod schemas on frontend
- Mongoose validation on backend
- SQL injection prevention (NoSQL → MongoDB)
- XSS prevention (helmet middleware)

### 6. CORS Configuration

```javascript
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## 📂 File Storage System

### MinIO/S3 Integration

**Configuration**:
```javascript
const minioClient = new Minio.Client({
  endPoint: process.env.MINIO_ENDPOINT,
  port: parseInt(process.env.MINIO_PORT),
  useSSL: process.env.MINIO_USE_SSL === 'true',
  accessKey: process.env.MINIO_ACCESS_KEY,
  secretKey: process.env.MINIO_SECRET_KEY
});
```

**Bucket Structure**:
```
projects/
  └── {projectId}/
      ├── source/
      │   └── {fileName}
      └── deployments/
          └── {deploymentId}.zip

avatars/
  └── {userId}.{ext}

templates/
  └── {templateId}/
```

**Upload Flow**:
```javascript
async uploadFile(file, path) {
  const bucketName = 'projects';
  const objectName = `${projectId}/source/${file.originalname}`;
  
  await minioClient.putObject(
    bucketName,
    objectName,
    file.buffer,
    file.size,
    { 'Content-Type': file.mimetype }
  );
  
  return { bucket: bucketName, key: objectName };
}
```

---

## 🎯 What We're Building

**Eleven's Deployment Hub** is a production-ready, full-stack deployment platform that:

1. **Authenticates users** with JWT tokens, email verification, and team collaboration
2. **Manages projects** with file uploads, templates, and storage quotas
3. **Builds applications** using Docker containers and npm/yarn
4. **Deploys to Nginx** with auto-generated configurations and custom domains
5. **Monitors health** with real-time logs and status updates
6. **Provides analytics** with deployment stats and AI code analysis

### Current Implementation Status

✅ **Complete (85%)**:
- Authentication & user management
- Project CRUD operations
- File upload/management
- Storage quota system
- Template gallery
- Build queue system
- Worker processing
- Nginx deployment

🚧 **In Progress (15%)**:
- Custom domains & SSL
- Analytics dashboard
- GitHub integration
- Webhook system
- AI code analysis

---

## 📝 Summary

This platform follows industry-standard patterns:
- **MVC architecture** for clean separation of concerns
- **JWT authentication** with refresh token rotation
- **Queue-based** async job processing
- **Real-time updates** via WebSockets
- **Cloud storage** integration (S3/MinIO)
- **Docker-based** containerized builds
- **Nginx** reverse proxy for deployments

The system is designed to scale horizontally with multiple worker processes and Redis-backed queues, making it production-ready for deployment platforms like Vercel or Netlify.

---

*Generated on December 7, 2025*
