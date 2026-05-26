# InkingiPro Database Architecture

## Unified Database Architecture

```prisma
// ============================================================================
// FILE: schema.prisma
// DATABASE: PostgreSQL 15+
// ORM: Prisma 5+
// TABLES: 25 Tables
// ============================================================================

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================================
// ENUMS
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
  pending_supervisor
  revision_required
  awaiting_client_payment
  paid
}

enum TransactionType {
  deposit
  release
  refund
  freeze
  unfreeze
  auto_payment
  penalty
}

enum TransactionStatus {
  pending
  completed
  failed
  reversed
}

enum PaymentMethod {
  mtn_momo
  airtel_money
  bank_transfer
}

enum RfqStatus {
  open
  closed
  cancelled
}

enum QuoteStatus {
  pending_selection
  selected
  rejected
}

enum PurchaseOrderStatus {
  issued
  accepted
  shipped
  completed
}

enum DeliveryStatus {
  preparing
  in_transit
  delivered
  pending_confirmation
  confirmed
  rejected
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

enum DisputeCategory {
  quality
  timeline
  cost
  other
}

enum NotificationChannel {
  push
  email
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

enum AssignmentStatus {
  pending
  accepted
  declined
  removed
}

enum InspectionDecision {
  approved
  revision_required
}

enum KycDocumentType {
  national_id
  passport
  ier_license
  indemnity_insurance
  business_registration
  tax_compliance
  certification
}

enum KycDocumentStatus {
  pending
  approved
  rejected
}

// ============================================================================
// TABLE 1: User
// ============================================================================

model User {
  id                       String                 @id @default(uuid())
  name                     String
  email                    String                 @unique
  phoneNumber              String                 @unique
  emailVerified            Boolean                @default(false)
  phoneNumberVerified      Boolean                @default(false)
  passwordHash             String
  image                    String?
  role                     UserRole               @default(client)
  
  // KYC Fields
  kycStatus                KYCStatus              @default(pending)
  kycSubmittedAt           DateTime?
  kycReviewedAt            DateTime?
  kycReviewedBy            String?
  kycRejectionReason       String?
  
  // Engineer-specific
  ierLicenseNumber         String?                @unique
  ierLicenseExpiry         DateTime?
  ierLicenseVerified       Boolean                @default(false)
  professionalIndemnityUrl String?
  
  // Supplier-specific
  businessRegistrationNumber String?               @unique
  taxComplianceNumber      String?
  businessRegistrationUrl  String?
  taxCertificateUrl        String?
  
  // Account status
  isActive                 Boolean                @default(true)
  isSuspended              Boolean                @default(false)
  suspensionReason         String?
  suspendedUntil           DateTime?
  banned                   Boolean?               @default(false)
  banReason                String?
  banExpires               DateTime?
  
  // Preferences
  notificationPrefs        Json                   @default("{}")
  locale                   String                 @default("en")
  currencyPreference       String                 @default("RWF")
  fcmToken                 String?
  
  // Timestamps
  createdAt                DateTime               @default(now())
  updatedAt                DateTime               @updatedAt
  lastLoginAt              DateTime?
  
  // Relations
  sessions                 Session[]
  accounts                 Account[]
  kycDocuments             KycDocument[]
  
  clientProjects           Project[]              @relation("ClientProjects")
  engineerProjects         Project[]              @relation("EngineerProjects")
  projectMembers           ProjectMember[]
  
  milestonesCreated        Milestone[]            @relation("CreatedBy")
  transactions             Transaction[]
  
  rfqsCreated              Rfq[]                  @relation("RFQCreatedBy")
  quotesSubmitted          Quote[]                @relation("QuoteSubmittedBy")
  purchaseOrders           PurchaseOrder[]
  deliveries               Delivery[]
  progressPhotos           ProgressPhoto[]
  inspections              Inspection[]
  
  disputesRaised           Dispute[]              @relation("DisputeRaisedBy")
  disputeEvidence          DisputeEvidence[]
  
  auditLogs                AuditLog[]
  activityLogs             ActivityLog[]
  notifications            Notification[]
  apiKeys                  ApiKey[]
  
  refreshTokens            RefreshToken[]
  passwordResets           PasswordReset[]

  @@index([email])
  @@index([phoneNumber])
  @@index([role])
  @@index([kycStatus])
  @@map("users")
}

// ============================================================================
// TABLE 2: Session (Better Auth)
// ============================================================================

model Session {
  id             String   @id
  expiresAt      DateTime
  token          String   @unique
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  ipAddress      String?
  userAgent      String?
  userId         String
  user           User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  impersonatedBy String?

  @@index([userId])
  @@map("sessions")
}

// ============================================================================
// TABLE 3: Account (Better Auth)
// ============================================================================

model Account {
  id                    String    @id
  accountId             String
  providerId            String
  userId                String
  user                  User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  accessToken           String?
  refreshToken          String?
  idToken               String?
  accessTokenExpiresAt  DateTime?
  refreshTokenExpiresAt DateTime?
  scope                 String?
  password              String?
  createdAt             DateTime  @default(now())
  updatedAt             DateTime  @updatedAt

  @@index([userId])
  @@map("accounts")
}

// ============================================================================
// TABLE 4: Verification (Better Auth)
// ============================================================================

model Verification {
  id         String   @id
  identifier String
  value      String
  expiresAt  DateTime
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@index([identifier])
  @@map("verifications")
}

// ============================================================================
// TABLE 5: RefreshToken
// ============================================================================

model RefreshToken {
  id         String   @id @default(uuid())
  userId     String
  token      String   @unique
  expiresAt  DateTime
  revoked    Boolean  @default(false)
  createdAt  DateTime @default(now())
  
  user       User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
  @@index([token])
  @@map("refresh_tokens")
}

// ============================================================================
// TABLE 6: PasswordReset
// ============================================================================

model PasswordReset {
  id         String   @id @default(uuid())
  userId     String
  token      String   @unique
  expiresAt  DateTime
  usedAt     DateTime?
  createdAt  DateTime @default(now())
  
  user       User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
  @@index([token])
  @@map("password_resets")
}

// ============================================================================
// TABLE 7: KycDocument
// ============================================================================

model KycDocument {
  id                   String            @id @default(uuid())
  userId               String
  type                 KycDocumentType
  cloudinaryUrl        String
  cloudinaryPublicId   String
  status               KycDocumentStatus @default(pending)
  reviewedBy           String?
  reviewedAt           DateTime?
  rejectionReason      String?
  createdAt            DateTime          @default(now())
  
  user                 User              @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([userId, type])
  @@index([userId])
  @@index([status])
  @@map("kyc_documents")
}

// ============================================================================
// TABLE 8: Project
// ============================================================================

model Project {
  id                 String        @id @default(uuid())
  name               String
  description        String?
  status             ProjectStatus @default(draft)
  budget             Decimal       @db.Decimal(15, 2)
  currency           String        @default("RWF")
  address            String?
  gpsBoundary        Json?
  sitePhotos         Json          @default("[]")
  architecturalPlans Json          @default("[]")
  startDate          DateTime?
  endDate            DateTime?
  
  clientId           String
  client             User          @relation("ClientProjects", fields: [clientId], references: [id])
  engineerId         String?
  engineer           User?         @relation("EngineerProjects", fields: [engineerId], references: [id])
  
  createdAt          DateTime      @default(now())
  updatedAt          DateTime      @updatedAt

  // Relations
  milestones         Milestone[]
  escrowAccount      EscrowAccount?
  projectMembers     ProjectMember[]
  rfqs               Rfq[]
  progressPhotos     ProgressPhoto[]
  disputes           Dispute[]
  messages           Message[]
  auditLogs          AuditLog[]

  @@index([clientId])
  @@index([engineerId])
  @@index([status])
  @@map("projects")
}

// ============================================================================
// TABLE 9: ProjectMember
// ============================================================================

model ProjectMember {
  id          String           @id @default(uuid())
  projectId   String
  userId      String
  role        String
  status      AssignmentStatus @default(pending)
  invitedAt   DateTime         @default(now())
  acceptedAt  DateTime?
  removedAt   DateTime?
  
  project     Project          @relation(fields: [projectId], references: [id], onDelete: Cascade)
  user        User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([projectId, userId])
  @@index([projectId])
  @@index([userId])
  @@index([status])
  @@map("project_members")
}

// ============================================================================
// TABLE 10: Milestone
// ============================================================================

model Milestone {
  id                 String          @id @default(uuid())
  projectId          String
  engineerId         String
  name               String
  description        String?
  budgetPercentage   Decimal         @db.Decimal(5, 2)
  durationDays       Int?
  acceptanceCriteria String?
  dependsOn          String?
  order              Int
  status             MilestoneStatus @default(pending)
  completedAt        DateTime?
  paidAt             DateTime?
  createdAt          DateTime        @default(now())
  updatedAt          DateTime        @updatedAt

  // Relations
  project            Project         @relation(fields: [projectId], references: [id], onDelete: Cascade)
  engineer           User            @relation("CreatedBy", fields: [engineerId], references: [id])
  dependentMilestone Milestone?      @relation("MilestoneDependency", fields: [dependsOn], references: [id])
  
  boqItems           BoqItem[]
  inspections        Inspection[]
  rfqs               Rfq[]
  transactions       Transaction[]
  progressPhotos     ProgressPhoto[]
  disputes           Dispute[]

  @@index([projectId])
  @@index([engineerId])
  @@index([status])
  @@index([dependsOn])
  @@map("milestones")
}

// ============================================================================
// TABLE 11: BoqItem
// ============================================================================

model BoqItem {
  id          String    @id @default(uuid())
  milestoneId String
  category    String
  name        String
  quantity    Decimal   @db.Decimal(10, 2)
  unit        String
  unitPrice   Decimal   @db.Decimal(15, 2)
  totalPrice  Decimal   @db.Decimal(15, 2)
  actualCost  Decimal?  @db.Decimal(15, 2)
  notes       String?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  milestone   Milestone @relation(fields: [milestoneId], references: [id], onDelete: Cascade)
  
  @@index([milestoneId])
  @@map("boq_items")
}

// ============================================================================
// TABLE 12: EscrowAccount
// ============================================================================

model EscrowAccount {
  id             String   @id @default(uuid())
  projectId      String   @unique
  balance        Decimal  @default(0) @db.Decimal(15, 2)
  lockedBalance  Decimal  @default(0) @db.Decimal(15, 2)
  currency       String   @default("RWF")
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  
  project        Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  transactions   Transaction[]
  
  @@map("escrow_accounts")
}

// ============================================================================
// TABLE 13: Transaction
// ============================================================================

model Transaction {
  id               String            @id @default(uuid())
  escrowAccountId  String
  milestoneId      String?
  actorId          String
  type             TransactionType
  method           PaymentMethod?
  amount           Decimal           @db.Decimal(15, 2)
  status           TransactionStatus @default(pending)
  reference        String?
  metadata         Json?
  completedAt      DateTime?
  createdAt        DateTime          @default(now())
  updatedAt        DateTime          @updatedAt
  
  escrowAccount    EscrowAccount     @relation(fields: [escrowAccountId], references: [id])
  milestone        Milestone?        @relation(fields: [milestoneId], references: [id])
  actor            User              @relation(fields: [actorId], references: [id])
  
  @@index([escrowAccountId])
  @@index([milestoneId])
  @@index([actorId])
  @@index([status])
  @@map("transactions")
}

// ============================================================================
// TABLE 14: Rfq
// ============================================================================

model Rfq {
  id              String    @id @default(uuid())
  projectId       String
  milestoneId     String
  engineerId      String
  title           String
  specs           Json      @default("{}")
  quantity        Decimal   @db.Decimal(10, 2)
  unit            String
  deadline        DateTime
  status          RfqStatus @default(open)
  expiresAt       DateTime?
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt

  project         Project   @relation(fields: [projectId], references: [id])
  milestone       Milestone @relation(fields: [milestoneId], references: [id])
  engineer        User      @relation("RFQCreatedBy", fields: [engineerId], references: [id])
  
  quotes          Quote[]
  purchaseOrder   PurchaseOrder?

  @@index([projectId])
  @@index([milestoneId])
  @@index([status])
  @@map("rfqs")
}

// ============================================================================
// TABLE 15: Quote
// ============================================================================

model Quote {
  id             String      @id @default(uuid())
  rfqId          String
  supplierId     String
  unitPrice      Decimal     @db.Decimal(15, 2)
  totalPrice     Decimal     @db.Decimal(15, 2)
  deliveryDays   Int
  warrantyMonths Int?
  terms          String?
  certUrls       Json        @default("[]")
  status         QuoteStatus @default(pending_selection)
  selectedAt     DateTime?
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt

  rfq            Rfq         @relation(fields: [rfqId], references: [id])
  supplier       User        @relation("QuoteSubmittedBy", fields: [supplierId], references: [id])
  purchaseOrder  PurchaseOrder?

  @@index([rfqId])
  @@index([supplierId])
  @@index([status])
  @@map("quotes")
}

// ============================================================================
// TABLE 16: PurchaseOrder
// ============================================================================

model PurchaseOrder {
  id             String              @id @default(uuid())
  rfqId          String              @unique
  quoteId        String              @unique
  supplierId     String
  poNumber       String              @unique
  cloudinaryUrl  String
  status         PurchaseOrderStatus @default(issued)
  issuedAt       DateTime            @default(now())
  acceptedAt     DateTime?
  completedAt    DateTime?
  createdAt      DateTime            @default(now())
  updatedAt      DateTime            @updatedAt

  rfq            Rfq                 @relation(fields: [rfqId], references: [id])
  quote          Quote               @relation(fields: [quoteId], references: [id])
  supplier       User                @relation(fields: [supplierId], references: [id])
  deliveries     Delivery[]

  @@index([supplierId])
  @@index([status])
  @@map("purchase_orders")
}

// ============================================================================
// TABLE 17: Delivery
// ============================================================================

model Delivery {
  id              String         @id @default(uuid())
  purchaseOrderId String
  supplierId      String
  status          DeliveryStatus @default(preparing)
  startGps        Json?
  endGps          Json?
  proofPhotos     Json           @default("[]")
  notes           String?
  rejectionReason String?
  startedAt       DateTime?
  arrivedAt       DateTime?
  confirmedAt     DateTime?
  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt

  purchaseOrder   PurchaseOrder  @relation(fields: [purchaseOrderId], references: [id])
  supplier        User           @relation(fields: [supplierId], references: [id])

  @@index([purchaseOrderId])
  @@index([supplierId])
  @@index([status])
  @@map("deliveries")
}

// ============================================================================
// TABLE 18: ProgressPhoto
// ============================================================================

model ProgressPhoto {
  id                 String    @id @default(uuid())
  projectId          String
  milestoneId        String?
  uploadedById       String
  cloudinaryUrl      String
  cloudinaryPublicId String
  gpsLocation        Json?
  caption            String?
  isVideo            Boolean   @default(false)
  videoDuration      Int?
  createdAt          DateTime  @default(now())

  project            Project   @relation(fields: [projectId], references: [id])
  milestone          Milestone? @relation(fields: [milestoneId], references: [id])
  uploadedBy         User      @relation(fields: [uploadedById], references: [id])

  @@index([projectId])
  @@index([milestoneId])
  @@index([createdAt])
  @@map("progress_photos")
}

// ============================================================================
// TABLE 19: Inspection
// ============================================================================

model Inspection {
  id            String              @id @default(uuid())
  milestoneId   String
  supervisorId  String
  checklist     Json                @default("{}")
  photos        Json                @default("[]")
  rating        Int?
  signatureUrl  String?
  notes         String?
  decision      InspectionDecision?
  attemptNumber Int                 @default(1)
  signedAt      DateTime?
  createdAt     DateTime            @default(now())
  updatedAt     DateTime            @updatedAt

  milestone     Milestone           @relation(fields: [milestoneId], references: [id])
  supervisor    User                @relation(fields: [supervisorId], references: [id])

  @@unique([milestoneId, attemptNumber])
  @@index([milestoneId])
  @@index([supervisorId])
  @@map("inspections")
}

// ============================================================================
// TABLE 20: Dispute
// ============================================================================

model Dispute {
  id              String          @id @default(uuid())
  projectId       String
  milestoneId     String?
  raisedById      String
  category        DisputeCategory
  description     String
  status          DisputeStatus   @default(open)
  amountInDispute Decimal         @db.Decimal(15, 2)
  resolution      Json?
  resolvedAt      DateTime?
  resolvedBy      String?
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt

  project         Project         @relation(fields: [projectId], references: [id])
  milestone       Milestone?      @relation(fields: [milestoneId], references: [id])
  raisedBy        User            @relation("DisputeRaisedBy", fields: [raisedById], references: [id])
  
  evidence        DisputeEvidence[]

  @@index([projectId])
  @@index([milestoneId])
  @@index([status])
  @@map("disputes")
}

// ============================================================================
// TABLE 21: DisputeEvidence
// ============================================================================

model DisputeEvidence {
  id             String   @id @default(uuid())
  disputeId      String
  uploadedById   String
  cloudinaryUrl  String
  description    String?
  createdAt      DateTime @default(now())

  dispute        Dispute  @relation(fields: [disputeId], references: [id], onDelete: Cascade)
  uploadedBy     User     @relation(fields: [uploadedById], references: [id])

  @@index([disputeId])
  @@map("dispute_evidence")
}

// ============================================================================
// TABLE 22: Message
// ============================================================================

model Message {
  id         String   @id @default(uuid())
  projectId  String
  senderId   String
  content    String
  photoUrl   String?
  createdAt  DateTime @default(now())

  project    Project  @relation(fields: [projectId], references: [id])
  sender     User     @relation(fields: [senderId], references: [id])

  @@index([projectId])
  @@index([senderId])
  @@map("messages")
}

// ============================================================================
// TABLE 23: Notification
// ============================================================================

model Notification {
  id             String             @id @default(uuid())
  userId         String
  channel        NotificationChannel
  title          String
  body           String
  data           Json               @default("{}")
  status         NotificationStatus @default(pending)
  sentAt         DateTime?
  deliveredAt    DateTime?
  readAt         DateTime?
  failureReason  String?
  createdAt      DateTime           @default(now())

  user           User               @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([status])
  @@index([createdAt])
  @@map("notifications")
}

// ============================================================================
// TABLE 24: AuditLog
// ============================================================================

model AuditLog {
  id          String   @id @default(uuid())
  actorId     String?
  action      String
  entityType  String
  entityId    String?
  oldValues   Json?
  newValues   Json?
  ipAddress   String?
  userAgent   String?
  result      String
  createdAt   DateTime @default(now())

  projectId   String?
  project     Project? @relation(fields: [projectId], references: [id])
  actor       User?    @relation(fields: [actorId], references: [id])

  @@index([actorId])
  @@index([entityType, entityId])
  @@index([createdAt])
  @@map("audit_logs")
}

// ============================================================================
// TABLE 25: ActivityLog
// ============================================================================

model ActivityLog {
  id         String   @id @default(uuid())
  userId     String
  action     String
  metadata   Json?
  ipAddress  String?
  createdAt  DateTime @default(now())

  user       User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([createdAt])
  @@map("activity_logs")
}

// ============================================================================
// TABLE 26: ApiKey
// ============================================================================

model ApiKey {
  id          String    @id @default(uuid())
  userId      String
  name        String
  keyHash     String    @unique
  prefix      String
  permissions Json      @default("[]")
  expiresAt   DateTime?
  lastUsedAt  DateTime?
  revokedAt   DateTime?
  createdAt   DateTime  @default(now())

  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("api_keys")
}

// ============================================================================
// TABLE 27: SystemSetting
// ============================================================================

model SystemSetting {
  id          String   @id @default(uuid())
  key         String   @unique
  value       Json
  description String?
  updatedBy   String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("system_settings")
}

// ============================================================================
// TABLE 28: EmailTemplate
// ============================================================================

model EmailTemplate {
  id          String   @id @default(uuid())
  name        String   @unique
  subject     String
  htmlContent String
  plainText   String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("email_templates")
}
```

