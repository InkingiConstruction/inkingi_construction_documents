# InkingiPro - Complete Database Design & API Structure

**Version:** 1.0
**Status:** Final
**Date:** May 22, 2026
**Database:** PostgreSQL 15+
**ORM:** Prisma
**API:** REST (Node.js + Express)

---

## Part 1: Complete Database Schema Reference

### 1.1 All Tables in the System (18 Tables Total)

| # | Table Name | Purpose | Role Access |
|---|------------|---------|-------------|
| 1 | `users` | All user accounts | All roles |
| 2 | `user_sessions` | Active login sessions | System |
| 3 | `kyc_documents` | KYC upload records | All + Admin |
| 4 | `projects` | Construction projects | Client, Engineer, Supervisor, Supplier, Admin |
| 5 | `project_members` | User assignments to projects | System |
| 6 | `milestones` | Project milestones/payment gates | Client, Engineer, Supervisor, Admin |
| 7 | `boq_items` | Bill of Quantities line items | Engineer, Client, Admin |
| 8 | `escrow_accounts` | Escrow balance per project | Client, Engineer, Admin |
| 9 | `transactions` | All financial transactions | Client, Engineer, Admin |
| 10 | `rfqs` | Request for Quotes (supply chain) | Engineer, Supplier, Admin |
| 11 | `quotes` | Supplier quotes for RFQs | Supplier, Engineer, Admin |
| 12 | `purchase_orders` | Approved purchase orders | Engineer, Supplier, Admin |
| 13 | `deliveries` | Material delivery tracking | Supplier, Engineer, Supervisor, Admin |
| 14 | `progress_photos` | Daily progress photos/videos | Engineer, Client, Admin |
| 15 | `inspections` | Supervisor inspection reports | Supervisor, Client, Engineer, Admin |
| 16 | `disputes` | Dispute cases | All roles + Admin |
| 17 | `dispute_evidence` | Evidence uploaded for disputes | All roles + Admin |
| 18 | `audit_logs` | System audit trail | Admin only |
| 19 | `notifications` | Push/email/SMS notification queue | System |
| 20 | `system_settings` | Platform configuration | Admin only |

---

### 1.2 Additional Models Not in Original SRS (Added)

Based on real-world implementation needs, these models are **missing** from the original SRS and are added here:

| Added Table | Why It's Needed |
|-------------|-----------------|
| `user_sessions` | Track active sessions for security (logout, force logout) |
| `project_members` | Many-to-many relationship for project assignments (supports multiple engineers/supervisors per project) |
| `purchase_orders` | Separate from quotes for audit trail after selection |
| `dispute_evidence` | One-to-many evidence per dispute (not just JSONB array) |
| `system_settings` | Runtime configuration without redeployment |
| `refresh_tokens` | JWT refresh token management (Better Auth replacement) |
| `password_resets` | Track password reset requests with expiry |
| `email_templates` | Editable email templates for admin |
| `api_keys` | For supplier/external system integration |
| `activity_logs` | User activity tracking (lighter than audit_logs) |

---

## Part 2: Prisma Schema Models (Complete)

