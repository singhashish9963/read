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

### Design Patterns

1. **MVC Pattern**: Controllers → Services → Models
2. **Repository Pattern**: Data access abstraction
3. **Middleware Pattern**: Request/response processing pipeline
4. **Observer Pattern**: Socket.IO for real-time updates
5. **Queue Pattern**: Bull queues for async job processing
6. **Factory Pattern**: Build configuration generation

---

## 🛠️ Technology Stack

### Backend Technologies

```javascript
{
  // Core Framework
  "express": "^4.18.2",          // Web framework
  "mongoose": "^8.0.3",          // MongoDB ODM
  
  // Authentication & Security
  "jsonwebtoken": "^9.0.2",      // JWT tokens
  "bcryptjs": "^2.4.3",          // Password hashing
  "helmet": "^7.1.0",            // Security headers
  "express-rate-limit": "^7.1.5", // Rate limiting
  
  // File Storage & Processing
  "multer": "^1.4.5-lts.1",      // File uploads
  "minio": "^7.1.3",             // S3-compatible storage
  "sharp": "^0.33.1",            // Image processing
  "archiver": "^6.0.1",          // ZIP creation
  "unzipper": "^0.11.4",         // ZIP extraction
  
  // Queue & Background Jobs
  "bull": "^4.12.0",             // Redis-based queue
  "node-cron": "^4.2.1",         // Scheduled tasks
  
  // Real-time & Email
  "socket.io": "^4.8.1",         // WebSocket
  "nodemailer": "^6.9.7",        // Email service
  
  // AI & External Services
  "@google/generative-ai": "^0.1.3", // Gemini AI
  "@octokit/rest": "^22.0.1",    // GitHub API
  "simple-git": "^3.30.0",       // Git operations
  
  // Utils
  "nanoid": "^3.3.7",            // Unique ID generation
  "axios": "^1.6.2"              // HTTP client
}
```

### Frontend Technologies

```javascript
{
  // Core Framework
  "react": "^19.2.0",
  "react-dom": "^19.2.0",
  "vite": "^7.2.4",
  
  // Routing & State
  "react-router-dom": "^7.10.1",
  "zustand": "^5.0.9",           // State management
  "@tanstack/react-query": "^5.90.12", // Server state
  
  // Styling
  "tailwindcss": "^4.1.17",
  "@tailwindcss/vite": "^4.1.17",
  "clsx": "^2.1.1",
  "tailwind-merge": "^3.4.0",
  "class-variance-authority": "^0.7.1",
  
  // Forms & Validation
  "react-hook-form": "^7.68.0",
  "@hookform/resolvers": "^5.2.2",
  "zod": "^4.1.13",
  
  // UI & UX
  "lucide-react": "^0.556.0",    // Icons
  "framer-motion": "^12.23.25",  // Animations
  "react-hot-toast": "^2.6.0",   // Notifications
  "date-fns": "^4.1.0",          // Date formatting
  
  // Real-time
  "socket.io-client": "^4.8.1",
  
  // HTTP
  "axios": "^1.13.2"
}
```

---

## 🗄️ Database Schema & Models

### 1. User Model (`task1_auth/models/User.js`)

**Purpose**: Core user authentication and profile management

```javascript
{
  username: String (unique, 3-30 chars),
  email: String (unique, validated),
  password: String (hashed with bcrypt, salt rounds: 12),
  avatar: String (URL to avatar image),
  bio: String (max 500 chars),
  role: Enum ['user', 'admin'],
  
  // Email Verification
  isEmailVerified: Boolean,
  emailVerificationToken: String,
  emailVerificationExpires: Date,
  
  // Password Reset
  passwordResetToken: String,
  passwordResetExpires: Date,
  
  // Two-Factor Auth (Future)
  twoFactorEnabled: Boolean,
  twoFactorSecret: String,
  
  // Security
  apiKey: String,
  lastLogin: Date,
  loginAttempts: Number,
  lockUntil: Date,
  isActive: Boolean,
  
  timestamps: true (createdAt, updatedAt)
}
```