## Entity Relationship Diagram (ERD)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              INKINGIPRO DATABASE ERD                                  │
│                                   Version 1.0                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘

                                    ┌─────────────┐
                                    │    User     │
                                    │─────────────│
                                    │ id (PK)     │
                                    │ name        │
                                    │ email       │
                                    │ phoneNumber │
                                    │ role        │
                                    │ kycStatus   │
                                    └──────┬──────┘
                                           │
              ┌────────────────────────────┼────────────────────────────┐
              │                            │                            │
              ▼                            ▼                            ▼
    ┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
    │    Session      │          │    Account      │          │  RefreshToken   │
    │─────────────────│          │─────────────────│          │─────────────────│
    │ id (PK)         │          │ id (PK)         │          │ id (PK)         │
    │ userId (FK)─────┼──────────┼ userId (FK)─────┼──────────┼ userId (FK)─────┼───┐
    │ token           │          │ providerId      │          │ token           │   │
    │ expiresAt       │          │ accessToken     │          │ expiresAt       │   │
    └─────────────────┘          └─────────────────┘          └─────────────────┘   │
                                                                                     │
    ┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐   │
    │  Verification   │          │  PasswordReset  │          │   ApiKey        │   │
    │─────────────────│          │─────────────────│          │─────────────────│   │
    │ id (PK)         │          │ id (PK)         │          │ id (PK)         │   │
    │ identifier      │          │ userId (FK)─────┼──────────┼ userId (FK)─────┼───┘
    │ value           │          │ token           │          │ keyHash         │
    │ expiresAt       │          │ expiresAt       │          │ permissions     │
    └─────────────────┘          └─────────────────┘          └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    KYC DOMAIN                                        │
