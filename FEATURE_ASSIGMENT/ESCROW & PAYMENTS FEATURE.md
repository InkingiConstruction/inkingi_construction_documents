# INKINGIPRO - ESCROW & PAYMENTS FEATURE DOCUMENTATION

**Feature:** Escrow & Payments (Deposit, Balance, Request Payment, Approve Payment, Release Payment, Transaction History)
**Feature IDs:** ESCROW-01, ESCROW-04, ESCROW-05, ESCROW-06, ESCROW-07, ESCROW-09 from MVP list
**Status:** Ready for Development
**Estimated Time:** 18-22 Hours
**Database Tables Used:** escrow_accounts, transactions, projects, milestones, users

---

## TABLE OF CONTENTS

1. Feature Overview
2. Database Tables Used (Names Only)
3. Complete API Endpoints
4. Frontend Screens Required
5. Business Rules & Validation
6. Testing Checklist with Checkboxes
7. Common Problems & Solutions
8. Time Tracking & Task Breakdown
9. Prerequisites (What Must Be Done First)

---

## 1. FEATURE OVERVIEW

### 1.1 What This Feature Does

This feature handles all money movement in the platform. It is the core trust mechanism of InkingiPro.

**PART A: Deposit Funds via MTN Mobile Money (ESCROW-01)**
- Client selects MTN Mobile Money as payment method
- Client enters amount to deposit (minimum 100,000 RWF)
- System calls MTN MoMo API to request payment
- MTN sends webhook confirmation when payment is complete
- System updates escrow balance and creates transaction record
- Client receives email receipt

**PART B: View Escrow Balance (ESCROW-04)**
- Client views available balance in escrow
- Client sees locked balance (funds in dispute)
- Client sees total deposited and total released
- Engineer sees balance for projects they work on

**PART C: Request Milestone Payment (ESCROW-05)**
- Engineer selects active milestone that is not yet paid
- Engineer uploads completion photos to Cloudinary
- Engineer adds completion notes
- System creates payment request and notifies supervisor

**PART D: Approve Milestone Payment (ESCROW-06)**
- Client receives notification of payment request
- Client reviews completion photos and inspection report
- Client clicks Approve button
- System marks milestone as ready for release

**PART E: Release Payment from Escrow (ESCROW-07)**
- After client approves, system releases funds
- Escrow balance decreases by milestone amount
- Transaction record created with type "release"
- Engineer receives payment notification
- Milestone status changes to "paid"

**PART F: View Transaction History (ESCROW-09)**
- User views all transactions on a project
- Shows deposits, releases, refunds
- Each transaction shows amount, date, type, status
- Filter by date range and transaction type

### 1.2 Why This Feature is MVP Priority

| Reason | Explanation |
|--------|-------------|
| Core Value Proposition | Escrow is what makes InkingiPro different from direct payments |
| Trust | Money held until work is verified builds diaspora investor confidence |
| Payment Flow | Without this, engineer never gets paid, client has no control |
| Audit Trail | Transaction history provides proof of all money movements |

### 1.3 Sequential Flow Steps

**Deposit Flow:**
- Step 1: Client opens project details and taps "Deposit Funds"
- Step 2: Client enters amount (minimum 100,000 RWF)
- Step 3: Client selects "MTN Mobile Money" as payment method
- Step 4: Client enters MTN phone number
- Step 5: System calls MTN API to create payment request
- Step 6: MTN sends USSD push to client's phone
- Step 7: Client enters PIN on phone to confirm
- Step 8: MTN sends webhook to InkingiPro backend
- Step 9: System verifies webhook signature
- Step 10: System updates escrow_accounts.balance += amount
- Step 11: System creates transaction record (type: deposit)
- Step 12: System sends email receipt to client
- Step 13: System sends push notification to client "Deposit successful"