```prisma
// ============================================================================
// 📄 FILE HEADER COMMENT
// ============================================================================
// FILE NAME        : schema.prisma
// WHAT THIS FILE DOES : Complete database schema for InkingiPro platform
// HOW IT DOES IT      : Defines 20+ tables with relations, indexes, and enums
// DATA SOURCE         : None - this is the source of truth for database structure
// DATA DESTINATION    : Migrated to PostgreSQL via `prisma migrate dev`
// PRINCIPLE APPLIED   : DRY (Centralized schema used by all services)
// ============================================================================

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================================
// ENUMS (Shared across multiple tables)
// ============================================================================

enum UserRole {
  client
  engineer
  supervisor
  supplier
  admin
}

enum KYCStatus {
  pending
  submitted
  approved
  rejected
  additional_info_requested
}

enum ProjectStatus {
  draft
  active
  paused
  completed
  terminated
}

enum MilestoneStatus {
  pending
  active
  completed
  revision_required
  paid
}

enum TransactionType {
  deposit
  release
  refund
  freeze
  unfreeze
}

enum TransactionStatus {
  pending
  completed
  failed
  reversed
}

enum DisputeStatus {
  open
  under_review
  resolved_full_payment
  resolved_partial
  resolved_refund
  resolved_termination
  closed
}

enum NotificationChannel {
  email
  push
  sms
  in_app
}

enum NotificationStatus {
  pending
  sent
  delivered
  failed
  read
}

// ============================================================================
// TABLE 1: users
// ============================================================================

model User {
  id                       String              @id @default(uuid())
  email                    String              @unique
  phone                    String              @unique
  password_hash            String
  full_name                String
  role                     UserRole
  profile_picture_url      String?
  
  // KYC Fields
  kyc_status               KYCStatus           @default(pending)
  kyc_submitted_at         DateTime?
  kyc_reviewed_at          DateTime?
  kyc_reviewed_by          String?
  kyc_rejection_reason     String?
  
  // Engineer-specific
  ier_license_number       String?
  ier_license_expiry       DateTime?
  ier_license_verified     Boolean             @default(false)
  professional_indemnity_url String?
  
  // Supplier-specific
  business_registration_number String?
  tax_compliance_number    String?
  business_registration_url String?
  tax_certificate_url      String?
  
  // Verification flags
  email_verified           Boolean             @default(false)
  phone_verified           Boolean             @default(false)
  
  // Account status
  is_active                Boolean             @default(true)
  is_suspended             Boolean             @default(false)
  suspension_reason        String?
  suspended_until          DateTime?
  
  // Preferences
  notification_preferences Json                @default("{}")
  locale                   String              @default("en")
  currency_preference      String              @default("RWF")
  
  // Timestamps
  created_at               DateTime            @default(now())
  updated_at               DateTime            @updatedAt
  last_login_at            DateTime?
  
  // Relations
  projects_as_client       Project[]           @relation("ClientProjects")
  projects_as_engineer     Project[]           @relation("EngineerProjects")
  project_members          ProjectMember[]
  kyc_documents            KYCDocument[]
  milestones_created       Milestone[]         @relation("CreatedBy")
  transactions             Transaction[]
  rfqs_created             RFQ[]               @relation("RFQCreatedBy")
  quotes_submitted         Quote[]             @relation("QuoteSubmittedBy")
  purchase_orders          PurchaseOrder[]
  deliveries               Delivery[]
  progress_photos          ProgressPhoto[]
  inspections              Inspection[]
  disputes_raised          Dispute[]           @relation("DisputeRaisedBy")
  dispute_evidence         DisputeEvidence[]
  audit_logs               AuditLog[]
  notifications            Notification[]
  sessions                 UserSession[]
  refresh_tokens           RefreshToken[]
  password_resets          PasswordReset[]
  api_keys                 ApiKey[]
  activity_logs            ActivityLog[]

  @@index([email])
  @@index([phone])
  @@index([role])
  @@index([kyc_status])
  @@map("users")
}

// ============================================================================
// TABLE 2: user_sessions
// ============================================================================

model UserSession {
  id           String   @id @default(uuid())
  user_id      String
  token        String   @unique
  device_info  Json?    // Device fingerprint, OS, browser
  ip_address   String?
  expires_at   DateTime
  created_at   DateTime @default(now())
  revoked_at   DateTime?
  
  user         User     @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([token])
  @@index([expires_at])
  @@map("user_sessions")
}

// ============================================================================
// TABLE 3: refresh_tokens
// ============================================================================

model RefreshToken {
  id         String   @id @default(uuid())
  user_id    String
  token      String   @unique
  expires_at DateTime
  revoked    Boolean  @default(false)
  created_at DateTime @default(now())
  
  user       User     @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([token])
  @@map("refresh_tokens")
}

// ============================================================================
// TABLE 4: password_resets
// ============================================================================

model PasswordReset {
  id         String   @id @default(uuid())
  user_id    String
  token      String   @unique
  expires_at DateTime
  used_at    DateTime?
  created_at DateTime @default(now())
  
  user       User     @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([token])
  @@map("password_resets")
}

// ============================================================================
// TABLE 5: kyc_documents
// ============================================================================

model KYCDocument {
  id             String     @id @default(uuid())
  user_id        String
  document_type  String     // national_id, ier_license, insurance, business_reg, tax_cert
  cloudinary_url  String
  cloudinary_public_id String
  status         String     @default("pending") // pending, approved, rejected
  reviewed_by    String?
  reviewed_at    DateTime?
  rejection_reason String?
  created_at     DateTime   @default(now())
  
  user           User       @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([status])
  @@map("kyc_documents")
}

// ============================================================================
// TABLE 6: projects
// ============================================================================

model Project {
  id               String         @id @default(uuid())
  client_id        String
  engineer_id      String?
  name             String
  description      String?
  budget           Int            // In RWF (smallest unit)
  address          String
  gps_boundary     Json?          // GeoJSON polygon
  status           ProjectStatus  @default(draft)
  start_date       DateTime?
  end_date         DateTime?
  site_photos      Json?          // Array of Cloudinary URLs
  plans_urls       Json?          // Array of Cloudinary URLs (PDFs)
  created_at       DateTime       @default(now())
  updated_at       DateTime       @updatedAt
  
  // Relations
  client           User           @relation("ClientProjects", fields: [client_id], references: [id])
  engineer         User?          @relation("EngineerProjects", fields: [engineer_id], references: [id])
  project_members  ProjectMember[]
  milestones       Milestone[]
  escrow_account   EscrowAccount?
  transactions     Transaction[]
  rfqs             RFQ[]
  progress_photos  ProgressPhoto[]
  disputes         Dispute[]
  audit_logs       AuditLog[]
  
  @@index([client_id])
  @@index([engineer_id])
  @@index([status])
  @@map("projects")
}

// ============================================================================
// TABLE 7: project_members (Many-to-many for multiple assignments)
// ============================================================================

model ProjectMember {
  id           String   @id @default(uuid())
  project_id   String
  user_id      String
  role         String   // supervisor, supplier, additional_engineer
  status       String   @default("pending") // pending, active, declined, removed
  invited_at   DateTime @default(now())
  accepted_at  DateTime?
  removed_at   DateTime?
  
  project      Project  @relation(fields: [project_id], references: [id], onDelete: CASCADE)
  user         User     @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@unique([project_id, user_id])
  @@index([project_id])
  @@index([user_id])
  @@index([status])
  @@map("project_members")
}

// ============================================================================
// TABLE 8: milestones
// ============================================================================

model Milestone {
  id                 String          @id @default(uuid())
  project_id         String
  title              String
  description        String?
  budget_percentage  Int             // 1-100
  duration_days      Int?
  acceptance_criteria String?
  depends_on         String?         // FK to another milestone
  status             MilestoneStatus @default(pending)
  order_index        Int             // Display order
  created_by         String
  created_at         DateTime        @default(now())
  updated_at         DateTime        @updatedAt
  completed_at       DateTime?
  paid_at            DateTime?
  
  // Relations
  project            Project         @relation(fields: [project_id], references: [id], onDelete: CASCADE)
  creator            User            @relation("CreatedBy", fields: [created_by], references: [id])
  dependent_milestone Milestone?      @relation("MilestoneDependency", fields: [depends_on], references: [id])
  boq_items          BoQItem[]
  transactions       Transaction[]
  inspections        Inspection[]
  disputes           Dispute[]
  progress_photos    ProgressPhoto[]
  
  @@index([project_id])
  @@index([status])
  @@index([depends_on])
  @@map("milestones")
}

// ============================================================================
// TABLE 9: boq_items
// ============================================================================

model BoQItem {
  id           String   @id @default(uuid())
  milestone_id String
  category     String   // concrete, steel, timber, finishes, labor, equipment
  material     String
  quantity     Float
  unit         String   // bags, cubic_meters, pieces, lumpsum
  unit_price   Int      // RWF
  total_price  Int      // calculated: quantity * unit_price
  actual_cost  Int?     // Actual cost after purchase
  notes        String?
  created_at   DateTime @default(now())
  updated_at   DateTime @updatedAt
  
  milestone    Milestone @relation(fields: [milestone_id], references: [id], onDelete: CASCADE)
  
  @@index([milestone_id])
  @@map("boq_items")
}

// ============================================================================
// TABLE 10: escrow_accounts
// ============================================================================

model EscrowAccount {
  id             String   @id @default(uuid())
  project_id     String   @unique
  balance        Int      @default(0)      // Available balance in RWF
  locked_balance Int      @default(0)      // Balance in dispute
  currency       String   @default("RWF")
  created_at     DateTime @default(now())
  updated_at     DateTime @updatedAt
  
  project        Project  @relation(fields: [project_id], references: [id], onDelete: CASCADE)
  transactions   Transaction[]
  
  @@map("escrow_accounts")
}

// ============================================================================
// TABLE 11: transactions
// ============================================================================

model Transaction {
  id                 String          @id @default(uuid())
  project_id         String
  escrow_account_id  String
  milestone_id       String?
  amount             Int
  type               TransactionType
  status             TransactionStatus @default(pending)
  reference_id       String?         // MTN transaction ID, bank reference
  description        String?
  actor_id           String          // User who initiated
  metadata           Json?           // Additional data (webhook response, etc.)
  created_at         DateTime        @default(now())
  completed_at       DateTime?
  
  project            Project         @relation(fields: [project_id], references: [id])
  escrow_account     EscrowAccount   @relation(fields: [escrow_account_id], references: [id])
  milestone          Milestone?      @relation(fields: [milestone_id], references: [id])
  actor              User            @relation(fields: [actor_id], references: [id])
  
  @@index([project_id])
  @@index([escrow_account_id])
  @@index([milestone_id])
  @@index([actor_id])
  @@index([status])
  @@index([created_at])
  @@map("transactions")
}

// ============================================================================
// TABLE 12: rfqs (Request for Quotes)
// ============================================================================

model RFQ {
  id               String   @id @default(uuid())
  project_id       String
  milestone_id     String
  engineer_id      String
  title            String
  material_spec    Json     // JSONB with material requirements
  quantity         Float
  unit             String
  delivery_deadline DateTime
  status           String   @default("open") // open, closed, awarded, expired
  created_at       DateTime @default(now())
  expires_at       DateTime
  
  project          Project  @relation(fields: [project_id], references: [id])
  milestone        Milestone @relation(fields: [milestone_id], references: [id])
  engineer         User     @relation("RFQCreatedBy", fields: [engineer_id], references: [id])
  quotes           Quote[]
  purchase_order   PurchaseOrder?
  
  @@index([project_id])
  @@index([milestone_id])
  @@index([status])
  @@map("rfqs")
}

// ============================================================================
// TABLE 13: quotes
// ============================================================================

model Quote {
  id             String   @id @default(uuid())
  rfq_id         String
  supplier_id    String
  unit_price     Int
  total_price    Int
  delivery_days  Int
  terms          String?
  status         String   @default("pending") // pending, selected, rejected, expired
  certificate_urls Json?   // Array of Cloudinary URLs
  submitted_at   DateTime @default(now())
  selected_at    DateTime?
  
  rfq            RFQ      @relation(fields: [rfq_id], references: [id])
  supplier       User     @relation("QuoteSubmittedBy", fields: [supplier_id], references: [id])
  purchase_order PurchaseOrder?
  
  @@index([rfq_id])
  @@index([supplier_id])
  @@index([status])
  @@map("quotes")
}

// ============================================================================
// TABLE 14: purchase_orders
// ============================================================================

model PurchaseOrder {
  id              String   @id @default(uuid())
  quote_id        String   @unique
  rfq_id          String
  po_number       String   @unique
  cloudinary_url  String   // PDF stored on Cloudinary
  status          String   @default("issued") // issued, accepted, shipped, completed
  issued_at       DateTime @default(now())
  accepted_at     DateTime?
  completed_at    DateTime?
  
  quote           Quote    @relation(fields: [quote_id], references: [id])
  rfq             RFQ      @relation(fields: [rfq_id], references: [id])
  deliveries      Delivery[]
  
  @@index([po_number])
  @@index([status])
  @@map("purchase_orders")
}

// ============================================================================
// TABLE 15: deliveries
// ============================================================================

model Delivery {
  id                 String   @id @default(uuid())
  purchase_order_id  String
  supplier_id        String
  engineer_id        String?
  supervisor_id      String?
  start_gps          Json     // { lat, lng, timestamp }
  end_gps            Json     // { lat, lng, timestamp }
  photos             Json     // Array of Cloudinary URLs
  status             String   @default("in_transit") // in_transit, delivered, confirmed, rejected
  started_at         DateTime @default(now())
  delivered_at       DateTime?
  confirmed_at       DateTime?
  rejection_reason   String?
  created_at         DateTime @default(now())
  
  purchase_order     PurchaseOrder @relation(fields: [purchase_order_id], references: [id])
  supplier           User          @relation(fields: [supplier_id], references: [id])
  engineer           User?         @relation(fields: [engineer_id], references: [id])
  supervisor         User?         @relation(fields: [supervisor_id], references: [id])
  
  @@index([purchase_order_id])
  @@index([supplier_id])
  @@index([status])
  @@map("deliveries")
}

// ============================================================================
// TABLE 16: progress_photos
// ============================================================================

model ProgressPhoto {
  id            String   @id @default(uuid())
  project_id    String
  milestone_id  String?
  uploaded_by   String
  cloudinary_url String
  cloudinary_public_id String
  gps_location  Json?    // { lat, lng } from device
  caption       String?
  is_video      Boolean  @default(false)
  video_duration Int?    // seconds, max 120
  created_at    DateTime @default(now())
  
  project       Project  @relation(fields: [project_id], references: [id])
  milestone     Milestone? @relation(fields: [milestone_id], references: [id])
  uploader      User     @relation(fields: [uploaded_by], references: [id])
  
  @@index([project_id])
  @@index([milestone_id])
  @@index([uploaded_by])
  @@index([created_at])
  @@map("progress_photos")
}

// ============================================================================
// TABLE 17: inspections
// ============================================================================

model Inspection {
  id             String   @id @default(uuid())
  milestone_id   String   @unique
  supervisor_id  String
  checklist      Json     // JSONB of Q&A pairs
  rating         Int      // 1-5 overall rating
  photos         Json     // Array of Cloudinary URLs
  signature_url  String   // Cloudinary URL of signature image
  notes          String?
  status         String   @default("pending") // pending, approved, rejected
  submitted_at   DateTime @default(now())
  
  milestone      Milestone @relation(fields: [milestone_id], references: [id])
  supervisor     User      @relation(fields: [supervisor_id], references: [id])
  
  @@index([milestone_id])
  @@index([supervisor_id])
  @@index([status])
  @@map("inspections")
}

// ============================================================================
// TABLE 18: disputes
// ============================================================================

model Dispute {
  id               String        @id @default(uuid())
  project_id       String
  milestone_id     String?
  raised_by        String
  against_user_id  String?
  category         String        // quality, timeline, cost, other
  description      String
  status           DisputeStatus @default(open)
  amount_in_dispute Int          // RWF amount locked
  resolved_at      DateTime?
  resolved_by      String?
  decision         Json?         // JSONB with resolution details
  created_at       DateTime      @default(now())
  
  project          Project       @relation(fields: [project_id], references: [id])
  milestone        Milestone?    @relation(fields: [milestone_id], references: [id])
  raised_by_user   User          @relation("DisputeRaisedBy", fields: [raised_by], references: [id])
  evidence         DisputeEvidence[]
  
  @@index([project_id])
  @@index([milestone_id])
  @@index([status])
  @@map("disputes")
}

// ============================================================================
// TABLE 19: dispute_evidence
// ============================================================================

model DisputeEvidence {
  id             String   @id @default(uuid())
  dispute_id     String
  uploaded_by    String
  cloudinary_url String
  description    String?
  uploaded_at    DateTime @default(now())
  
  dispute        Dispute  @relation(fields: [dispute_id], references: [id], onDelete: CASCADE)
  uploader       User     @relation(fields: [uploaded_by], references: [id])
  
  @@index([dispute_id])
  @@map("dispute_evidence")
}

// ============================================================================
// TABLE 20: audit_logs
// ============================================================================

model AuditLog {
  id                String   @id @default(uuid())
  actor_id          String?
  action            String   // login, logout, kyc_approve, payment_release, dispute_open
  entity_type       String   // user, project, milestone, escrow, dispute
  entity_id         String?
  old_values        Json?
  new_values        Json?
  ip_address        String?
  user_agent        String?
  device_fingerprint String?
  result            String   // success, failure, partial
  created_at        DateTime @default(now())
  
  actor             User?    @relation(fields: [actor_id], references: [id])
  
  @@index([actor_id])
  @@index([entity_type, entity_id])
  @@index([created_at])
  @@map("audit_logs")
}

// ============================================================================
// TABLE 21: notifications
// ============================================================================

model Notification {
  id             String             @id @default(uuid())
  user_id        String
  channel        NotificationChannel
  title          String
  body           String
  data           Json?              // Additional payload
  status         NotificationStatus @default(pending)
  sent_at        DateTime?
  delivered_at   DateTime?
  read_at        DateTime?
  failure_reason String?
  created_at     DateTime           @default(now())
  
  user           User               @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([status])
  @@index([created_at])
  @@map("notifications")
}

// ============================================================================
// TABLE 22: api_keys (for supplier/external integration)
// ============================================================================

model ApiKey {
  id             String   @id @default(uuid())
  user_id        String
  name           String
  key_hash       String   @unique
  prefix         String   // First 8 chars for identification
  permissions    Json     // Array of allowed actions
  expires_at     DateTime?
  last_used_at   DateTime?
  created_at     DateTime @default(now())
  revoked_at     DateTime?
  
  user           User     @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([key_hash])
  @@map("api_keys")
}

// ============================================================================
// TABLE 23: activity_logs (lighter than audit_logs for user activity)
// ============================================================================

model ActivityLog {
  id         String   @id @default(uuid())
  user_id    String
  action     String   // viewed_project, uploaded_photo, etc.
  metadata   Json?
  ip_address String?
  created_at DateTime @default(now())
  
  user       User     @relation(fields: [user_id], references: [id], onDelete: CASCADE)
  
  @@index([user_id])
  @@index([created_at])
  @@map("activity_logs")
}

// ============================================================================
// TABLE 24: system_settings
// ============================================================================

model SystemSetting {
  id           String   @id @default(uuid())
  key          String   @unique
  value        Json
  description  String?
  updated_by   String?
  updated_at   DateTime @updatedAt
  created_at   DateTime @default(now())
  
  @@map("system_settings")
}

// ============================================================================
// TABLE 25: email_templates
// ============================================================================

model EmailTemplate {
  id           String   @id @default(uuid())
  name         String   @unique  // kyc_approval, payment_receipt, etc.
  subject      String
  html_content String
  plain_text   String?
  updated_at   DateTime @updatedAt
  created_at   DateTime @default(now())
  
  @@map("email_templates")
}
```