└─────────────────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────┐
                    │      User       │
                    └────────┬────────┘
                             │ 1
                             │
                             │ *
                    ┌────────▼────────┐
                    │  KycDocument    │
                    │─────────────────│
                    │ id (PK)         │
                    │ userId (FK)     │
                    │ type            │
                    │ cloudinaryUrl   │
                    │ status          │
                    └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                  PROJECT DOMAIN                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐                    ┌─────────────────┐
    │      User       │                    │      User       │
    │   (as Client)   │                    │  (as Engineer)  │
    └────────┬────────┘                    └────────┬────────┘
             │ 1                                     │ 0..1
             │                                       │
             │ ┌─────────────────────────────────────┘
             │ │
             ▼ ▼
    ┌─────────────────┐          1..*      ┌─────────────────┐
    │     Project     │───────────────────▶│  ProjectMember  │
    │─────────────────│                    │─────────────────│
    │ id (PK)         │                    │ id (PK)         │
    │ clientId (FK)───┼────┐               │ projectId (FK)──┼────┐
    │ engineerId (FK)─┼────┼───────────────│ userId (FK)──────┼────┼───┐
    │ name            │    │               │ role            │    │   │
    │ budget          │    │               │ status          │    │   │
    │ status          │    │               └─────────────────┘    │   │
    └────────┬────────┘    │                                       │   │
             │             │                                       │   │
             │ 1           │                                       │   │
             │             │                                       │   │
             ▼             ▼                                       │   │
    ┌─────────────────┐ ┌─────────────────┐                        │   │
    │   EscrowAccount │ │    Milestone    │◄───────────────────────┘   │
    │─────────────────│ │─────────────────│                            │
    │ id (PK)         │ │ id (PK)         │                            │
    │ projectId (FK)──┼─│ projectId (FK)──┼────────────────────────────┘
    │ balance         │ │ engineerId (FK) │
    │ lockedBalance   │ │ name            │
    └─────────────────┘ │ budgetPercentage│
                        │ status          │
                        └────────┬────────┘
                                 │ 1
                                 │
                                 │ *
                    ┌────────────▼────────────┐
                    │                       │
          ┌─────────┴─────────┐     ┌────────┴─────────┐
          │                   │     │                  │
    ┌─────▼─────┐       ┌──────▼──────┐       ┌────────▼────────┐
    │  BoqItem  │       │ Transaction │       │  Inspection     │
    │───────────│       │─────────────│       │────────────────│
    │ id (PK)   │       │ id (PK)     │       │ id (PK)         │
    │ milestone │       │ escrowAccId │       │ milestoneId(FK) │
    │ name      │       │ milestoneId │       │ supervisorId    │
    │ quantity  │       │ amount      │       │ decision        │
    └───────────┘       │ type        │       └─────────────────┘
                        └─────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                               SUPPLY CHAIN DOMAIN                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐           ┌─────────────────┐
    │    Milestone    │           │      User       │
    │                 │           │   (Engineer)    │
    └────────┬────────┘           └────────┬────────┘
             │ 1                            │ 1
             │                              │
             ▼                              ▼
    ┌─────────────────┐           ┌─────────────────┐
    │       Rfq       │           │       Rfq       │
    │─────────────────│           │─────────────────│
    │ id (PK)         │           │ id (PK)         │
    │ projectId (FK)  │           │ engineerId (FK) │
    │ milestoneId(FK)─┼───────────│ title           │
    │ engineerId (FK) │           │ specs           │
    │ status          │           │ deadline        │
    └────────┬────────┘           └─────────────────┘
             │ 1
             │
             │ *
    ┌────────▼────────┐
    │     Quote       │
    │─────────────────│
    │ id (PK)         │
    │ rfqId (FK)      │
    │ supplierId (FK) │
    │ unitPrice       │
    │ status          │
    └────────┬────────┘
             │ 1
             │
             │ 1
    ┌────────▼────────────┐
    │   PurchaseOrder     │
    │─────────────────────│
    │ id (PK)             │
    │ rfqId (FK)          │
    │ quoteId (FK)        │
    │ poNumber            │
    │ cloudinaryUrl       │
    │ status              │
    └────────┬────────────┘
             │ 1
             │
             │ *
    ┌────────▼────────┐
    │    Delivery     │
    │─────────────────│
    │ id (PK)         │
    │ purchaseOrderId │
    │ supplierId (FK) │
    │ status          │
    │ startGps        │
    │ endGps          │
    └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              PROGRESS & QUALITY DOMAIN                               │
