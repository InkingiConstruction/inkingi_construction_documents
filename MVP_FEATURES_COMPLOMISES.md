# InkingiPro - MVP Features Matrix & 2-Week Sprint Plan
## 3-Developer Team Edition

**Version:** 2.0
**Status:** Final
**Date:** May 22, 2026
**Team Size:** 3 Developers
**Target:** Fully Working MVP in 2 Weeks

---

## Executive Summary

With **3 developers** and **2 weeks**, we cannot build all 87 features. We must build the **absolute minimum viable product** that demonstrates the core value proposition:

> **"A client can deposit money, an engineer can upload photos, a supervisor can verify, and the client can release payment."**

Everything else is **cut, simplified, or faked** for the MVP.

---

## Part 1: What We MUST Cut (Real Talk)

| Feature | Why Cut |
|---------|---------|
| Full KYC with multiple documents | Use simple admin approval flag only |
| IER license verification | Manual check by admin |
| GPS polygon drawing | Use text address + manual verification |
| Offline mode | Assume internet connectivity |
| Video uploads | Photos only |
| RFQ / Quotes / Supply chain | Engineer manually contacts suppliers |
| Dispute resolution | Handle manually by admin via email |
| Advanced analytics | Basic counts only |
| Socket.IO chat | Use phone/email |
| Push notifications | Use email only (simpler) |
| Airtel Money | MTN only |
| Bank transfer | MTN only |
| Multiple document types | Single ID upload |
| BoQ export to PDF/Excel | View only in app |
| Digital signature | Checkbox + typed name |
| FCM/APNs | Skip, use email |
| Redis / Bull | Simplified queue using setTimeout |
| Separate background workers | Same process |

---

## Part 2: MVP Feature Set (20 Features Total)

### Core Features That MUST Work

| ID | Feature | Role | Complexity | Hours |
|----|---------|------|------------|-------|
| **1** | User Registration (email + password) | All | Medium | 4 |
| **2** | Email OTP Verification | All | Medium | 3 |
| **3** | Login (email + password) | All | Low | 2 |
| **4** | JWT Authentication | All | Low | 2 |
| **5** | Simple KYC (upload ID, admin approves) | All | Medium | 6 |
| **6** | Create Project (name, budget, address) | Client | Low | 3 |
| **7** | View Projects List | Client, Engineer | Low | 2 |
| **8** | Invite Engineer (by email) | Client | Medium | 4 |
| **9** | Accept Invitation | Engineer | Medium | 3 |
| **10** | Create Milestones (with 100% validation) | Engineer | Medium | 5 |
| **11** | Client Approves Milestones | Client | Low | 2 |
| **12** | Deposit Funds (MTN MoMo webhook) | Client | High | 8 |
| **13** | View Escrow Balance | Client, Engineer | Low | 2 |
| **14** | Upload Progress Photos (Cloudinary) | Engineer | Medium | 5 |
| **15** | Request Milestone Payment | Engineer | Low | 2 |
| **16** | View Pending Inspections | Supervisor | Low | 2 |
| **17** | Complete Inspection (checklist + rating) | Supervisor | Medium | 5 |
| **18** | Approve/Reject Payment | Client | Low | 2 |
| **19** | Release Funds from Escrow | System | Medium | 4 |
| **20** | Admin Web Portal (basic KYC + project view) | Admin | High | 10 |

**Total Estimated Hours:** 76 hours

**With 3 developers working 8 hours/day for 10 days = 240 available hours**

**Buffer:** 240 - 76 = 164 hours for debugging, integration, and polish ✅

---

## Part 3: Simplified Database Schema (MVP)

Only **8 tables** instead of 15+:

```sql
-- 1. Users (all roles)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  full_name TEXT NOT NULL,
  role TEXT CHECK (role IN ('client', 'engineer', 'supervisor', 'admin')) NOT NULL,
  kyc_status TEXT DEFAULT 'pending' CHECK (kyc_status IN ('pending', 'approved', 'rejected')),
  kyc_document_url TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 2. Projects
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  client_id UUID REFERENCES users(id) NOT NULL,
  engineer_id UUID REFERENCES users(id),
  name TEXT NOT NULL,
  budget INTEGER NOT NULL, -- RWF
  address TEXT NOT NULL,
  status TEXT DEFAULT 'draft' CHECK (status IN ('draft', 'active', 'completed')),
  created_at TIMESTAMP DEFAULT NOW()
);

-- 3. Milestones
CREATE TABLE milestones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) NOT NULL,
  title TEXT NOT NULL,
  budget_percentage INTEGER NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'active', 'completed', 'paid')),
  created_at TIMESTAMP DEFAULT NOW()
);

-- 4. Escrow Accounts
CREATE TABLE escrow_accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) UNIQUE NOT NULL,
  balance INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 5. Transactions
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) NOT NULL,
  milestone_id UUID REFERENCES milestones(id),
  amount INTEGER NOT NULL,
  type TEXT CHECK (type IN ('deposit', 'release', 'refund')) NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'completed', 'failed')),
  mtn_transaction_id TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 6. Progress Photos
CREATE TABLE progress_photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) NOT NULL,
  milestone_id UUID REFERENCES milestones(id),
  cloudinary_url TEXT NOT NULL,
  uploaded_by UUID REFERENCES users(id) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 7. Inspections
CREATE TABLE inspections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  milestone_id UUID REFERENCES milestones(id) UNIQUE NOT NULL,
  supervisor_id UUID REFERENCES users(id) NOT NULL,
  checklist_answers JSONB NOT NULL,
  rating INTEGER CHECK (rating BETWEEN 1 AND 5),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'rejected')),
  notes TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 8. Invitations
CREATE TABLE invitations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) NOT NULL,
  email TEXT NOT NULL,
  role TEXT NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'expired')),
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Part 4: 2-Week Daily Breakdown (3 Developers)

### Week 1: Backend + Core APIs

#### Day 1 (Monday) - Foundation

**All 3 developers together:**

| Time | Task | Who |
|------|------|-----|
| 9-11 AM | Setup project structure, Git, Render, PostgreSQL | All |
| 11-1 PM | Create Express server, health check, environment config | All |
| 2-4 PM | Create database schema (8 tables) | All |
| 4-6 PM | Deploy to Render, verify connection | All |

**End of Day 1:** Express server live on Render, database ready

---

#### Day 2 (Tuesday) - Authentication

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Registration endpoint + bcrypt password | 8 |
| Dev 2 | Email OTP with Nodemailer + verification endpoint | 8 |
| Dev 3 | Login endpoint + JWT generation | 8 |

**End of Day 2:** 
- User can register ✅
- User receives email OTP ✅
- User can verify and login ✅
- JWT returned ✅

---

#### Day 3 (Wednesday) - KYC + Projects

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Cloudinary setup + KYC upload endpoint | 8 |
| Dev 2 | Project CRUD endpoints | 8 |
| Dev 3 | Milestone endpoints + 100% validation | 8 |

**End of Day 3:**
- User can upload ID to Cloudinary ✅
- Client can create project ✅
- Engineer can create milestones ✅
- Validation works ✅

---

#### Day 4 (Thursday) - Invitations + Escrow Start

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Invite engineer by email + accept endpoint | 8 |
| Dev 2 | Escrow account creation + balance endpoints | 8 |
| Dev 3 | MTN MoMo webhook integration (sandbox) | 8 |

**End of Day 4:**
- Client can invite engineer ✅
- Engineer can accept ✅
- Escrow balance viewable ✅
- MTN deposit started (may need Day 5 polish) ✅

---

#### Day 5 (Friday) - Complete Escrow + Admin API

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Finish MTN webhook + transaction recording | 8 |
| Dev 2 | Payment request + release endpoints | 8 |
| Dev 3 | Admin API (view users, approve KYC, view projects) | 8 |

**End of Day 5 (Week 1):**
- Deposits work via MTN sandbox ✅
- Payment request flow ready ✅
- Admin API ready for frontend ✅

---

### Week 2: Mobile App + Admin Portal

#### Day 6 (Monday) - React Native Setup + Auth Screens

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | React Native project init + navigation setup | 8 |
| Dev 2 | Login + Registration screens | 8 |
| Dev 3 | OTP Verification screen | 8 |

**End of Day 6:**
- Mobile app runs on emulator ✅
- User can register and login from mobile ✅

---

#### Day 7 (Tuesday) - Client Mobile Screens

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Project List + Create Project screens | 8 |
| Dev 2 | Project Details + Milestone Approval screens | 8 |
| Dev 3 | Deposit screen + Escrow Balance view | 8 |

**End of Day 7:**
- Client can create project ✅
- Client can view milestones ✅
- Client can deposit funds ✅
- Client can approve milestones ✅

---

#### Day 8 (Wednesday) - Engineer + Supervisor Mobile Screens

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Engineer Dashboard + Accept Invitation | 8 |
| Dev 2 | Milestone Builder screen (with validation) | 8 |
| Dev 3 | Progress Photo Upload + Payment Request screens | 8 |

**End of Day 8:**
- Engineer can accept invitations ✅
- Engineer can create milestones ✅
- Engineer can upload photos to Cloudinary ✅
- Engineer can request payment ✅

---

#### Day 9 (Thursday) - Supervisor Screens + Complete Flow

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Supervisor Dashboard + Pending Inspections | 8 |
| Dev 2 | Inspection Checklist screen (simple form) | 8 |
| Dev 3 | Client Payment Approval screen + Integration | 8 |

**End of Day 9:**
- Supervisor can view pending inspections ✅
- Supervisor can complete checklist ✅
- Client can approve payment ✅
- **Full flow works end-to-end** ✅

---

#### Day 10 (Friday) - Admin Web Portal + Polish

| Developer | Task | Hours |
|-----------|------|-------|
| Dev 1 | Admin Portal (React + Vite) on Vercel | 8 |
| Dev 2 | KYC Review screen (list + approve/reject buttons) | 8 |
| Dev 3 | Bug fixes, testing, demo preparation | 8 |

**End of Day 10:**
- Admin portal live on Vercel ✅
- Admin can approve KYC ✅
- All bugs fixed ✅
- **MVP READY FOR DEMO** ✅

---

## Part 5: Simplified API Endpoints (MVP)

Only **20 endpoints** instead of 50+:

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register user |
| POST | `/api/auth/verify-otp` | Verify email OTP |
| POST | `/api/auth/login` | Login |
| POST | `/api/kyc/upload` | Upload ID to Cloudinary |
| GET | `/api/admin/kyc/pending` | Admin view pending KYC |
| POST | `/api/admin/kyc/:userId/approve` | Admin approve KYC |
| POST | `/api/admin/kyc/:userId/reject` | Admin reject KYC |
| GET | `/api/projects` | List user's projects |
| POST | `/api/projects` | Create project |
| GET | `/api/projects/:id` | Project details |
| POST | `/api/projects/:id/invite` | Invite engineer |
| POST | `/api/invitations/:id/accept` | Accept invitation |
| GET | `/api/projects/:id/milestones` | List milestones |
| POST | `/api/projects/:id/milestones` | Create milestone |
| POST | `/api/escrow/deposit` | Initiate MTN deposit |
| POST | `/api/webhooks/mtn` | MTN callback |
| GET | `/api/escrow/:projectId/balance` | Get balance |
| POST | `/api/milestones/:id/request-payment` | Request payment |
| POST | `/api/inspections` | Submit inspection |
| POST | `/api/milestones/:id/payment-decision` | Approve/reject payment |

---

## Part 6: Simplified Mobile Screens (MVP)

### Client Screens (5 screens)

1. **LoginScreen** - Email + password
2. **OTPScreen** - Enter 6-digit code
3. **ProjectListScreen** - Shows all projects with status
4. **CreateProjectScreen** - Name, budget, address only (no GPS)
5. **ProjectDetailScreen** - Milestones, balance, approve buttons
6. **DepositScreen** - Enter amount, call MTN

### Engineer Screens (4 screens)

1. **EngineerDashboardScreen** - Assigned projects + invitations
2. **MilestoneBuilderScreen** - Add milestones, shows total %
3. **ProgressUploadScreen** - Camera, upload to Cloudinary
4. **RequestPaymentScreen** - Select milestone, confirm

### Supervisor Screens (2 screens)

1. **SupervisorDashboardScreen** - Pending inspections list
2. **InspectionScreen** - Checklist (Yes/No questions), rating, notes

### Admin Web Portal (3 screens)

1. **KYCPendingScreen** - List users with pending KYC
2. **KYCReviewScreen** - View ID image, approve/reject buttons
3. **ProjectAuditScreen** - View all projects and transactions

---

## Part 7: What "Works" Means for MVP

| Feature | Definition of "Works" |
|---------|----------------------|
| Registration | User creates account, receives email OTP, verifies |
| Login | Email + password returns JWT |
| KYC | Admin sees uploaded ID, clicks Approve button |
| Create Project | Saves to database, appears in list |
| Invite Engineer | Email sent, engineer clicks link to accept |
| Milestones | Engineer creates, validates sum=100%, client approves |
| Deposit | User enters amount, MTN sandbox returns success, balance updates |
| Progress Photos | Camera opens, uploads to Cloudinary, appears in project |
| Payment Request | Engineer clicks button, status changes |
| Inspection | Supervisor sees request, answers Yes/No, submits |
| Payment Approval | Client clicks Approve, escrow balance decreases |
| Admin Portal | Can login, see pending KYC, approve users |

---

## Part 8: Risk Mitigation for 3 Developers

| Risk | Solution |
|------|----------|
| MTN sandbox fails | Hardcode a "demo deposit" button that adds balance |
| Cloudinary issues | Store photos locally as base64 in database (fallback) |
| Render cold start | Set keep-alive ping every 5 minutes |
| Email not sending | Log OTP to console for demo |
| Time running out | Cut features further (remove supervisor, admin does inspection) |

---

## Part 9: Absolute Minimum Viable (If Behind Schedule)

**If Day 5 is behind, cut to this:**

| Role | Minimum Features |
|------|------------------|
| Client | Register, login, create project, deposit (fake), approve payment |
| Engineer | Login, accept invite, create milestones, upload photos |
| Supervisor | Login, view inspection request, submit "approved" |
| Admin | Login, approve KYC manually |

**Fake deposit:** Add button "Add 1,000,000 RWF (Demo)" that directly updates escrow balance without MTN.

---

## Part 10: Success Checklist for Day 10 Demo

### Pre-Demo Setup (30 minutes)

- [ ] 5 test accounts created (Client, Engineer, Supervisor, Admin)
- [ ] 1 project created with 3 milestones
- [ ] Escrow funded with demo money
- [ ] All Cloudinary folders created
- [ ] Render services running
- [ ] Vercel admin portal deployed

### Demo Script (15 minutes)

| Time | Action | Who |
|------|--------|-----|
| 0:00 | Open mobile app, login as Client | Client |
| 1:00 | Create new project "Demo Villa" with budget 10,000,000 RWF | Client |
| 2:00 | Invite engineer@example.com | Client |
| 3:00 | Login as Engineer, accept invitation | Engineer |
| 4:00 | Create 3 milestones (30%, 40%, 30%) | Engineer |
| 5:00 | Login as Client, approve milestones | Client |
| 6:00 | Deposit 5,000,000 RWF via MTN (or demo button) | Client |
| 7:00 | Login as Engineer, upload 5 progress photos | Engineer |
| 8:00 | Request payment for Milestone 1 | Engineer |
| 9:00 | Login as Supervisor, complete inspection | Supervisor |
| 10:00 | Login as Client, approve payment | Client |
| 11:00 | Show escrow balance decreased by 3,000,000 RWF | Client |
| 12:00 | Login as Admin, show KYC approval flow | Admin |
| 13:00 | Show transaction history | Any |
| 14:00 | Q&A | All |

### Success Criteria (All Must Be Green)

- [ ] **Registration works** - User creates account, receives email
- [ ] **Login works** - JWT returned, user stays logged in
- [ ] **KYC works** - Admin approves in portal, user notified
- [ ] **Project creation works** - Saves to database
- [ ] **Engineer invitation works** - Email sent, acceptance works
- [ ] **Milestone validation works** - Blocks if sum ≠ 100%
- [ ] **Deposit works** - Balance increases
- [ ] **Photo upload works** - Cloudinary URL saved
- [ ] **Payment request works** - Status changes
- [ ] **Inspection works** - Checklist saved
- [ ] **Payment release works** - Balance decreases
- [ ] **No crashes** - App stays running

---

## Summary

| Metric | Value |
|--------|-------|
| **Team Size** | 3 developers |
| **Timeline** | 2 weeks (10 days) |
| **Total Features** | 20 (down from 87) |
| **Database Tables** | 8 (down from 15+) |
| **API Endpoints** | 20 (down from 50+) |
| **Mobile Screens** | 11 (Client 6, Engineer 4, Supervisor 2) |
| **Admin Screens** | 3 |
| **Success Rate** | Achievable with discipline |

---

**The ONLY question that matters on Day 10:** 

> *Can a client deposit money, an engineer upload photos, a supervisor verify, and the client release payment?*

If YES → MVP successful.
If NO → Keep cutting until yes.

---

*This is the real MVP plan for 3 developers in 2 weeks. No fluff. No nice-to-haves. Just working software.*