---

## Part 3: API Structure (REST Endpoints)

### 3.1 API Versioning Strategy

```
Base URL: https://api.inkingipro.com/api/v1/

All endpoints require JWT Bearer token except:
- /auth/*
- /webhooks/*
- /health
```

### 3.2 Complete API Routes Table

| Method | Endpoint | Description | Request Body | Response | Auth Role |
|--------|----------|-------------|--------------|----------|-----------|
| **AUTHENTICATION** |
| POST | `/auth/register` | Register new user | `{email, phone, password, full_name, role}` | `{user_id, message}` | None |
| POST | `/auth/verify-email` | Verify email OTP | `{email, otp}` | `{verified: true}` | None |
| POST | `/auth/verify-phone` | Verify phone OTP | `{phone, otp}` | `{verified: true}` | None |
| POST | `/auth/login` | Login with email/password | `{email, password}` | `{access_token, refresh_token, user}` | None |
| POST | `/auth/login/otp` | Login with phone OTP | `{phone}` | `{message: "OTP sent"}` | None |
| POST | `/auth/verify-otp-login` | Verify OTP and login | `{phone, otp}` | `{access_token, refresh_token, user}` | None |
| POST | `/auth/refresh` | Refresh access token | `{refresh_token}` | `{access_token}` | None |
| POST | `/auth/logout` | Logout user | None | `{message: "Logged out"}` | All |
| POST | `/auth/forgot-password` | Request password reset | `{email}` | `{message: "OTP sent"}` | None |
| POST | `/auth/reset-password` | Reset password with OTP | `{email, otp, new_password}` | `{message: "Password updated"}` | None |
| GET | `/auth/me` | Get current user profile | None | `{user}` | All |
| PUT | `/auth/me` | Update profile | `{full_name, phone, locale}` | `{user}` | All |
| POST | `/auth/change-password` | Change password | `{current_password, new_password}` | `{message: "Password updated"}` | All |
| **KYC MANAGEMENT** |
| POST | `/kyc/upload-url` | Get signed Cloudinary URL | `{document_type}` | `{url, public_id, folder}` | All |
| POST | `/kyc/documents` | Save KYC document | `{document_type, cloudinary_url, public_id}` | `{document_id}` | All |
| GET | `/kyc/status` | Get user KYC status | None | `{status, documents[]}` | All |
| GET | `/admin/kyc/pending` | List pending KYC submissions | Query: `?page=1&limit=20` | `{users[], total}` | Admin |
| GET | `/admin/kyc/:userId` | Get KYC details for user | None | `{user, documents[]}` | Admin |
| POST | `/admin/kyc/:userId/approve` | Approve KYC | `{notes?}` | `{message: "Approved"}` | Admin |
| POST | `/admin/kyc/:userId/reject` | Reject KYC | `{reason}` | `{message: "Rejected"}` | Admin |
| POST | `/admin/kyc/:userId/request-info` | Request additional documents | `{request_message}` | `{message: "Request sent"}` | Admin |
| **PROJECT MANAGEMENT** |
| GET | `/projects` | List user's projects | Query: `?status=active&page=1&limit=20` | `{projects[], total}` | All |
| POST | `/projects` | Create new project | `{name, budget, address, description, start_date?, end_date?}` | `{project}` | Client |
| GET | `/projects/:id` | Get project details | None | `{project, milestones[], escrow_balance}` | Member |
| PUT | `/projects/:id` | Update project | `{name, description, address}` | `{project}` | Client |
| DELETE | `/projects/:id` | Soft delete project | None | `{message: "Deleted"}` | Client |
| POST | `/projects/:id/invite` | Invite user to project | `{email, role}` (role: engineer/supervisor/supplier) | `{invitation_id}` | Client |
| GET | `/projects/:id/members` | List project members | None | `{members[]}` | Member |
| DELETE | `/projects/:id/members/:userId` | Remove member | None | `{message: "Removed"}` | Client |
| GET | `/projects/:id/timeline` | Get progress timeline with photos | Query: `?limit=50` | `{photos[], inspections[]}` | Member |
| **MILESTONES** |
| GET | `/projects/:projectId/milestones` | List milestones | None | `{milestones[]}` | Member |
| POST | `/projects/:projectId/milestones` | Create milestone | `{title, budget_percentage, duration_days?, acceptance_criteria?, depends_on?}` | `{milestone}` | Engineer |
| PUT | `/milestones/:id` | Update milestone | `{title, budget_percentage, acceptance_criteria}` | `{milestone}` | Engineer |
| DELETE | `/milestones/:id` | Delete milestone | None | `{message: "Deleted"}` | Engineer |
| POST | `/milestones/:id/activate` | Client activates milestone | None | `{milestone}` | Client |
| POST | `/milestones/:id/request-payment` | Engineer requests payment | `{completion_notes, photo_ids[]}` | `{payment_request}` | Engineer |
| POST | `/milestones/:id/payment-decision` | Client decision on payment | `{decision: approve/reject/dispute, notes?}` | `{milestone, transaction?}` | Client |
| GET | `/milestones/:id/inspection` | Get inspection for milestone | None | `{inspection}` | Member |
| **BOQ (Bill of Quantities)** |
| GET | `/milestones/:milestoneId/boq` | List BOQ items | None | `{items[]}` | Engineer, Client |
| POST | `/milestones/:milestoneId/boq` | Create BOQ item | `{category, material, quantity, unit, unit_price}` | `{item}` | Engineer |
| PUT | `/boq/:id` | Update BOQ item | `{quantity, unit_price}` | `{item}` | Engineer |
| DELETE | `/boq/:id` | Delete BOQ item | None | `{message: "Deleted"}` | Engineer |
| GET | `/boq/:milestoneId/export/pdf` | Export BOQ to PDF | None | PDF file | Engineer, Client |
| GET | `/boq/:milestoneId/export/excel` | Export BOQ to Excel | None | XLSX file | Engineer, Client |
| **ESCROW & PAYMENTS** |
| GET | `/escrow/projects/:projectId/balance` | Get escrow balance | None | `{balance, locked_balance, total_deposited, total_released}` | Member |
| POST | `/escrow/deposit/mtn` | Initiate MTN deposit | `{amount, phone_number}` | `{transaction_id, payment_reference}` | Client |
| POST | `/escrow/deposit/airtel` | Initiate Airtel deposit | `{amount, phone_number}` | `{transaction_id, payment_reference}` | Client |
| POST | `/escrow/deposit/bank` | Initiate bank transfer | `{amount, reference?}` | `{bank_details, transaction_id}` | Client |
| GET | `/escrow/transactions` | List user transactions | Query: `?project_id=&page=1&limit=20` | `{transactions[], total}` | Member |
| GET | `/escrow/transactions/:id` | Get transaction details | None | `{transaction}` | Member |
| POST | `/webhooks/mtn` | MTN MoMo webhook | MTN payload | `{status: "received"}` | Public |
| POST | `/webhooks/airtel` | Airtel webhook | Airtel payload | `{status: "received"}` | Public |
| **SUPPLY CHAIN (RFQ/Quotes)** |
| GET | `/projects/:projectId/rfqs` | List RFQs | Query: `?status=open` | `{rfqs[]}` | Engineer, Admin |
| POST | `/projects/:projectId/rfqs` | Create RFQ | `{milestone_id, title, material_spec, quantity, unit, delivery_deadline}` | `{rfq}` | Engineer |
| GET | `/rfqs/:id` | Get RFQ details | None | `{rfq, quotes[]}` | Member |
| PUT | `/rfqs/:id` | Update RFQ | `{material_spec, quantity, delivery_deadline}` | `{rfq}` | Engineer |
| POST | `/rfqs/:id/close` | Close RFQ (stop accepting quotes) | None | `{message: "Closed"}` | Engineer |
| GET | `/rfqs/:id/quotes` | List quotes for RFQ | None | `{quotes[]}` | Engineer, Admin |
| POST | `/rfqs/:id/quotes` | Supplier submit quote | `{unit_price, total_price, delivery_days, terms}` | `{quote}` | Supplier |
| PUT | `/quotes/:id` | Update quote (before selection) | `{unit_price, delivery_days}` | `{quote}` | Supplier |
| POST | `/quotes/:id/select` | Engineer selects quote | None | `{purchase_order}` | Engineer |
| **PURCHASE ORDERS & DELIVERIES** |
| GET | `/purchase-orders` | List purchase orders | Query: `?status=issued` | `{orders[]}` | Supplier, Engineer |
| GET | `/purchase-orders/:id` | Get PO details | None | `{po, deliveries[]}` | Member |
| GET | `/purchase-orders/:id/download` | Download PO PDF | None | PDF file | Member |
| POST | `/deliveries` | Start delivery | `{purchase_order_id, start_gps}` | `{delivery}` | Supplier |
| PUT | `/deliveries/:id/gps` | Update delivery GPS | `{current_gps}` | `{delivery}` | Supplier |
| POST | `/deliveries/:id/complete` | Complete delivery | `{end_gps, photos[], notes?}` | `{delivery}` | Supplier |
| POST | `/deliveries/:id/confirm` | Confirm delivery receipt | `{confirmed: true/false, rejection_reason?}` | `{delivery}` | Engineer, Supervisor |
| **PROGRESS PHOTOS** |
| GET | `/projects/:projectId/photos` | List progress photos | Query: `?milestone_id=&limit=50` | `{photos[]}` | Member |
| POST | `/projects/:projectId/photos/upload-url` | Get signed URL for photo | `{milestone_id?}` | `{url, public_id, folder}` | Engineer |
| POST | `/projects/:projectId/photos` | Save photo record | `{milestone_id?, cloudinary_url, public_id, gps_location?, caption?}` | `{photo}` | Engineer |
| DELETE | `/photos/:id` | Delete photo | None | `{message: "Deleted"}` | Engineer, Admin |
| **INSPECTIONS** |
| GET | `/inspections/pending` | List pending inspections | None | `{inspections[]}` | Supervisor |
| GET | `/inspections/:id` | Get inspection details | None | `{inspection}` | Member |
| POST | `/inspections` | Submit inspection | `{milestone_id, checklist, rating, photos, signature, notes}` | `{inspection}` | Supervisor |
| PUT | `/inspections/:id` | Update inspection (before submit) | `{checklist, rating, notes}` | `{inspection}` | Supervisor |
| GET | `/inspections/:id/download` | Download inspection report PDF | None | PDF file | Member |
| **DISPUTES** |
| GET | `/projects/:projectId/disputes` | List project disputes | None | `{disputes[]}` | Member |
| POST | `/disputes` | Initiate dispute | `{project_id, milestone_id?, category, description, amount_in_dispute}` | `{dispute}` | All |
| GET | `/disputes/:id` | Get dispute details | None | `{dispute, evidence[]}` | Involved parties |
| POST | `/disputes/:id/evidence` | Upload evidence | `{cloudinary_url, description}` | `{evidence}` | Involved parties |
| PUT | `/disputes/:id/close` | Close dispute (withdraw) | None | `{message: "Closed"}` | Initiator |
| POST | `/admin/disputes/:id/resolve` | Admin resolution | `{decision_type, amount_to_release?, penalty_amount?, notes}` | `{dispute, transaction?}` | Admin |
| POST | `/disputes/:id/appeal` | Appeal decision | `{reason, new_evidence_urls?}` | `{appeal}` | Any party |
| **NOTIFICATIONS** |
| GET | `/notifications` | List user notifications | Query: `?status=unread&page=1&limit=20` | `{notifications[], total}` | All |
| PUT | `/notifications/:id/read` | Mark as read | None | `{notification}` | All |
| PUT | `/notifications/read-all` | Mark all as read | None | `{message: "All marked read"}` | All |
| PUT | `/notifications/preferences` | Update preferences | `{email_enabled, push_enabled, sms_enabled}` | `{preferences}` | All |
| **ADMIN ONLY** |
| GET | `/admin/users` | List all users | Query: `?role=&kyc_status=&page=1&limit=20` | `{users[], total}` | Admin |
| GET | `/admin/users/:id` | Get user details | None | `{user, kyc_documents[], projects[]}` | Admin |
| POST | `/admin/users/:id/suspend` | Suspend user | `{reason, days?}` | `{message: "Suspended"}` | Admin |
| POST | `/admin/users/:id/activate` | Activate suspended user | None | `{message: "Activated"}` | Admin |
| GET | `/admin/audit-logs` | View audit logs | Query: `?action=&entity_type=&page=1&limit=50` | `{logs[], total}` | Admin |
| GET | `/admin/reports/compliance` | Generate compliance report | Query: `?start_date=&end_date=&format=pdf` | PDF or JSON | Admin |
| GET | `/admin/reports/financial` | Generate financial report | Query: `?period=month` | `{report}` | Admin |
| GET | `/admin/reports/supplier-performance` | Supplier performance report | Query: `?supplier_id=&period=90d` | `{report}` | Admin |
| GET | `/admin/stats/dashboard` | Dashboard statistics | None | `{kyc_pending, active_projects, disputes_open, total_escrow}` | Admin |
| PUT | `/admin/settings` | Update system settings | `{key, value}` | `{setting}` | Admin |
| GET | `/admin/settings` | Get all system settings | None | `{settings[]}` | Admin |
| **SYSTEM** |
| GET | `/health` | Health check | None | `{status: "ok", timestamp, services}` | Public |
| GET | `/api/docs` | Swagger documentation | None | HTML | Public |