└─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐           ┌─────────────────┐
    │     Project     │           │    Milestone    │
    └────────┬────────┘           └────────┬────────┘
             │ 1                            │ 1
             │                              │
             │ *                            │ *
    ┌────────▼────────┐           ┌─────────▼─────────┐
    │  ProgressPhoto  │           │   Inspection     │
    │─────────────────│           │──────────────────│
    │ id (PK)         │           │ id (PK)          │
    │ projectId (FK)  │           │ milestoneId (FK) │
    │ milestoneId(FK) │           │ supervisorId (FK)│
    │ uploadedById(FK)│           │ checklist        │
    │ cloudinaryUrl   │           │ rating           │
    └─────────────────┘           └──────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                 DISPUTE DOMAIN                                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐           ┌─────────────────┐
    │     Project     │           │    Milestone    │
    └────────┬────────┘           └────────┬────────┘
             │ 1                            │ 0..1
             │                              │
             ▼                              ▼
    ┌─────────────────┐           ┌─────────────────┐
    │     Dispute     │           │     Dispute     │
    │─────────────────│           │─────────────────│
    │ id (PK)         │           │ id (PK)         │
    │ projectId (FK)  │           │ milestoneId(FK) │
    │ raisedById (FK) │           │ status          │
    │ category        │           │ amountInDispute │
    │ description     │           └────────┬────────┘
    └────────┬────────┘                    │
             │ 1                           │
             │                             │ *
             │ *                           │
    ┌────────▼────────┐           ┌─────────▼─────────┐
    │ DisputeEvidence │           │   DisputeEvidence │
    │─────────────────│           │───────────────────│
    │ id (PK)         │           │ id (PK)           │
    │ disputeId (FK)  │           │ disputeId (FK)    │
    │ uploadedById(FK)│           │ cloudinaryUrl     │
    │ cloudinaryUrl   │           │ description       │
    └─────────────────┘           └───────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              COMMUNICATION DOMAIN                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐           ┌─────────────────┐
    │     Project     │           │      User       │
    └────────┬────────┘           └────────┬────────┘
             │ 1                            │ 1
             │                              │
             │ *                            │ *
    ┌────────▼────────┐           ┌─────────▼─────────┐
    │     Message     │           │   Notification    │
    │─────────────────│           │───────────────────│
    │ id (PK)         │           │ id (PK)           │
    │ projectId (FK)  │           │ userId (FK)       │
    │ senderId (FK)   │           │ title             │
    │ content         │           │ body              │
    │ photoUrl        │           │ channel           │
    └─────────────────┘           │ status            │
                                  └───────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                 LOGGING DOMAIN                                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────┐           ┌─────────────────┐
    │      User       │           │     Project     │
    └────────┬────────┘           └────────┬────────┘
             │ 0..1                         │ 0..1
             │                              │
             ▼                              ▼
    ┌─────────────────┐           ┌─────────────────┐
    │    AuditLog     │           │    AuditLog     │
    │─────────────────│           │─────────────────│
    │ id (PK)         │           │ id (PK)         │
    │ actorId (FK)────┼───────────│ projectId (FK)  │
    │ action          │           │ entityType      │
    │ entityType      │           │ entityId        │
    │ oldValues       │           │ oldValues       │
    │ newValues       │           │ newValues       │
    └─────────────────┘           └─────────────────┘

    ┌─────────────────┐
    │  ActivityLog    │
    │─────────────────│
    │ id (PK)         │
    │ userId (FK)─────┼───┐
    │ action          │   │
    │ metadata        │   │
    │ ipAddress       │   │
    └─────────────────┘   │
                          │
    ┌─────────────────┐   │
    │      User       │   │
    └─────────────────┘   │
                          │
    ┌─────────────────┐   │
    │   SystemSetting │   │
    │─────────────────│   │
    │ id (PK)         │   │
    │ key             │   │
    │ value           │   │
    └─────────────────┘   │
                          │
    ┌─────────────────┐   │
    │  EmailTemplate  │   │
    │─────────────────│   │
    │ id (PK)         │   │
    │ name            │   │
    │ subject         │   │
    │ htmlContent     │   │
    └─────────────────┘   │
                          │
                          │
                     ┌────┴────┐
                     │  User   │
                     └─────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              LEGEND                                                  │
