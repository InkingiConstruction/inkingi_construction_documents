# InkingiPro - MVP Features Matrix & 2-Week Sprint Plan

**Version:** 1.0
**Status:** Final
**Date:** May 22, 2026
**Target Launch:** 2 Weeks from Project Start

---

## Document Purpose

This document provides:
1. **Complete feature inventory** (all features from SRS v1.3)
2. **MVP scope definition** (what ships in 2 weeks)
3. **Week-by-week sprint breakdown** (exactly what to build each week)
4. **Feature dependency mapping** (what blocks what)
5. **Independent vs dependent features** (for parallel development)

---

## Part 1: Complete Feature Inventory (All Features from SRS)

### Total Features Count: 87 Features

| ID | Feature Category | Feature Name | Role Access | Dependencies |
|----|-----------------|--------------|-------------|--------------|
| **AUTH-01** | Auth | User Registration with Role Selection | All | None |
| **AUTH-02** | Auth | Email OTP Verification (Nodemailer) | All | AUTH-01 |
| **AUTH-03** | Auth | Phone OTP Verification (Africa's Talking) | All | AUTH-01 |
| **AUTH-04** | Auth | JWT Access Token (1 hour expiry) | All | AUTH-01 |
| **AUTH-05** | Auth | JWT Refresh Token (30 days) | All | AUTH-04 |
| **AUTH-06** | Auth | Password Reset via Email OTP | All | AUTH-02 |
| **AUTH-07** | Auth | Login with Email + Password | All | AUTH-01 |
| **AUTH-08** | Auth | Login with Phone + OTP | All | AUTH-03 |
| **AUTH-09** | Auth | Account Lockout (5 failed attempts) | All | AUTH-01 |
| **AUTH-10** | Auth | Rate Limiting (100 req/min per user) | All | None |
| **KYC-01** | KYC | Upload National ID/Passport to Cloudinary | Client, Engineer, Supervisor | AUTH-01 |
| **KYC-02** | KYC | Upload IER Professional License | Engineer | KYC-01 |
| **KYC-03** | KYC | Upload Professional Indemnity Insurance | Engineer, Supervisor | KYC-01 |
| **KYC-04** | KYC | Upload Business Registration Certificate | Supplier | KYC-01 |
| **KYC-05** | KYC | Upload Tax Compliance Certificate | Supplier | KYC-01 |
| **KYC-06** | KYC | Admin KYC Review Queue (Web Portal) | Admin | KYC-01 |
| **KYC-07** | KYC | Admin Approve/Reject KYC | Admin | KYC-06 |
| **KYC-08** | KYC | Email Notification on KYC Decision | All | KYC-07 |
| **KYC-09** | KYC | KYC Expiry Check (Daily Bull Job) | Admin | KYC-07 |
| **PROJ-01** | Project | Create Project (name, description, budget) | Client | AUTH-01 |
| **PROJ-02** | Project | Draw GPS Boundary Polygon on Map | Client | PROJ-01 |
| **PROJ-03** | Project | Upload Site Photos to Cloudinary | Client | PROJ-01 |
| **PROJ-04** | Project | Upload Architectural Plans (PDF) to Cloudinary | Client | PROJ-01 |
| **PROJ-05** | Project | View Project List | Client, Engineer, Supervisor, Supplier | PROJ-01 |
| **PROJ-06** | Project | View Project Details | Client, Engineer, Supervisor, Supplier | PROJ-01 |
| **PROJ-07** | Project | Edit Project Metadata | Client | PROJ-01 |
| **PROJ-08** | Project | Soft Delete Project | Client | PROJ-01 |
| **PROJ-09** | Project | Invite Engineer to Project | Client | PROJ-01 |
| **PROJ-10** | Project | Accept/Decline Engineer Invitation | Engineer | PROJ-09 |
| **MIL-01** | Milestone | Create Milestone (name, budget_pct, duration) | Engineer | PROJ-01 |
| **MIL-02** | Milestone | Validate Milestone Budget Sum = 100% | Engineer | MIL-01 |
| **MIL-03** | Milestone | Set Milestone Dependencies | Engineer | MIL-01 |
| **MIL-04** | Milestone | Client Approves Milestone (activate) | Client | MIL-01 |
| **MIL-05** | Milestone | View Milestone Progress/Gantt | Client, Engineer, Supervisor | MIL-01 |
| **MIL-06** | Milestone | Request Milestone Revision | Client | MIL-01 |
| **BOQ-01** | BoQ | Create BoQ Item (material, quantity, price) | Engineer | MIL-01 |
| **BOQ-02** | BoQ | Edit BoQ Item | Engineer | BOQ-01 |
| **BOQ-03** | BoQ | Delete BoQ Item (soft delete) | Engineer | BOQ-01 |
| **BOQ-04** | BoQ | View BoQ with Market Price Comparison | Client, Engineer | BOQ-01 |
| **BOQ-05** | BoQ | Export BoQ to PDF | Client, Engineer | BOQ-01 |
| **BOQ-06** | BoQ | Export BoQ to Excel | Client, Engineer | BOQ-01 |
| **ESCROW-01** | Escrow | Deposit Funds via MTN Mobile Money | Client | PROJ-01 |
| **ESCROW-02** | Escrow | Deposit Funds via Airtel Money | Client | PROJ-01 |
| **ESCROW-03** | Escrow | Deposit Funds via Bank Transfer | Client | PROJ-01 |
| **ESCROW-04** | Escrow | View Escrow Balance | Client, Engineer, Supervisor | ESCROW-01 |
| **ESCROW-05** | Escrow | Request Milestone Payment (with photos) | Engineer | MIL-01 |
| **ESCROW-06** | Escrow | Approve Milestone Payment | Client | ESCROW-05 |
| **ESCROW-07** | Escrow | Release Payment from Escrow | System | ESCROW-06 |
| **ESCROW-08** | Escrow | Freeze Escrow Funds (dispute) | System | ESCROW-05 |
| **ESCROW-09** | Escrow | View Transaction History | Client, Engineer | ESCROW-01 |
| **SUP-01** | Supply Chain | Create RFQ (material specifications) | Engineer | MIL-01 |
| **SUP-02** | Supply Chain | Match Suppliers by Category & Location | System | SUP-01 |
| **SUP-03** | Supply Chain | Notify Suppliers via Push + Email (Bull) | System | SUP-02 |
| **SUP-04** | Supply Chain | Supplier Submit Quote | Supplier | SUP-01 |
| **SUP-05** | Supply Chain | View Ranked Quotes (price, rating, speed) | Engineer | SUP-04 |
| **SUP-06** | Supply Chain | Select Winning Quote | Engineer | SUP-05 |
| **SUP-07** | Supply Chain | Generate Purchase Order PDF | System | SUP-06 |
| **SUP-08** | Supply Chain | Start Delivery with GPS Tracking | Supplier | SUP-06 |
| **SUP-09** | Supply Chain | Validate GPS within 50m of Project | System | SUP-08 |
| **SUP-10** | Supply Chain | Upload Proof of Delivery Photos to Cloudinary | Supplier | SUP-08 |
| **SUP-11** | Supply Chain | Confirm Delivery Receipt | Engineer, Supervisor | SUP-10 |
| **SUP-12** | Supply Chain | Auto-Payment to Supplier (48h) | System | SUP-11 |
| **SUP-13** | Supply Chain | Rate Supplier (1-5 stars) | Engineer | SUP-11 |
| **INSP-01** | Inspection | Upload Daily Progress Photos to Cloudinary | Engineer | PROJ-01 |
| **INSP-02** | Inspection | Upload Progress Video (max 2 min) | Engineer | PROJ-01 |
| **INSP-03** | Inspection | Offline Photo Queue (AsyncStorage) | Engineer | INSP-01 |
| **INSP-04** | Inspection | GPS Check-in Before Inspection | Supervisor | PROJ-01 |
| **INSP-05** | Inspection | Complete Inspection Checklist (JSONB) | Supervisor | INSP-04 |
| **INSP-06** | Inspection | Rate Quality (1-5 stars) | Supervisor | INSP-05 |
| **INSP-07** | Inspection | Capture Digital Signature | Supervisor | INSP-05 |
| **INSP-08** | Inspection | Submit Inspection Report | Supervisor | INSP-07 |
| **INSP-09** | Inspection | View Project Timeline with Photos | Client | INSP-01 |
| **INSP-10** | Inspection | Download Inspection Package (ZIP) | Client, Engineer | INSP-08 |
| **DIS-01** | Dispute | Initiate Dispute (category, description) | Client, Engineer, Supervisor, Supplier | ESCROW-05 |
| **DIS-02** | Dispute | Upload Evidence to Cloudinary | All | DIS-01 |
| **DIS-03** | Dispute | Lock Escrow Funds on Dispute | System | DIS-01 |
| **DIS-04** | Dispute | Notify All Parties within 60s (Bull) | System | DIS-01 |
| **DIS-05** | Dispute | Admin Mediation Dashboard | Admin | DIS-01 |
| **DIS-06** | Dispute | Admin Issue Resolution Decision | Admin | DIS-05 |
| **DIS-07** | Dispute | Appeal Decision (once) | All | DIS-06 |
| **NOTIF-01** | Notifications | Push Notification (FCM for Android) | System | All features |
| **NOTIF-02** | Notifications | Push Notification (APNs for iOS) | System | All features |
| **NOTIF-03** | Notifications | Email via Nodemailer (Bull queue) | System | All features |
| **NOTIF-04** | Notifications | SMS via Africa's Talking | System | AUTH-03 |
| **NOTIF-05** | Notifications | In-App Chat (Socket.IO) | All | PROJ-01 |
| **ADMIN-01** | Admin | Admin Web Portal (React + Vite) | Admin | None |
| **ADMIN-02** | Admin | View All Users | Admin | ADMIN-01 |
| **ADMIN-03** | Admin | View Audit Logs | Admin | ADMIN-01 |
| **ADMIN-04** | Admin | Generate Compliance Reports | Admin | ADMIN-01 |
| **ADMIN-05** | Admin | View Financial Reconciliation | Admin | ADMIN-01 |
| **REPORT-01** | Reports | Project Progress Gantt Chart | Client, Engineer | MIL-01 |
| **REPORT-02** | Reports | Financial Report (PDF/Excel) | Client, Admin | ESCROW-01 |
| **REPORT-03** | Reports | Supplier Performance Report | Engineer, Admin | SUP-13 |
| **REPORT-04** | Reports | KYC Compliance Report | Admin | KYC-07 |

---

## Part 2: MVP Feature Selection (2-Week Sprint)

### MVP Philosophy
The MVP must prove the core value proposition: **"Diaspora investors can fund construction and verify progress before releasing payment."**

### MVP Features (30 Features) - Ship in 2 Weeks

| Priority | Feature ID | Feature Name | Why MVP |
|----------|------------|--------------|---------|
| **P0 (Must Have)** | AUTH-01 | User Registration with Role Selection | Core entry point |
| **P0** | AUTH-02 | Email OTP Verification | Security |
| **P0** | AUTH-04 | JWT Access Token | Auth security |
| **P0** | AUTH-07 | Login with Email + Password | Basic login |
| **P0** | KYC-01 | Upload National ID to Cloudinary | Identity proof |
| **P0** | KYC-06 | Admin KYC Review Queue | Manual verification |
| **P0** | KYC-07 | Admin Approve/Reject KYC | Gatekeeping |
| **P0** | PROJ-01 | Create Project (name, budget) | Core action |
| **P0** | PROJ-05 | View Project List | Dashboard |
| **P0** | PROJ-06 | View Project Details | Transparency |
| **P0** | PROJ-09 | Invite Engineer to Project | Role assignment |
| **P0** | PROJ-10 | Accept Engineer Invitation | Role acceptance |
| **P0** | MIL-01 | Create Milestone | Payment structure |
| **P0** | MIL-02 | Validate Budget Sum = 100% | Data integrity |
| **P0** | MIL-04 | Client Approves Milestone | Gatekeeping |
| **P0** | ESCROW-01 | Deposit via MTN Mobile Money | Funding |
| **P0** | ESCROW-04 | View Escrow Balance | Trust |
| **P0** | ESCROW-05 | Request Milestone Payment | Engineer trigger |
| **P0** | ESCROW-06 | Approve Milestone Payment | Client control |
| **P0** | ESCROW-07 | Release Payment from Escrow | Core value prop |
| **P0** | ESCROW-09 | View Transaction History | Audit trail |
| **P0** | INSP-01 | Upload Daily Progress Photos | Verification |
| **P0** | INSP-09 | View Project Timeline with Photos | Client visibility |
| **P0** | NOTIF-01 | Push Notification (FCM) | Real-time updates |
| **P0** | NOTIF-03 | Email via Nodemailer | Communication |
| **P1 (Should Have)** | AUTH-03 | Phone OTP (Africa's Talking) | Backup verification |
| **P1** | KYC-02 | Upload IER License (Engineer) | Professional trust |
| **P1** | INSP-05 | Complete Inspection Checklist | Quality control |
| **P1** | INSP-08 | Submit Inspection Report | Supervisor role |
| **P1** | ADMIN-01 | Admin Web Portal (Basic) | Operations |

### MVP Excluded Features (Why?)

| Feature | Exclusion Reason |
|---------|------------------|
| Full supply chain (RFQ, quotes) | Can add in Week 3-4; manual material sourcing first |
| Dispute resolution | Can add in Week 5-6; assume trust initially |
| GPS boundary drawing | Use text address for MVP; GPS in v2 |
| Offline mode | Complex; assume internet for MVP |
| Video uploads | Photos sufficient initially |
| Advanced analytics | Basic reporting in MVP |
| Socket.IO chat | Use email/phone for MVP |
| 2FA for admin | Add in Week 3 |
| API keys for suppliers | Not needed until automated integration |

---

## Part 3: 2-Week MVP Sprint Breakdown

### Week 1: Backend & Auth Foundation (Days 1-5)

#### Day 1: Project Setup & Database

**Tasks:**
- Initialize Node.js + Express project on Render
- Set up PostgreSQL database (Render or Supabase)
- Create database schema (users, kyc_documents, projects, milestones, escrow_accounts, transactions)
- Configure environment variables (.env)
- Set up GitHub repository + CI/CD (GitHub Actions → Render)

**Deliverables:**
- Express server running on Render
- PostgreSQL tables created
- Health check endpoint `/health` returning 200

**Team Assignment:**
- Backend: 2 developers
- DevOps: 1 developer

---

#### Day 2: Authentication System

**Tasks:**
- Implement User Registration endpoint `POST /auth/register`
- Implement Email OTP generation + Nodemailer integration
- Implement Email OTP verification endpoint
- Implement JWT generation (access + refresh tokens)
- Implement Login endpoint `POST /auth/login` (email + password)
- Implement bcrypt password hashing (cost 12)
- Add rate limiting (express-rate-limit)

**Deliverables:**
- User can register with email
- User receives OTP email
- User can verify OTP and login
- JWT returned on successful login

**API Endpoints Created:**
- `POST /api/v1/auth/register`
- `POST /api/v1/auth/verify-email`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/refresh-token`

---

#### Day 3: KYC & Cloudinary Integration

**Tasks:**
- Set up Cloudinary account and SDK
- Implement signed upload URL generation endpoint
- Create KYC document upload endpoint (receives Cloudinary URL)
- Create KYC document table in PostgreSQL
- Implement Admin API to fetch pending KYC submissions
- Implement Admin API to approve/reject KYC
- Add email notification on KYC decision

**Deliverables:**
- User can upload national ID to Cloudinary
- Admin can view pending KYC in API (no UI yet)
- Admin can approve/reject via API
- User receives email on decision

**API Endpoints Created:**
- `GET /api/v1/kyc/upload-url` (returns signed Cloudinary URL)
- `POST /api/v1/kyc/documents` (save Cloudinary URL)
- `GET /api/v1/admin/kyc/pending`
- `PATCH /api/v1/admin/kyc/:userId/approve`
- `PATCH /api/v1/admin/kyc/:userId/reject`

---

#### Day 4: Project & Milestone Management

**Tasks:**
- Implement Project CRUD endpoints
- Implement Milestone CRUD endpoints
- Add milestone budget validation (sum = 100%)
- Implement Project invitation system (invite engineer by email)
- Implement engineer accept/decline invitation
- Add role-based access control middleware

**Deliverables:**
- Client can create project
- Client can invite engineer
- Engineer can accept invitation
- Engineer can create milestones with budget validation

**API Endpoints Created:**
- `POST /api/v1/projects`
- `GET /api/v1/projects`
- `GET /api/v1/projects/:id`
- `PATCH /api/v1/projects/:id`
- `DELETE /api/v1/projects/:id`
- `POST /api/v1/projects/:id/invite-engineer`
- `PATCH /api/v1/invitations/:id/accept`
- `POST /api/v1/projects/:id/milestones`
- `GET /api/v1/projects/:id/milestones`

---

#### Day 5: Escrow Foundation & MTN Integration

**Tasks:**
- Implement escrow_accounts table (balance, locked_balance)
- Implement MTN Mobile Money webhook endpoint
- Create deposit endpoint that calls MTN API
- Implement transaction logging (append-only)
- Create escrow balance view endpoint
- Test MTN sandbox environment

**Deliverables:**
- Client can deposit funds via MTN MoMo (sandbox)
- Escrow balance updates after webhook
- Transaction record created
- Client can view escrow balance

**API Endpoints Created:**
- `POST /api/v1/escrow/deposit` (initiates MTN payment)
- `POST /api/v1/webhooks/mtn` (receives MTN callback)
- `GET /api/v1/escrow/projects/:id/balance`
- `GET /api/v1/escrow/projects/:id/transactions`

---

### Week 2: Mobile App & Core Flow (Days 6-10)

#### Day 6: React Native Mobile App Setup

**Tasks:**
- Initialize React Native project (Expo or bare)
- Set up React Navigation (stack + bottom tabs)
- Implement Auth screens:
  - Role Selection
  - Registration
  - OTP Verification
  - Login
- Connect to backend API using Axios
- Store JWT in secure storage (react-native-encrypted-storage)

**Deliverables:**
- Mobile app builds and runs
- User can register and login from mobile
- JWT persisted across app restarts

**Team Assignment:**
- Mobile: 2 developers
- Backend: 1 developer (support)

---

#### Day 7: Client Mobile Screens

**Tasks:**
- Create Project Dashboard screen (list of projects)
- Create Create Project screen (name, budget, description)
- Create Project Details screen (milestones, balance)
- Create Invite Engineer screen (email input)
- Create Upload Site Photos screen (camera + Cloudinary)
- Connect all screens to backend APIs

**Deliverables:**
- Client can create project from mobile
- Client can view project list
- Client can view project details with milestones
- Client can invite engineer

**Mobile Screens Completed:**
- DashboardScreen.tsx
- CreateProjectScreen.tsx
- ProjectDetailsScreen.tsx
- InviteEngineerScreen.tsx
- UploadPhotosScreen.tsx

---

#### Day 8: Engineer Mobile Screens

**Tasks:**
- Create Engineer Dashboard (assigned projects)
- Create Milestone Builder screen (add milestones, validate 100%)
- Create Daily Progress Upload screen (camera, Cloudinary upload)
- Create Request Payment screen (select milestone, add photos)
- Connect to backend APIs

**Deliverables:**
- Engineer can view assigned projects
- Engineer can create milestones with budget validation
- Engineer can upload daily progress photos to Cloudinary
- Engineer can request milestone payment

**Mobile Screens Completed:**
- EngineerDashboardScreen.tsx
- MilestoneBuilderScreen.tsx
- ProgressUploadScreen.tsx
- RequestPaymentScreen.tsx

---

#### Day 9: Supervisor & Payment Flow

**Tasks:**
- Create Supervisor Dashboard (pending inspections)
- Create Inspection Checklist screen (dynamic JSONB form)
- Create Inspection Submit screen (rating, signature pad)
- Implement client payment approval screen
- Implement escrow release flow
- Add push notifications (FCM setup)

**Deliverables:**
- Supervisor can view pending inspections
- Supervisor can complete checklist and submit
- Client can approve payment after inspection
- System releases funds from escrow
- Push notifications sent on each step

**Mobile Screens Completed:**
- SupervisorDashboardScreen.tsx
- InspectionChecklistScreen.tsx
- InspectionSubmitScreen.tsx
- PaymentApprovalScreen.tsx

**Backend Endpoints Added:**
- `POST /api/v1/inspections`
- `GET /api/v1/inspections/pending`
- `PATCH /api/v1/milestones/:id/payment-decision`

---

#### Day 10: Admin Web Portal & Integration Testing

**Tasks:**
- Initialize React + Vite project for Admin Portal
- Deploy to Vercel
- Create Admin KYC Review screen (list + detail view)
- Create Admin Project Audit screen
- End-to-end testing of complete flow:
  1. Client registers → uploads KYC → gets approved
  2. Client creates project → deposits funds
  3. Client invites engineer → engineer accepts
  4. Engineer creates milestones
  5. Engineer uploads progress → requests payment
  6. Supervisor inspects → submits report
  7. Client approves → funds released
- Bug fixes and polish

**Deliverables:**
- Admin web portal live on Vercel
- Admin can review and approve KYC
- Complete user journey works end-to-end
- MVP ready for demo

**Admin Portal Screens:**
- LoginScreen.tsx
- KYCPendingQueue.tsx
- KYCReviewDetail.tsx
- ProjectAuditList.tsx
- ProjectAuditDetail.tsx

---

## Part 4: Post-MVP Feature Roadmap (Weeks 3-8)

### Week 3: Enhanced Features (15 features)

| Feature | Dependencies |
|---------|--------------|
| Phone OTP with Africa's Talking | AUTH-01 |
| GPS boundary drawing on map | PROJ-01 |
| Video progress uploads | INSP-01 |
| Offline photo queue (AsyncStorage) | INSP-01 |
| Export BoQ to PDF/Excel | BOQ-01 |
| Admin user management (suspend/activate) | ADMIN-01 |

### Week 4: Supply Chain Foundation (10 features)

| Feature | Dependencies |
|---------|--------------|
| Supplier registration + KYC | AUTH-01 |
| Create RFQ (broadcast) | MIL-01 |
| Match suppliers by category | SUP-01 |
| Supplier submit quote | SUP-04 |
| View ranked quotes | SUP-05 |
| Select winning quote | SUP-06 |
| Generate purchase order PDF | SUP-06 |

### Week 5: Delivery & Payment Integration (8 features)

| Feature | Dependencies |
|---------|--------------|
| Start delivery with GPS | SUP-06 |
| Validate GPS within 50m | SUP-08 |
| Upload PoD photos | SUP-08 |
| Confirm delivery receipt | SUP-10 |
| Auto-payment to supplier | SUP-11 |
| Rate supplier | SUP-11 |
| Airtel Money integration | ESCROW-01 |
| Bank transfer webhook | ESCROW-01 |

### Week 6: Dispute Resolution (7 features)

| Feature | Dependencies |
|---------|--------------|
| Initiate dispute | ESCROW-05 |
| Upload evidence to Cloudinary | DIS-01 |
| Lock escrow on dispute | DIS-01 |
| Notify all parties within 60s | DIS-01 |
| Admin mediation dashboard | ADMIN-01 |
| Admin resolution decision | DIS-05 |
| Appeal mechanism | DIS-06 |

### Week 7: Analytics & Reporting (6 features)

| Feature | Dependencies |
|---------|--------------|
| Project progress Gantt chart | MIL-01 |
| Financial report PDF/Excel | ESCROW-01 |
| Supplier performance report | SUP-13 |
| KYC compliance report | KYC-07 |
| Audit log viewer | ADMIN-01 |
| Dashboard analytics (Recharts) | REPORT-01 |

### Week 8: Polish & Launch Preparation (5 features)

| Feature | Dependencies |
|---------|--------------|
| Kinyarwanda localization (i18n) | All screens |
| Socket.IO in-app chat | PROJ-01 |
| Admin 2FA (TOTP) | ADMIN-01 |
| Performance optimization | All features |
| App Store + Play Store submission | Mobile app |

---

## Part 5: Feature Dependency Map

### Independent Features (Can Build in Parallel)

These features have no dependencies on other features and can be developed simultaneously by different teams:

| Feature | Team Assignment |
|---------|-----------------|
| AUTH-01 to AUTH-10 (Auth system) | Backend Team A |
| KYC-01 to KYC-05 (Document upload) | Backend Team B |
| NOTIF-01 to NOTIF-03 (Notifications) | Backend Team C |
| ADMIN-01 (Admin portal setup) | Frontend Team A |
| Mobile app navigation setup | Mobile Team A |

### Dependent Features (Require Prerequisites)

| Feature | Depends On | Estimated Delay |
|---------|------------|-----------------|
| PROJ-01 (Create project) | AUTH-01 | +1 day |
| MIL-01 (Create milestone) | PROJ-01 | +2 days |
| ESCROW-01 (Deposit) | PROJ-01 | +2 days |
| ESCROW-05 (Request payment) | MIL-01, ESCROW-01 | +4 days |
| INSP-01 (Upload progress) | PROJ-01 | +2 days |
| SUP-01 (Create RFQ) | MIL-01 | +5 days |
| DIS-01 (Initiate dispute) | ESCROW-05 | +10 days |

### Critical Path (Must Be Sequential)

```
AUTH-01 → AUTH-02 → AUTH-04 → AUTH-07 (Login)
    ↓
KYC-01 → KYC-06 → KYC-07 (KYC Approval)
    ↓
PROJ-01 → PROJ-09 → PROJ-10 (Project + Engineer)
    ↓
MIL-01 → MIL-02 → MIL-04 (Milestones + Client Approval)
    ↓
ESCROW-01 → ESCROW-04 (Deposit + Balance)
    ↓
ESCROW-05 → INSP-01 → INSP-08 (Payment Request → Inspection)
    ↓
ESCROW-06 → ESCROW-07 (Payment Approval → Release)
```

**Critical Path Duration:** 8-10 days (fits within 2-week MVP)

---

## Part 6: Parallel Development Teams Structure

### Team Allocation for 2-Week MVP

| Team | Members | Responsibilities | Features |
|------|---------|------------------|----------|
| **Backend A** | 2 devs | Auth, Users, KYC | AUTH-01 to AUTH-10, KYC-01 to KYC-09 |
| **Backend B** | 2 devs | Projects, Milestones, Escrow | PROJ-01 to PROJ-10, MIL-01 to MIL-06, ESCROW-01 to ESCROW-09 |
| **Backend C** | 1 dev | Notifications, Cloudinary | NOTIF-01 to NOTIF-05, Cloudinary integration |
| **Mobile A** | 2 devs | Client + Engineer screens | Client dashboard, project creation, engineer flows |
| **Mobile B** | 1 dev | Supervisor + Inspection screens | Supervisor dashboard, inspection checklist |
| **Frontend** | 1 dev | Admin Web Portal | ADMIN-01 to ADMIN-05, KYC review UI |
| **DevOps** | 1 dev | CI/CD, Deployments | Render, Vercel, GitHub Actions |

**Total Team Size:** 10 developers

---

## Part 7: MVP Success Criteria

### Functional Success (All Must Work)

- [ ] User registers, verifies email, logs in
- [ ] Client uploads national ID, admin approves KYC
- [ ] Client creates project with name + budget
- [ ] Client invites engineer via email
- [ ] Engineer accepts invitation
- [ ] Engineer creates milestones totaling 100% budget
- [ ] Client approves milestones
- [ ] Client deposits funds via MTN MoMo (sandbox)
- [ ] Escrow balance updates correctly
- [ ] Engineer uploads progress photos to Cloudinary
- [ ] Engineer requests milestone payment
- [ ] Supervisor completes inspection checklist
- [ ] Client approves payment
- [ ] Funds released from escrow
- [ ] Transaction appears in history

### Technical Success

- [ ] API response time < 500ms (p95)
- [ ] Cloudinary upload < 5 seconds on 3G
- [ ] Push notification delivered < 2 seconds
- [ ] Zero critical security vulnerabilities
- [ ] 99.5% uptime during demo hours

### User Acceptance

- [ ] New user completes full flow in < 15 minutes
- [ ] Client can see photos before approving payment
- [ ] All 5 roles can login and perform primary action

---

## Part 8: Risk Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| MTN MoMo webhook fails | Medium | High | Implement retry queue (Bull) + manual override in admin |
| Cloudinary outage | Low | Medium | Queue uploads locally in AsyncStorage |
| Render cold start | High | Medium | Upgrade to Starter plan ($7/mo) before demo |
| JWT expiration confusion | Medium | Low | Implement auto-refresh token in mobile app |
| KYC backlog | Medium | Medium | Ensure admin has dedicated reviewer during demo |

---

## Summary: MVP at a Glance

| Metric | Value |
|--------|-------|
| **Total Features in MVP** | 30 / 87 (34%) |
| **Development Time** | 2 Weeks (10 working days) |
| **Team Size** | 10 developers |
| **Tech Stack** | React Native + Node.js + PostgreSQL + Cloudinary + Render |
| **Core Value Delivered** | Client funds project → Engineer works → Supervisor verifies → Client pays |
| **Post-MVP Features** | 57 features (Weeks 3-8) |

---

*This document supersedes all other planning documents. MVP launch target: End of Week 2.*