**Request Payment Flow:**
- Step 1: Engineer navigates to active milestone
- Step 2: Engineer taps "Request Payment"
- Step 3: Engineer uploads minimum 5 completion photos
- Step 4: Engineer adds completion notes
- Step 5: System checks milestone has active status
- Step 6: System checks milestone not already paid
- Step 7: System creates payment request record
- Step 8: System sends notification to supervisor "Inspection required"
- Step 9: Milestone status changes to "pending_supervisor"

**Approve and Release Payment Flow:**
- Step 1: Client receives notification "Milestone ready for approval"
- Step 2: Client opens milestone review screen
- Step 3: Client views engineer completion photos
- Step 4: Client views supervisor inspection report (if completed)
- Step 5: Client taps "Approve Payment"
- Step 6: System checks escrow has sufficient balance
- Step 7: System updates escrow_accounts.balance -= milestone_amount
- Step 8: System creates transaction record (type: release)
- Step 9: System sends email receipt to engineer
- Step 10: System sends push notification to engineer "Payment released"
- Step 11: Milestone status changes to "paid"

---

## 2. DATABASE TABLES USED (Names Only)

| Table Name | Purpose In This Feature |
|------------|-------------------------|
| escrow_accounts | Store current balance and locked balance per project |
| transactions | Record every deposit, release, refund with audit trail |
| projects | Get project budget and verify client ownership |
| milestones | Get milestone budget amount and verify status |
| users | Verify user roles and send notifications |

**Tables written to:** escrow_accounts (balance update), transactions (insert), milestones (status update)

**Tables accessed (read only):** projects, users

---

## 3. COMPLETE API ENDPOINTS

### 3.1 Escrow Endpoints (8 Endpoints)

| Method | Endpoint | What It Does | Request Body | Response |
|--------|----------|--------------|--------------|----------|
| POST | /api/v1/escrow/deposit/mtn | Initiate MTN deposit | amount, phone_number, project_id | transaction_id, payment_reference |
| POST | /api/v1/webhooks/mtn | MTN callback webhook | MTN payload | status received |
| GET | /api/v1/escrow/projects/:projectId/balance | Get escrow balance | None | balance, locked_balance, total_deposited, total_released |
| POST | /api/v1/milestones/:id/request-payment | Engineer requests payment | completion_notes, photo_urls | payment_request_id |
| GET | /api/v1/milestones/:id/payment-status | Get payment request status | None | status, requested_at, approved_at |
| POST | /api/v1/milestones/:id/approve-payment | Client approves payment | None | success, transaction_id |
| POST | /api/v1/milestones/:id/reject-payment | Client rejects payment | rejection_reason | success |
| GET | /api/v1/escrow/projects/:projectId/transactions | Get transaction history | page, limit, type_filter | transactions array |

### 3.2 Endpoint Details

**ENDPOINT 1: Initiate MTN Deposit**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/escrow/deposit/mtn |
| Auth Required | Yes (JWT token) |
| Role Required | Client (project owner) |

**Request Body:**
```json
{
    "project_id": "project-id-456",
    "amount": 500000,
    "phone_number": "+250788123456"
}
```