**Key Methods**:
- `comparePassword(candidatePassword)`: Verify password using bcrypt
- `isLocked()`: Check if account is locked due to failed attempts
- `incLoginAttempts()`: Increment login attempts counter

**Security Features**:
- Password hashing with bcrypt (12 rounds)
- Account locking after 5 failed attempts (1 hour)
- Email verification required
- Token-based password reset

---

### 2. Project Model (`task2-projects/models/Project.js`)

**Purpose**: Represents a deployable project/application

```javascript
{
  name: String (3-100 chars),
  description: String (max 1000 chars),
  
  // Subdomain & Deployment
  subdomain: String (unique, format: xxx.local),
  deploymentUrl: String,
  customDomain: String (optional),
  
  // Ownership
  userId: ObjectId (ref: User, indexed),
  teamId: ObjectId (ref: Team, optional, indexed),
  
  // Configuration
  framework: Enum [
    'react', 'vue', 'angular', 'next', 'nuxt', 
    'svelte', 'static', 'node', 'express', 'other'
  ],
  visibility: Enum ['public', 'private'],
  status: Enum ['active', 'archived', 'draft'],
  
  // Metadata
  tags: [String],
  isFavorite: Boolean,
  repositoryUrl: String (GitHub URL),
  thumbnail: String,
  
  // Stats
  totalFiles: Number,
  totalSize: Number (bytes),
  lastDeployedAt: Date,
  viewCount: Number,
  
  // Cloning
  clonedFrom: ObjectId (ref: Project),
  
  timestamps: true
}
```

**Subdomain Validation**:
- Must be lowercase
- Pattern: `[a-z0-9]([a-z0-9-]*[a-z0-9])?.local`
- Example: `my-app.local`, `project123.local`

**Indexes**:
- `userId` (for fast user project lookup)
- `teamId` (for team projects)
- `subdomain` (unique constraint)

---

### 3. Deployment Model (`task3-deployment/models/Deployment.js`)

**Purpose**: Tracks deployment history and build status

```javascript
{
  projectId: ObjectId (ref: Project, indexed),
  userId: ObjectId (ref: User),
  version: String,
  
  // Status & Lifecycle
  status: Enum [
    'queued', 'building', 'deploying', 
    'success', 'failed', 'cancelled'
  ],
  
  // Build Logs
  buildLogs: [{
    message: String,
    type: Enum ['info', 'warning', 'error', 'success', 'command'],
    timestamp: Date
  }],
  
  // Build Configuration
  framework: String,
  buildCommand: String,
  installCommand: String,
  outputDir: String,
  nodeVersion: String (default: '20.x'),
  
  // URLs & Storage
  deploymentUrl: String,
  subdomain: String,
  filesPath: String (local path),
  minioKey: String (S3 path),
  
  // Performance Metrics
  buildDuration: Number (seconds),
  buildSize: Number (bytes),
  startedAt: Date,
  completedAt: Date,
  
  // Error Tracking
  error: {
    message: String,
    stack: String,
    code: String
  },
  
  // Retry & Rollback
  attempts: Number,
  rollbackFrom: ObjectId (ref: Deployment),
  
  // Status
  isActive: Boolean,
  healthCheckStatus: Enum ['pending', 'healthy', 'unhealthy'],
  
  timestamps: true
}
```

**Methods**:
- `addLog(message, type)`: Append log entry
- `virtual duration`: Calculate build duration

**Indexes**:
- `{ projectId: 1, createdAt: -1 }` (project deployment history)
- `{ userId: 1, status: 1 }` (user deployments by status)
- `{ status: 1, createdAt: -1 }` (global status queries)

---

### 4. Session Model (`task1_auth/models/Session.js`)

**Purpose**: JWT refresh token management

```javascript
{
  userId: ObjectId (ref: User, indexed),
  refreshToken: String (hashed, unique),
  deviceInfo: {
    userAgent: String,
    ip: String,
    device: String,
    browser: String
  },
  isActive: Boolean,
  expiresAt: Date,
  lastUsedAt: Date,
  
  timestamps: true
}
```