---

## Part 4: Prisma Setup Commands

```bash
# 1. Install Prisma
npm install @prisma/client
npm install -D prisma

# 2. Initialize Prisma
npx prisma init

# 3. Copy the schema above to prisma/schema.prisma

# 4. Generate Prisma Client
npx prisma generate

# 5. Create migration
npx prisma migrate dev --name init

# 6. Push to production database
npx prisma db push

# 7. Open Prisma Studio to view data
npx prisma studio
```

---

## Part 5: Environment Variables (.env)

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/inkingipro"

# JWT
JWT_SECRET="your-secret-key-min-32-chars"
JWT_REFRESH_SECRET="your-refresh-secret-min-32-chars"

# Cloudinary
CLOUDINARY_CLOUD_NAME="your-cloud-name"
CLOUDINARY_API_KEY="your-api-key"
CLOUDINARY_API_SECRET="your-api-secret"

# Email (Nodemailer)
SMTP_HOST="smtp.gmail.com"
SMTP_PORT="587"
SMTP_USER="noreply@inkingipro.com"
SMTP_PASS="your-app-password"
EMAIL_FROM="noreply@inkingipro.com"

# Africa's Talking
AFRICA_TALKING_API_KEY="your-api-key"
AFRICA_TALKING_USERNAME="sandbox"
AFRICA_TALKING_SENDER_ID="INKINGI"