├─────────────────────────────────────────────────────────────────────────────────────┤
│  PK  = Primary Key                                                                   │
│  FK  = Foreign Key                                                                   │
│  1   = One-to-One relationship                                                       │
│  *   = Many-to-Many relationship                                                     │
│  0..1= Zero or One relationship                                                      │
│  1..*= One to Many relationship                                                      │
│  ────= Foreign Key relationship                                                      │
│  ─ ─ = Optional relationship                                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow Diagram (DFD) - Level 0 (Context Diagram)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│                              ┌─────────────────────────┐                             │
│                              │                         │                             │
│                              │    INKINGIPRO SYSTEM    │                             │
│                              │                         │                             │
│                              └───────────┬─────────────┘                             │
│                                          │                                           │
│         ┌────────────────────────────────┼────────────────────────────────┐          │
│         │                                │                                │          │
│         ▼                                ▼                                ▼          │
│   ┌───────────┐                   ┌───────────┐                   ┌───────────┐     │
│   │  Client   │                   │ Engineer  │                   │ Supervisor│     │
│   └───────────┘                   └───────────┘                   └───────────┘     │
│                                                                                      │
│   ┌───────────┐                   ┌───────────┐                   ┌───────────┐     │
│   │ Supplier  │                   │   Admin   │                   │  Visitor  │     │
│   └───────────┘                   └───────────┘                   └───────────┘     │
│                                                                                      │
│   External Entities:                                                                 │
│   - Client: Creates projects, deposits funds, approves milestones                    │
│   - Engineer: Creates BOQ, RFQs, manages milestones                                  │
│   - Supervisor: Conducts inspections, reviews deliveries                             │
│   - Supplier: Responds to RFQs, manages deliveries                                   │
│   - Admin: Manages users, KYC, disputes, system settings                             │
│   - Visitor: Views public info, registers as new user                                │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow Diagram (DFD) - Level 1 (Major Processes)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│                                    EXTERNAL ENTITIES                                 │
│                                                                                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │ Client  │  │Engineer │  │Supervisor│ │Supplier │  │ Admin   │  │Visitor  │       │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘       │
│       │            │            │            │            │            │           │
│       └────────────┴────────────┴────────────┴────────────┴────────────┘           │
│                                          │                                           │
│                                          ▼                                           │
│                          ┌─────────────────────────────┐                            │
│                          │                             │                            │
│                          │         API GATEWAY         │                            │
│                          │      (Authentication &      │                            │
│                          │         Rate Limiting)      │                            │
│                          │                             │                            │
│                          └─────────────┬───────────────┘                            │
│                                        │                                             │
│         ┌──────────────────────────────┼──────────────────────────────┐             │
│         │                              │                              │             │
│         ▼                              ▼                              ▼             │
│  ┌──────────────┐              ┌──────────────┐              ┌──────────────┐       │
│  │ PROCESS 1    │              │ PROCESS 2    │              │ PROCESS 3    │       │
│  │ User & KYC   │              │ Project &    │              │ Supply Chain │       │
│  │ Management   │              │ Milestone    │              │ Management   │       │
│  │              │              │ Management   │              │              │       │
│  └──────┬───────┘              └──────┬───────┘              └──────┬───────┘       │
│         │                             │                             │               │
│         ▼                             ▼                             ▼               │
│  ┌──────────────┐              ┌──────────────┐              ┌──────────────┐       │
│  │ PROCESS 4    │              │ PROCESS 5    │              │ PROCESS 6    │       │
│  │ Payment &    │              │ Quality &    │              │ Dispute &    │       │
│  │ Escrow       │              │ Inspection   │              │ Resolution   │       │
│  │              │              │              │              │              │       │
│  └──────┬───────┘              └──────┬───────┘              └──────┬───────┘       │
│         │                             │                             │               │
│         └─────────────────────────────┼─────────────────────────────┘               │
│                                       │                                             │
│                                       ▼                                             │
│                          ┌─────────────────────────────┐                            │
│                          │                             │                            │
│                          │       DATA STORES           │                            │
│                          │                             │                            │
│                          │  ┌─────────────────────┐    │                            │
│                          │  │ PostgreSQL Database │    │                            │
│                          │  │    (25 Tables)      │    │                            │
│                          │  └─────────────────────┘    │                            │
│                          │                             │                            │
│                          │  ┌─────────────────────┐    │                            │
│                          │  │     Cloudinary      │    │                            │
│                          │  │   (Media Storage)   │    │                            │
│                          │  └─────────────────────┘    │                            │
│                          │                             │                            │
│                          │  ┌─────────────────────┐    │                            │
│                          │  │   Redis (Cache)     │    │                            │
│                          │  │   & Session Store   │    │                            │
│                          │  └─────────────────────┘    │                            │
│                          │                             │                            │
│                          └─────────────────────────────┘                            │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow Diagram (DFD) - Level 2 (Process Details)