**Validation Rules:**
| Field | Rule |
|-------|------|
| project_id | Required, must exist, user must be owner |
| amount | Required, minimum 100,000 RWF, maximum 10,000,000 RWF per transaction |
| phone_number | Required, valid MTN Rwanda format (+250788XXXXXX or +250789XXXXXX) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Deposit initiated. Check your phone to complete payment.",
    "data": {
        "transaction_id": "txn-789",
        "payment_reference": "MTN-REF-12345",
        "amount": 500000,
        "status": "pending",
        "expires_in_seconds": 300
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| INSUFFICIENT_LIMIT | 400 | Amount below 100,000 RWF |
| PROJECT_NOT_FOUND | 404 | Project ID does not exist |
| NOT_PROJECT_OWNER | 403 | User is not the client who owns project |
| MTN_API_ERROR | 502 | MTN API is down or returned error |

---

**ENDPOINT 2: MTN Webhook (Callback)**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/webhooks/mtn |
| Auth Required | No (but signature verified) |
| Role Required | None (public endpoint) |

**Request Body (from MTN):**
```json
{
    "transaction_id": "MTN-REF-12345",
    "status": "SUCCESSFUL",
    "amount": 500000,
    "payer_phone": "+250788123456",
    "reference": "txn-789"
}
```

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Webhook processed"
}
```

**Processing Logic:**
- Verify webhook signature using MTN public key
- Find transaction by reference
- Update transaction status to "completed"
- Update escrow_accounts.balance += amount
- Insert transaction record with type "deposit"
- Send email receipt to client

---

**ENDPOINT 3: Get Escrow Balance**

| Property | Value |
|----------|-------|
| Method | GET |
| URL | /api/v1/escrow/projects/:projectId/balance |
| Auth Required | Yes (JWT token) |
| Role Required | Client (owner) OR Engineer (assigned) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "data": {
        "balance": 5250000,
        "locked_balance": 0,
        "total_deposited": 8000000,
        "total_released": 2750000,
        "currency": "RWF",
        "last_updated": "2024-01-15T10:30:00Z"
    }
}
```

---

**ENDPOINT 4: Request Milestone Payment**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/milestones/:id/request-payment |
| Auth Required | Yes (JWT token) |
| Role Required | Engineer (assigned to project) |

**Request Body:**
```json
{
    "completion_notes": "Foundation completed. All inspections passed. Concrete cured for 7 days.",
    "photo_urls": [
        "https://cloudinary.com/project123/photo1.jpg",
        "https://cloudinary.com/project123/photo2.jpg",
        "https://cloudinary.com/project123/photo3.jpg",
        "https://cloudinary.com/project123/photo4.jpg",
        "https://cloudinary.com/project123/photo5.jpg"
    ]
}
```