---

### 5. Additional Models

**Team Model**: Multi-user collaboration
**TeamMember Model**: User roles in teams
**ProjectFile Model**: File metadata and S3 references
**ProjectSettings Model**: Build configuration per project
**Template Model**: One-click deploy templates
**ActivityLog Model**: Audit trail
**Notification Model**: In-app notifications
**Environment Model**: Environment variables
**HealthCheck Model**: Deployment health monitoring
**BuildQueue Model**: Queue management

---

## 🔐 Authentication System

### JWT Token Flow

```
┌─────────────┐
│   CLIENT    │
└──────┬──────┘
       │ 1. POST /api/auth/login
       │    { email, password }
       ↓
┌─────────────────┐
│    SERVER       │
│  ┌───────────┐  │
│  │Controller │  │
│  └─────┬─────┘  │
│        ↓        │
│  ┌───────────┐  │
│  │Verify     │  │
│  │Password   │  │
│  └─────┬─────┘  │
│        ↓        │
│  ┌───────────┐  │
│  │Generate   │  │
│  │Tokens     │  │
│  └─────┬─────┘  │
└────────┼────────┘
         │ 2. Response
         │    { accessToken, refreshToken, user }
         ↓
┌─────────────┐
│   CLIENT    │
│ localStorage│
│ - accessToken   (15 min)
│ - refreshToken  (7 days)
└──────┬──────┘
       │ 3. Subsequent requests
       │    Authorization: Bearer <accessToken>
       ↓
┌─────────────────┐
│    SERVER       │
│  Middleware     │
│  authenticateUser()
│  - Verify token │
│  - Attach user  │
└─────────────────┘
```

### Token Structure

**Access Token** (Short-lived: 15 minutes)
```javascript
{
  sub: userId,
  email: "user@example.com",
  role: "user",
  iat: 1733606400,
  exp: 1733607300  // 15 min
}
```

**Refresh Token** (Long-lived: 7 days)
```javascript
{
  sub: userId,
  type: "refresh",
  iat: 1733606400,
  exp: 1734211200  // 7 days
}
```

### Authentication Middleware

**`authenticateUser` Middleware** (`task1_auth/middleware/auth.js`):

```javascript
// Extracts JWT from Authorization header
// Verifies token signature
// Attaches user object to req.user
// Returns 401 if invalid/expired
```

**`checkTeamAccess` Middleware** (`task1_auth/middleware/teamAccess.js`):

```javascript
// Validates user has required permissions
// Checks team membership
// Validates project ownership
// Permissions: ['view', 'edit', 'deploy', 'admin']
```

### Token Refresh Flow

```javascript
// frontend/src/lib/api.js - Response Interceptor
if (error.response?.status === 401 && !originalRequest._retry) {
  1. Detect 401 error
  2. Get refreshToken from localStorage
  3. POST /api/auth/refresh
  4. Receive new accessToken
  5. Update localStorage
  6. Retry original request
  
  if refresh fails:
    - Clear all tokens
    - Redirect to /login
}
```

### Registration Flow

```
1. User submits signup form
2. Server validates input
3. Check if email/username exists
4. Hash password (bcrypt, 12 rounds)
5. Generate email verification token
6. Save user to database
7. Send verification email
8. Return success (email verification required)

Verification:
1. User clicks link in email
2. GET /api/auth/verify-email/:token
3. Server validates token & expiry
4. Set isEmailVerified = true
5. Redirect to login
```

### Password Reset Flow

```
Request Reset:
1. POST /api/auth/forgot-password { email }
2. Generate reset token (valid 1 hour)
3. Send email with reset link
4. Token stored in passwordResetToken

Reset Password:
1. GET /verify link (validate token)
2. POST /api/auth/reset-password/:token { newPassword }
3. Hash new password
4. Clear reset token
5. Invalidate all sessions
6. Force re-login
```

---

## 📦 Project Management

### Project Lifecycle

```
CREATE → UPLOAD → BUILD → DEPLOY → MONITOR
```

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