### PROCESS 1: User & KYC Management

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         PROCESS 1: USER & KYC MANAGEMENT                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   ┌─────────┐    1.1 Register      ┌──────────────┐                                 │
│   │ Visitor │─────────────────────▶│ Create User  │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    1.2 Login         ┌──────▼───────┐                                 │
│   │  User   │─────────────────────▶│ Authenticate│                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    1.3 Upload KYC    ┌──────▼───────┐    ┌─────────────┐              │
│   │  User   │─────────────────────▶│ Submit KYC  │───▶│ Cloudinary  │              │
│   └─────────┘                       └──────┬───────┘    └─────────────┘              │
│                                            │                                         │
│   ┌─────────┐    1.4 Review KYC    ┌──────▼───────┐                                 │
│   │ Admin   │─────────────────────▶│ Verify KYC  │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│                                            ▼                                         │
│                                   ┌─────────────────┐                               │
│                                   │   DATA STORES   │                               │
│                                   │─────────────────│                               │
│                                   │ • users         │                               │
│                                   │ • sessions      │                               │
│                                   │ • accounts      │                               │
│                                   │ • refresh_tokens│                               │
│                                   │ • kyc_documents │                               │
│                                   └─────────────────┘                               │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### PROCESS 2: Project & Milestone Management

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      PROCESS 2: PROJECT & MILESTONE MANAGEMENT                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   ┌─────────┐    2.1 Create        ┌──────────────┐                                 │
│   │ Client  │─────────────────────▶│   Project    │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    2.2 Invite        ┌──────▼───────┐                                 │
│   │ Client  │─────────────────────▶│ Add Member  │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    2.3 Create        ┌──────▼───────┐                                 │
│   │Engineer │─────────────────────▶│ Milestone   │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    2.4 Create BOQ    ┌──────▼───────┐                                 │
│   │Engineer │─────────────────────▶│  BOQ Items  │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    2.5 Activate      ┌──────▼───────┐                                 │
│   │ Client  │─────────────────────▶│  Milestone  │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│                                            ▼                                         │
│                                   ┌─────────────────┐                               │
│                                   │   DATA STORES   │                               │
│                                   │─────────────────│                               │
│                                   │ • projects      │                               │
│                                   │ • project_members│                              │
│                                   │ • milestones    │                               │
│                                   │ • boq_items     │                               │
│                                   └─────────────────┘                               │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### PROCESS 3: Supply Chain Management

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         PROCESS 3: SUPPLY CHAIN MANAGEMENT                           │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   ┌─────────┐    3.1 Create RFQ    ┌──────────────┐                                 │
│   │Engineer │─────────────────────▶│     RFQ      │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    3.2 Submit Quote  ┌──────▼───────┐                                 │
│   │Supplier │─────────────────────▶│    Quote    │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    3.3 Select Quote  ┌──────▼───────┐    ┌─────────────┐              │
│   │Engineer │─────────────────────▶│Purchase Order│───▶│ Cloudinary  │              │
│   └─────────┘                       └──────┬───────┘    └─────────────┘              │
│                                            │                                         │
│   ┌─────────┐    3.4 Start         ┌──────▼───────┐                                 │
│   │Supplier │─────────────────────▶│  Delivery   │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    3.5 Confirm       ┌──────▼───────┐                                 │
│   │Engineer │─────────────────────▶│  Delivery   │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│                                            ▼                                         │
│                                   ┌─────────────────┐                               │
│                                   │   DATA STORES   │                               │
│                                   │─────────────────│                               │
│                                   │ • rfqs          │                               │
│                                   │ • quotes        │                               │
│                                   │ • purchase_orders│                              │
│                                   │ • deliveries    │                               │
│                                   └─────────────────┘                               │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### PROCESS 4: Payment & Escrow Management

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        PROCESS 4: PAYMENT & ESCROW MANAGEMENT                        │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   ┌─────────┐    4.1 Deposit       ┌──────────────┐    ┌─────────────┐              │
│   │ Client  │─────────────────────▶│   Payment    │───▶│ MTN/Airtel  │              │
│   └─────────┘                       └──────┬───────┘    │   API       │              │
│                                            │            └─────────────┘              │
│   ┌─────────┐    4.2 Request       ┌──────▼───────┐                                 │
│   │Engineer │─────────────────────▶│   Release   │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    4.3 Approve       ┌──────▼───────┐                                 │
│   │ Client  │─────────────────────▶│   Release   │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    4.4 View          ┌──────▼───────┐                                 │
│   │  User   │─────────────────────▶│ Transaction │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│                                            ▼                                         │
│                                   ┌─────────────────┐                               │
│                                   │   DATA STORES   │                               │
│                                   │─────────────────│                               │
│                                   │ • escrow_accounts│                              │
│                                   │ • transactions  │                               │
│                                   └─────────────────┘                               │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### PROCESS 5: Quality & Inspection Management

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      PROCESS 5: QUALITY & INSPECTION MANAGEMENT                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   ┌─────────┐    5.1 Upload        ┌──────────────┐    ┌─────────────┐              │
│   │Engineer │─────────────────────▶│   Progress   │───▶│ Cloudinary  │              │
│   └─────────┘                       │    Photo     │    └─────────────┘              │
│                                     └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    5.2 Conduct       ┌──────▼───────┐                                 │
│   │Supervisor─────────────────────▶│ Inspection  │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    5.3 View          ┌──────▼───────┐                                 │
│   │ Client  │─────────────────────▶│  Inspection │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│                                            ▼                                         │
│                                   ┌─────────────────┐                               │
│                                   │   DATA STORES   │                               │
│                                   │─────────────────│                               │
│                                   │ • progress_photos│                              │
│                                   │ • inspections   │                               │
│                                   └─────────────────┘                               │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### PROCESS 6: Dispute & Resolution Management

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      PROCESS 6: DISPUTE & RESOLUTION MANAGEMENT                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   ┌─────────┐    6.1 Raise         ┌──────────────┐                                 │
│   │  User   │─────────────────────▶│   Dispute    │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    6.2 Upload        ┌──────▼───────┐    ┌─────────────┐              │
│   │  User   │─────────────────────▶│   Evidence   │───▶│ Cloudinary  │              │
│   └─────────┘                       └──────┬───────┘    └─────────────┘              │
│                                            │                                         │
│   ┌─────────┐    6.3 Review        ┌──────▼───────┐                                 │
│   │ Admin   │─────────────────────▶│   Dispute   │                                 │
│   └─────────┘                       └──────┬───────┘                                 │
│                                            │                                         │
│   ┌─────────┐    6.4 Resolve       ┌──────▼───────┐    ┌─────────────┐              │
│   │ Admin   │─────────────────────▶│ Resolution  │───▶│ Transaction │              │
│   └─────────┘                       └──────┬───────┘    └─────────────┘              │
│                                            │                                         │
│                                            ▼                                         │
│                                   ┌─────────────────┐                               │
│                                   │   DATA STORES   │                               │
│                                   │─────────────────│                               │
│                                   │ • disputes      │                               │
│                                   │ • dispute_evidence│                             │
│                                   └─────────────────┘                               │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