**Validation Rules:**
| Field | Rule |
|-------|------|
| milestone_id | Milestone must exist and belong to project |
| completion_notes | Required, min 20 characters, max 500 characters |
| photo_urls | Required, minimum 5 URLs, maximum 20 URLs |
| milestone status | Must be "active" (client approved) |
| milestone paid | Must not already be "paid" |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Payment request submitted. Waiting for supervisor inspection.",
    "data": {
        "milestone_id": "milestone-123",
        "status": "pending_supervisor",
        "requested_at": "2024-01-15T10:30:00Z",
        "amount_requested": 3750000
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| MILESTONE_NOT_ACTIVE | 400 | Milestone status is not "active" |
| MILESTONE_ALREADY_PAID | 400 | Milestone already has status "paid" |
| INSUFFICIENT_PHOTOS | 400 | Less than 5 photos provided |
| NOTES_TOO_SHORT | 400 | Completion notes less than 20 characters |

---

**ENDPOINT 5: Approve Milestone Payment**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/milestones/:id/approve-payment |
| Auth Required | Yes (JWT token) |
| Role Required | Client (project owner) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Payment approved and released from escrow",
    "data": {
        "milestone_id": "milestone-123",
        "amount_released": 3750000,
        "new_escrow_balance": 1500000,
        "transaction_id": "txn-890",
        "released_at": "2024-01-15T11:00:00Z"
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| INSUFFICIENT_ESCROW | 400 | Escrow balance less than milestone amount |
| MILESTONE_NOT_REQUESTED | 400 | Payment request not yet submitted |
| INSPECTION_REQUIRED | 400 | Supervisor inspection not yet completed |

---

**ENDPOINT 6: Get Transaction History**

| Property | Value |
|----------|-------|
| Method | GET |
| URL | /api/v1/escrow/projects/:projectId/transactions |
| Auth Required | Yes (JWT token) |
| Role Required | Client (owner) OR Engineer (assigned) |

**Query Parameters:**
| Parameter | Values | Default |
|-----------|--------|---------|
| type | deposit, release, refund, all | all |
| page | number | 1 |
| limit | number | 20 |
| from_date | ISO date | 30 days ago |
| to_date | ISO date | today |

**Success Response (200 OK):**
```json
{
    "success": true,
    "data": {
        "transactions": [
            {
                "id": "txn-001",
                "type": "deposit",
                "amount": 5000000,
                "status": "completed",
                "description": "Initial deposit via MTN MoMo",
                "created_at": "2024-01-10T09:00:00Z"
            },
            {
                "id": "txn-002",
                "type": "release",
                "amount": 1250000,
                "status": "completed",
                "description": "Payment for milestone: Site Preparation",
                "milestone_title": "Site Preparation",
                "created_at": "2024-01-12T14:30:00Z"
            }
        ],
        "summary": {
            "total_deposits": 5000000,
            "total_releases": 1250000,
            "net_balance": 3750000
        },
        "pagination": {
            "page": 1,
            "limit": 20,
            "total": 2,
            "pages": 1
        }
    }
}
```

---

## 4. FRONTEND SCREENS REQUIRED

### 4.1 React Native Mobile App Screens (5 Screens)

| Screen Name | File Location | What User Does On This Screen |
|-------------|---------------|-------------------------------|
| DepositScreen | screens/escrow/DepositScreen.tsx | Client enters amount, selects MTN, enters phone number |
| BalanceCard | components/escrow/BalanceCard.tsx | Shows current escrow balance (embedded in project detail) |
| RequestPaymentScreen | screens/milestone/RequestPaymentScreen.tsx | Engineer uploads photos, adds notes, submits request |
| ApprovePaymentScreen | screens/milestone/ApprovePaymentScreen.tsx | Client reviews photos, sees amount, approves or rejects |
| TransactionHistoryScreen | screens/escrow/TransactionHistoryScreen.tsx | Views all transactions with filters |

### 4.2 Screen Details

**Screen 1: DepositScreen**

| Element | Description |
|---------|-------------|
| Header | "Deposit Funds" + back button |
| Project Name | Shows current project name |
| Amount Input | Number input with RWF formatting, minimum 100,000 |
| Quick Amount Buttons | 100k, 500k, 1M, 5M |
| Payment Method Card | MTN Mobile Money logo and description |
| Phone Number Input | Pre-filled with client's phone, editable |
| Fee Info | "No deposit fees. Standard MTN charges apply." |
| Deposit Button | Large green button, disabled if amount invalid |
| Confirmation Dialog | Shows amount, phone, "Check your phone to complete payment" |

**Screen 2: RequestPaymentScreen (Engineer)**

| Element | Description |
|---------|-------------|
| Header | "Request Payment - [Milestone Name]" |
| Milestone Info | Shows milestone amount (calculated from percentage) |
| Photo Upload Section | 5 photo slots, each with camera/gallery picker |
| Photo Preview | Thumbnails of selected photos, can retake |
| Completion Notes | Text area, placeholder "Describe what was completed..." |
| Character Counter | Shows min 20 / max 500 |
| Submit Button | Disabled until 5 photos and 20+ characters |
| Warning | "Fake or incorrect photos will result in payment rejection" |

**Screen 3: ApprovePaymentScreen (Client)**

| Element | Description |
|---------|-------------|
| Header | "Review Payment Request" |
| Milestone Info | Milestone name, requested amount in RWF |
| Engineer Photos | Grid of 5+ photos, tap to enlarge |
| Completion Notes | Read-only display of engineer's notes |
| Inspection Report | Shows supervisor rating and checklist (if completed) |
| Action Buttons | Approve (green), Reject (red), Need More Info (yellow) |
| Reject Dialog | Text input for rejection reason (required) |
| Warning | "Once approved, funds are released and cannot be reversed" |

**Screen 4: TransactionHistoryScreen**

| Element | Description |
|---------|-------------|
| Header | "Transaction History" |
| Summary Cards | Total Deposits, Total Released, Current Balance |
| Filter Tabs | All, Deposits, Releases |
| Transaction List | Each row shows date, type, amount, status |
| Deposit Row | Green arrow down, "+5,000,000 RWF" |
| Release Row | Red arrow up, "-1,250,000 RWF" |
| Detail Modal | Tap transaction to see full details and receipt |

---

## 5. BUSINESS RULES & VALIDATION

### 5.1 Deposit Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| ESC-01 | Only client who owns project can deposit | 403 Forbidden |
| ESC-02 | Minimum deposit is 100,000 RWF | 400 error |
| ESC-03 | Maximum deposit per transaction is 10,000,000 RWF | 400 error |
| ESC-04 | Phone number must be valid MTN Rwanda number | 400 error |
| ESC-05 | MTN API called with timeout of 30 seconds | Retry or timeout error |
| ESC-06 | Webhook signature must be verified | Reject invalid webhook |
| ESC-07 | Deposit email receipt sent within 60 seconds | Bull job queued |
| ESC-08 | Transaction record created before MTN call | For audit trail |
| ESC-09 | Escrow balance updated only after webhook confirmation | Not before |

### 5.2 Payment Request Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| ESC-10 | Only engineer assigned to project can request | 403 Forbidden |
| ESC-11 | Milestone must have status "active" | 400 MILESTONE_NOT_ACTIVE |
| ESC-12 | Milestone must not be already paid | 400 MILESTONE_ALREADY_PAID |
| ESC-13 | Minimum 5 completion photos required | 400 INSUFFICIENT_PHOTOS |
| ESC-14 | Completion notes minimum 20 characters | 400 NOTES_TOO_SHORT |
| ESC-15 | Cannot request same milestone twice | 400 ALREADY_REQUESTED |
| ESC-16 | Supervisor notified within 60 seconds | Bull job |

### 5.3 Payment Approval Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| ESC-17 | Only client who owns project can approve | 403 Forbidden |
| ESC-18 | Escrow balance must be >= milestone amount | 400 INSUFFICIENT_ESCROW |
| ESC-19 | Milestone must have payment request submitted | 400 NOT_REQUESTED |
| ESC-20 | Supervisor inspection must be approved | 400 INSPECTION_REQUIRED |
| ESC-21 | After approval, funds released immediately | Transaction created |
| ESC-22 | Engineer receipt email sent within 60 seconds | Bull job |
| ESC-23 | Milestone status changes to "paid" | Database updated |
| ESC-24 | Cannot reverse approval | No undo button |

### 5.4 Transaction Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| ESC-25 | Every deposit creates transaction record | Automatic |
| ESC-26 | Every release creates transaction record | Automatic |
| ESC-27 | Transaction records are append-only | No updates or deletes |
| ESC-28 | Transaction history shows last 30 days by default | Adjustable filter |
| ESC-29 | Each transaction has unique ID for audit | UUID format |

---

## 6. TESTING CHECKLIST WITH CHECKBOXES

### 6.1 Deposit Tests (10 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 1 | Client deposits valid amount via MTN | MTN API called, webhook received, balance updated | ☐ |
| 2 | Deposit below minimum (50,000 RWF) | 400 INSUFFICIENT_LIMIT | ☐ |
| 3 | Deposit with wrong phone format | 400 INVALID_PHONE | ☐ |
| 4 | Engineer tries to deposit | 403 Forbidden | ☐ |
| 5 | Deposit to non-existent project | 404 PROJECT_NOT_FOUND | ☐ |
| 6 | MTN API timeout | 502 error, transaction marked failed | ☐ |
| 7 | MTN webhook with invalid signature | Webhook rejected, balance not updated | ☐ |
| 8 | Duplicate webhook received | Second webhook ignored | ☐ |
| 9 | Deposit email receipt received | Check email inbox | ☐ |
| 10 | Transaction record created | Check transactions table | ☐ |

### 6.2 View Balance Tests (4 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 11 | Client views escrow balance | Returns correct balance from escrow_accounts | ☐ |
| 12 | Engineer views assigned project balance | Returns correct balance | ☐ |
| 13 | Supervisor tries to view balance | 403 Forbidden (if not allowed) | ☐ |
| 14 | Balance updates immediately after deposit | Refresh shows new balance | ☐ |

### 6.3 Request Payment Tests (8 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 15 | Engineer requests payment with 5 photos | 200 OK, status pending_supervisor | ☐ |
| 16 | Request with only 4 photos | 400 INSUFFICIENT_PHOTOS | ☐ |
| 17 | Request with notes too short | 400 NOTES_TOO_SHORT | ☐ |
| 18 | Request on inactive milestone | 400 MILESTONE_NOT_ACTIVE | ☐ |
| 19 | Request on already paid milestone | 400 MILESTONE_ALREADY_PAID | ☐ |
| 20 | Client tries to request payment | 403 Forbidden | ☐ |
| 21 | Supervisor notification sent | Check notification table | ☐ |
| 22 | Request amount matches milestone budget | Verify against milestone budget_percentage | ☐ |

### 6.4 Approve Payment Tests (8 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 23 | Client approves valid payment request | 200 OK, funds released, balance decreased | ☐ |
| 24 | Approve when escrow insufficient | 400 INSUFFICIENT_ESCROW | ☐ |
| 25 | Approve without payment request | 400 NOT_REQUESTED | ☐ |
| 26 | Approve without supervisor approval | 400 INSPECTION_REQUIRED | ☐ |
| 27 | Engineer tries to approve | 403 Forbidden | ☐ |
| 28 | After approval, milestone status paid | Check milestones table | ☐ |
| 29 | Engineer receives email receipt | Check email inbox | ☐ |
| 30 | Transaction record created for release | Check transactions table | ☐ |

### 6.5 Transaction History Tests (5 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 31 | View transaction history | Returns all transactions for project | ☐ |
| 32 | Filter by deposit type | Returns only deposit transactions | ☐ |
| 33 | Filter by release type | Returns only release transactions | ☐ |
| 34 | Pagination works | Returns correct page of results | ☐ |
| 35 | Summary totals match balance | Total deposits - total releases = current balance | ☐ |

---

## 7. COMMON PROBLEMS & SOLUTIONS

### Problem 1: MTN Webhook Not Received

**Solution:** Implement polling fallback. If webhook not received after 2 minutes, query MTN API for transaction status. Also add manual admin button to mark transaction as completed for testing.

**How to verify:** Simulate webhook failure. System should query MTN status API after 120 seconds.

### Problem 2: Engineer Requests Payment Before Supervisor Assigned

**Solution:** Require supervisor assignment before allowing payment request. At least one supervisor must be assigned to project and have status "active".

**How to verify:** Try to request payment with no supervisor assigned. Get error "No supervisor assigned to this project"

### Problem 3: Client Approves Payment But Escrow Balance Shows Negative

**Solution:** Always check escrow balance BEFORE allowing approval. Use database row-level lock to prevent race conditions. Transaction should fail if balance insufficient.

**How to verify:** Send two concurrent approval requests. Only one succeeds.

### Problem 4: Duplicate Webhook Processing

**Solution:** Use idempotency key. MTN sends unique reference ID. Store processed reference IDs in Redis. If same reference received twice, ignore second.

**How to verify:** Send same webhook twice. First updates balance, second ignored.

### Problem 5: Currency Display Confusion (RWF vs USD)

**Solution:** Store everything in RWF. On client app, show RWF as primary. Optionally show USD equivalent using exchange rate from config. Update rate daily.

**How to verify:** Client sees "1,000,000 RWF (approx $770 USD)"

---

## 8. TIME TRACKING & TASK BREAKDOWN

### 8.1 Developer Tasks (Single Developer)

| Task # | Task Description | Hours | Completed? | Actual Hours |
|--------|------------------|-------|------------|--------------|
| 1 | Create escrow_accounts table migration | 0.5 | ☐ | |
| 2 | Create transactions table migration | 0.5 | ☐ | |
| 3 | Implement POST /escrow/deposit/mtn endpoint | 2 | ☐ | |
| 4 | Implement MTN API integration with axios | 1.5 | ☐ | |
| 5 | Implement MTN webhook handler | 1.5 | ☐ | |
| 6 | Implement GET /escrow/balance endpoint | 1 | ☐ | |
| 7 | Implement POST /milestones/:id/request-payment | 2 | ☐ | |
| 8 | Implement photo validation logic | 1 | ☐ | |
| 9 | Implement POST /milestones/:id/approve-payment | 2 | ☐ | |
| 10 | Implement escrow balance check and update | 1.5 | ☐ | |
| 11 | Implement GET /transactions history endpoint | 1.5 | ☐ | |
| 12 | Create DepositScreen (React Native) | 2 | ☐ | |
| 13 | Create BalanceCard component | 1 | ☐ | |
| 14 | Create RequestPaymentScreen (React Native) | 2 | ☐ | |
| 15 | Create ApprovePaymentScreen (React Native) | 2 | ☐ | |
| 16 | Create TransactionHistoryScreen (React Native) | 1.5 | ☐ | |
| 17 | Write API integration for all screens | 2 | ☐ | |
| 18 | Run all 35 test cases | 2 | ☐ | |
| 19 | Fix bugs found during testing | 2 | ☐ | |
| **Total** | | **28** | | |

### 8.2 Timeline (Single Developer)

| Day | Tasks | Hours |
|-----|-------|-------|
| Day 1 AM | Database + deposit endpoint (Tasks 1-4) | 4.5 |
| Day 1 PM | Webhook + balance endpoint (Tasks 5-6) | 2.5 |
| Day 2 AM | Request payment endpoint (Tasks 7-8) | 3 |
| Day 2 PM | Approve payment endpoint (Tasks 9-10) | 3.5 |
| Day 3 AM | Transaction history + DepositScreen (Tasks 11-12) | 3.5 |
| Day 3 PM | RequestPaymentScreen + BalanceCard (Tasks 13-14) | 3 |
| Day 4 AM | ApprovePaymentScreen + TransactionHistoryScreen (Tasks 15-16) | 3.5 |
| Day 4 PM | Integration + Testing + Bug fixes (Tasks 17-19) | 6 |

### 8.3 Feature Completion Checklist

| # | Item | Status |
|---|------|--------|
| 1 | All 8 API endpoints working | ☐ |
| 2 | All 5 mobile screens built | ☐ |
| 3 | All 35 test cases passed | ☐ |
| 4 | MTN deposit works end-to-end with sandbox | ☐ |
| 5 | Webhook updates balance correctly | ☐ |
| 6 | Engineer can request payment with 5+ photos | ☐ |
| 7 | Client can approve and funds release | ☐ |
| 8 | Escrow balance never goes negative | ☐ |
| 9 | Transaction history shows all movements | ☐ |
| 10 | Email receipts sent for deposit and release | ☐ |

---

## 9. PREREQUISITES (What Must Be Done First)

Before starting this feature, the following MUST be complete:

| Prerequisite | Status | Notes |
|--------------|--------|-------|
| Auth & KYC Feature | ☐ Pending | User must be able to login |
| Project Management Feature | ☐ Pending | Project must exist |
| Milestone Management Feature | ☐ Pending | Milestones must exist before requesting payment |
| Engineer assigned to project | ☐ Pending | Engineer must be assigned |
| Milestones approved by client | ☐ Pending | Milestones must be active |
| MTN MoMo sandbox credentials | ☐ Pending | API keys from MTN |

**Do NOT start this feature until Milestone Management is complete and tested.**

---

**END OF ESCROW & PAYMENTS FEATURE DOCUMENTATION**

**Next Feature Ready When You Are:** Inspection & Progress Photos (INSP-01, INSP-05, INSP-08, INSP-09)