# MTN MoMo
MTN_API_URL="https://sandbox.mtn.co.rw"
MTN_CLIENT_ID="your-client-id"
MTN_CLIENT_SECRET="your-client-secret"
MTN_SUBSCRIPTION_KEY="your-subscription-key"

# Airtel
AIRTEL_API_URL="https://openapi.airtel.africa"
AIRTEL_CLIENT_ID="your-client-id"
AIRTEL_CLIENT_SECRET="your-client-secret"

# Redis (optional for production)
REDIS_URL="redis://localhost:6379"

# Environment
NODE_ENV="development"
PORT="3000"
API_VERSION="v1"
```

---

## Part 6: Quick Reference Card

### Database Table Count: 25 Tables

| Category | Tables |
|----------|--------|
| **User Management** | users, user_sessions, refresh_tokens, password_resets, api_keys |
| **KYC** | kyc_documents |
| **Projects** | projects, project_members, milestones, boq_items |
| **Financial** | escrow_accounts, transactions |
| **Supply Chain** | rfqs, quotes, purchase_orders, deliveries |
| **Media** | progress_photos |
| **Quality** | inspections |
| **Disputes** | disputes, dispute_evidence |
| **Logging** | audit_logs, activity_logs, notifications |
| **System** | system_settings, email_templates |

### API Endpoints Count: 75+ Endpoints

| Category | Count |
|----------|-------|
| Authentication | 12 |
| KYC | 8 |
| Projects | 9 |
| Milestones | 8 |
| BOQ | 6 |
| Escrow/Payments | 8 |
| Supply Chain | 9 |
| Purchase Orders/Deliveries | 6 |
| Progress Photos | 4 |
| Inspections | 5 |
| Disputes | 7 |
| Notifications | 4 |
| Admin | 10 |
| System | 2 |

---

*This document replaces all previous database designs and provides the complete, production-ready schema for InkingiPro.*