## Table Relationship Summary

| # | Table | Primary Key | Foreign Keys | Related Tables |
|---|-------|-------------|--------------|----------------|
| 1 | User | id | - | Session, Account, RefreshToken, PasswordReset, KycDocument, Project, ProjectMember, Milestone, Transaction, Rfq, Quote, PurchaseOrder, Delivery, ProgressPhoto, Inspection, Dispute, DisputeEvidence, Message, Notification, AuditLog, ActivityLog, ApiKey |
| 2 | Session | id | userId | User |
| 3 | Account | id | userId | User |
| 4 | Verification | id | - | - |
| 5 | RefreshToken | id | userId | User |
| 6 | PasswordReset | id | userId | User |
| 7 | KycDocument | id | userId | User |
| 8 | Project | id | clientId, engineerId | User, ProjectMember, EscrowAccount, Milestone, Rfq, ProgressPhoto, Dispute, Message, AuditLog |
| 9 | ProjectMember | id | projectId, userId | Project, User |
| 10 | Milestone | id | projectId, engineerId, dependsOn | Project, User, BoqItem, Inspection, Rfq, Transaction, ProgressPhoto, Dispute |
| 11 | BoqItem | id | milestoneId | Milestone |
| 12 | EscrowAccount | id | projectId | Project, Transaction |
| 13 | Transaction | id | escrowAccountId, milestoneId, actorId | EscrowAccount, Milestone, User |
| 14 | Rfq | id | projectId, milestoneId, engineerId | Project, Milestone, User, Quote, PurchaseOrder |
| 15 | Quote | id | rfqId, supplierId | Rfq, User, PurchaseOrder |
| 16 | PurchaseOrder | id | rfqId, quoteId, supplierId | Rfq, Quote, User, Delivery |
| 17 | Delivery | id | purchaseOrderId, supplierId | PurchaseOrder, User |
| 18 | ProgressPhoto | id | projectId, milestoneId, uploadedById | Project, Milestone, User |
| 19 | Inspection | id | milestoneId, supervisorId | Milestone, User |
| 20 | Dispute | id | projectId, milestoneId, raisedById | Project, Milestone, User, DisputeEvidence |
| 21 | DisputeEvidence | id | disputeId, uploadedById | Dispute, User |
| 22 | Message | id | projectId, senderId | Project, User |
| 23 | Notification | id | userId | User |
| 24 | AuditLog | id | actorId, projectId | User, Project |
| 25 | ActivityLog | id | userId | User |
| 26 | ApiKey | id | userId | User |
| 27 | SystemSetting | id | - | - |
| 28 | EmailTemplate | id | - | - |

## Key Relationships Explained

### 1. User to Project
- **One-to-Many**: A Client can have many Projects
- **One-to-Many**: An Engineer can be assigned to many Projects
- **Many-to-Many**: Users can be members of many Projects (via ProjectMember)

### 2. Project to Milestone
- **One-to-Many**: A Project has many Milestones
- Milestones have dependencies (self-reference via `dependsOn`)

### 3. Milestone to Escrow
- **One-to-One**: Each Project has one EscrowAccount
- **One-to-Many**: Each Milestone can have many Transactions

### 4. Rfq to Quote to PurchaseOrder
- **One-to-Many**: One Rfq has many Quotes
- **One-to-One**: One Quote becomes one PurchaseOrder
- **One-to-Many**: One PurchaseOrder has many Deliveries

### 5. Dispute to Evidence
- **One-to-Many**: One Dispute has many Evidence documents

## Indexing Strategy

```sql
-- Critical indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_kyc_status ON users(kyc_status);

CREATE INDEX idx_projects_client_id ON projects(client_id);
CREATE INDEX idx_projects_engineer_id ON projects(engineer_id);
CREATE INDEX idx_projects_status ON projects(status);

CREATE INDEX idx_milestones_project_id ON milestones(project_id);
CREATE INDEX idx_milestones_status ON milestones(status);

CREATE INDEX idx_transactions_escrow_account_id ON transactions(escrow_account_id);
CREATE INDEX idx_transactions_status ON transactions(status);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_status ON notifications(status);

CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
```

This comprehensive database architecture provides:

1. **25+ tables** covering all business domains
2. **Complete relationship mapping** with proper foreign keys
3. **Visual ERD** showing all table connections
4. **4-level DFD** showing data flow through the system
5. **Indexing strategy** for performance optimization
6. **Role-based access control** built into the schema