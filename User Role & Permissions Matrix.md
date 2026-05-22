# InkingiPro - User Role & Permissions Matrix
## "Who Can Do What" Document

**Version:** 1.0
**Status:** Final
**Date:** May 22, 2026
**Auth Framework:** Better Auth (replaces JWT)

---

## Document Purpose

This document maps every user role in InkingiPro to their exact access permissions across all system features. Developers use this as the single source of truth for:

- Backend API route protection (Express middleware)
- Frontend UI visibility (React Native + React Web)
- Better Auth permission configuration

---

## Role Definitions

| Role | Description | Better Auth Role Type |
|------|-------------|----------------------|
| **Client** | Diaspora investor / property owner who funds projects | Organization Owner |
| **Lead Engineer** | IER-licensed professional who manages construction | Organization Admin |
| **Site Supervisor** | Independent quality inspector | Organization Member |
| **Material Supplier** | Verified vendor providing materials | Organization Member + API Key |
| **Platform Admin** | InkingiPro operations team | Super Admin (Global) |

---

## Permission Legend

| Symbol | Action | Better Auth Permission Check |
|--------|--------|------------------------------|
| ✅ | **Create** - Can add new records | `hasPermission("create")` |
| 👁️ | **Read** - Can view existing records | `hasPermission("read")` |
| ✏️ | **Update** - Can modify existing records | `hasPermission("update")` |
| ❌ | **Delete** - Can remove records | `hasPermission("delete")` |
| 🚫 | **No Access** - Cannot perform action | No permission granted |
| 🔒 | **Conditional** - Depends on status/ownership | Additional logic required |

---

## MATRIX 1: Authentication & Account Management

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Register new account | ✅ | ✅ | ✅ | ✅ | ✅ |
| Login to account | ✅ | ✅ | ✅ | ✅ | ✅ |
| Verify email with OTP | ✅ | ✅ | ✅ | ✅ | ✅ |
| Verify phone with SMS OTP | ✅ | ✅ | ✅ | ✅ | ✅ |
| Request password reset | ✅ | ✅ | ✅ | ✅ | ✅ |
| Change own password | ✏️ | ✏️ | ✏️ | ✏️ | ✏️ |
| View own profile | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Edit own profile | ✏️ | ✏️ | ✏️ | ✏️ | ✏️ |
| Delete own account | ❌ | ❌ | ❌ | ❌ | ✅ |
| View all users | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| Edit any user profile | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Delete any user account | 🚫 | 🚫 | 🚫 | 🚫 | ❌ |
| Impersonate another user | 🚫 | 🚫 | 🚫 | 🚫 | ✅ |

---

## MATRIX 2: KYC (Know Your Customer) Verification

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Upload national ID document | ✅ | ✅ | ✅ | ✅ | 🚫 |
| Upload passport photo | ✅ | ✅ | ✅ | ✅ | 🚫 |
| Upload IER professional license | 🚫 | ✅ | 🚫 | 🚫 | 🚫 |
| Upload professional indemnity insurance | 🚫 | ✅ | ✅ | 🚫 | 🚫 |
| Upload business registration certificate | 🚫 | 🚫 | 🚫 | ✅ | 🚫 |
| Upload tax compliance certificate | 🚫 | 🚫 | 🚫 | ✅ | 🚫 |
| View own KYC status | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| View all pending KYC submissions | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| Review KYC documents | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Approve KYC application | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Reject KYC application with reason | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Request additional KYC documents | 🚫 | 🚫 | 🚫 | 🚫 | ✅ |
| View KYC audit history | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |

---

## MATRIX 3: Project Management

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Create new project | ✅ | 🚫 | 🚫 | 🚫 | ✅ |
| Set project budget (RWF) | ✅ | 🚫 | 🚫 | 🚫 | ✏️ |
| Define GPS boundary polygon | ✅ | 🚫 | 🚫 | 🚫 | ✏️ |
| Upload site photos to Cloudinary | ✅ | ✅ | 🚫 | 🚫 | ✅ |
| Upload architectural plans (PDF) | ✅ | ✅ | 🚫 | 🚫 | ✅ |
| View own projects list | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| View all projects (any user) | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| View project details | 👁️ | 👁️ | 🔒 | 🔒 | 👁️ |
| Edit project name/description | ✏️ | 🚫 | 🚫 | 🚫 | ✏️ |
| Edit project budget | 🔒 | 🚫 | 🚫 | 🚫 | ✏️ |
| Delete project (soft delete) | ✏️ | 🚫 | 🚫 | 🚫 | ❌ |
| Archive project permanently | 🚫 | 🚫 | 🚫 | 🚫 | ❌ |
| Restore archived project | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |

*🔒 Supervisor sees only projects they are assigned to inspect*
*🔒 Supplier sees only projects they have RFQs or deliveries for*

---

## MATRIX 4: Milestones & Bill of Quantities (BoQ)

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Create milestone | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| Edit milestone description | 🚫 | ✏️ | 🚫 | 🚫 | ✏️ |
| Set milestone budget percentage | 🚫 | ✅ | 🚫 | 🚫 | ✏️ |
| Define milestone acceptance criteria | 🚫 | ✅ | 🚫 | 🚫 | ✏️ |
| Set milestone dependencies | 🚫 | ✅ | 🚫 | 🚫 | ✏️ |
| Delete milestone | 🚫 | 🔒 | 🚫 | 🚫 | ❌ |
| View all project milestones | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Approve milestone (activate) | ✏️ | 🚫 | 🚫 | 🚫 | ✏️ |
| Request milestone revision | ✏️ | ✏️ | 🚫 | 🚫 | ✏️ |
| Create BoQ items | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| Edit BoQ item price/quantity | 🚫 | ✏️ | 🚫 | 🚫 | ✏️ |
| Delete BoQ item | 🚫 | 🔒 | 🚫 | 🚫 | ❌ |
| View BoQ with market price comparison | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| Export BoQ to PDF | 👁️ | 👁️ | 🚫 | 🚫 | 👁️ |
| Export BoQ to Excel | 👁️ | 👁️ | 🚫 | 🚫 | 👁️ |

*🔒 Engineer cannot delete milestone after client approval*

---

## MATRIX 5: Escrow & Payments

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Deposit funds via MTN MoMo | ✅ | 🚫 | 🚫 | 🚫 | ✅ |
| Deposit funds via Airtel Money | ✅ | 🚫 | 🚫 | 🚫 | ✅ |
| Deposit funds via bank transfer | ✅ | 🚫 | 🚫 | 🚫 | ✅ |
| View escrow balance | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| View transaction history | 👁️ | 👁️ | 👁️ | 🔒 | 👁️ |
| Request milestone payment | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| Submit inspection report | 🚫 | 🚫 | ✅ | 🚫 | 🚫 |
| Approve milestone payment | ✏️ | 🚫 | 🚫 | 🚫 | ✏️ |
| Release payment from escrow | 🔒 | 🚫 | 🚫 | 🚫 | ✏️ |
| Freeze escrow funds | 🔒 | 🚫 | 🚫 | 🚫 | ✏️ |
| Reverse payment transaction | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| View reconciliation report | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| Receive delivery auto-payment | 🚫 | 🚫 | 🚫 | 👁️ | ✏️ |

*🔒 Supplier sees only delivery payments for their own materials*
*🔒 Client can release payment only after supervisor approval*

---

## MATRIX 6: Supply Chain (RFQ, Quotes, Delivery)

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Create RFQ broadcast | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| Set RFQ deadline | 🚫 | ✅ | 🚫 | 🚫 | ✏️ |
| Define material specifications (JSONB) | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| Cancel RFQ | 🚫 | 🔒 | 🚫 | 🚫 | ✏️ |
| View RFQ details | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Receive RFQ notification | 🚫 | 🚫 | 🚫 | 👁️ | 👁️ |
| Submit quote with price | 🚫 | 🚫 | 🚫 | ✅ | ✅ |
| Edit submitted quote | 🚫 | 🚫 | 🚫 | 🔒 | ✏️ |
| View all quotes for RFQ | 🚫 | 👁️ | 🚫 | 🔒 | 👁️ |
| Compare quotes (ranked) | 🚫 | 👁️ | 🚫 | 🚫 | 👁️ |
| Select winning quote | 🚫 | ✏️ | 🚫 | 🚫 | ✏️ |
| Generate purchase order (PDF) | 👁️ | ✅ | 🚫 | 👁️ | ✅ |
| Start delivery tracking (GPS) | 🚫 | 🚫 | 🚫 | ✅ | 🚫 |
| Capture delivery proof photos | 🚫 | 🚫 | 🚫 | ✅ | ✅ |
| Validate GPS at project site | 🚫 | 🚫 | 🚫 | ✅ | ✏️ |
| Confirm delivery receipt | 🚫 | ✅ | ✅ | 🚫 | ✅ |
| Request delivery revision | ✏️ | ✏️ | ✏️ | 🚫 | ✏️ |
| Rate supplier (1-5 stars) | 🚫 | ✏️ | ✏️ | 🚫 | 🚫 |

*🔒 Engineer cannot cancel RFQ after quotes received*
*🔒 Supplier cannot edit quote after selection*
*🔒 Supplier sees only their own quotes*

---

## MATRIX 7: Inspection & Quality Control

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Upload daily progress photos | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| Upload progress video (max 2 min) | 🚫 | ✅ | 🚫 | 🚫 | ✅ |
| View progress timeline | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| Complete inspection checklist | 🚫 | 🚫 | ✅ | 🚫 | 🚫 |
| Assign quality rating (1-5) | 🚫 | 🚫 | ✏️ | 🚫 | ✏️ |
| Capture digital signature | 🚫 | 🚫 | ✅ | 🚫 | 🚫 |
| Upload inspection photos | 🚫 | 🚫 | ✅ | 🚫 | ✅ |
| Submit inspection report | 🚫 | 🚫 | ✅ | 🚫 | 🚫 |
| Request re-inspection | ✏️ | ✏️ | 🚫 | 🚫 | ✏️ |
| View all inspection history | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| Download inspection package (ZIP) | ✅ | ✅ | ✅ | 🚫 | ✅ |
| Export compliance report | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |

---

## MATRIX 8: Dispute Resolution

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Initiate dispute | ✅ | ✅ | ✅ | ✅ | ✅ |
| Select dispute category | ✅ | ✅ | ✅ | ✅ | ✅ |
| Upload evidence documents | ✅ | ✅ | ✅ | ✅ | ✅ |
| View dispute details | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Add comments to dispute | ✅ | ✅ | ✅ | ✅ | ✅ |
| Request additional evidence | 🚫 | 🚫 | 🚫 | 🚫 | ✅ |
| Mediate dispute | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Issue resolution decision | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Release full payment | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Release partial payment | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Apply penalty to payment | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Terminate project due to dispute | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| File appeal against decision | ✏️ | ✏️ | ✏️ | ✏️ | ✏️ |
| View dispute analytics | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |

---

## MATRIX 9: Notifications & Communication

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Receive push notifications | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Receive email notifications | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Receive SMS notifications | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Configure notification preferences | ✏️ | ✏️ | ✏️ | ✏️ | ✏️ |
| Opt out of marketing emails | ✏️ | ✏️ | ✏️ | ✏️ | ✏️ |
| Send in-app message | ✅ | ✅ | ✅ | ✅ | ✅ |
| Send message with photo attachment | ✅ | ✅ | ✅ | ✅ | ✅ |
| Reply to message thread | ✅ | ✅ | ✅ | ✅ | ✅ |
| View message history | 👁️ | 👁️ | 👁️ | 👁️ | 👁️ |
| Delete own message | ❌ | ❌ | ❌ | ❌ | ❌ |
| Delete any message | 🚫 | 🚫 | 🚫 | 🚫 | ❌ |
| Broadcast global announcement | 🚫 | 🚫 | 🚫 | 🚫 | ✅ |

---

## MATRIX 10: Reporting & Analytics

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| View project progress Gantt chart | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| View budget variance report | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| View milestone completion report | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| View financial transaction report | 👁️ | 👁️ | 🚫 | 🔒 | 👁️ |
| View supplier performance report | 👁️ | 👁️ | 🚫 | 🔒 | 👁️ |
| View KYC compliance report | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| View dispute statistics | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| View system audit log | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| Export report as PDF | ✅ | ✅ | 🚫 | ✅ | ✅ |
| Export report as Excel | ✅ | ✅ | 🚫 | ✅ | ✅ |
| Schedule automated reports | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| View dashboard analytics | 👁️ | 👁️ | 👁️ | 🔒 | 👁️ |

*🔒 Supplier sees only their own payment and rating reports*

---

## MATRIX 11: Supervisor Assignment

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| View available supervisors list | 👁️ | 👁️ | 🚫 | 🚫 | 👁️ |
| Send supervisor invitation | ✅ | 🚫 | 🚫 | 🚫 | ✅ |
| Accept supervisor assignment | 🚫 | 🚫 | ✏️ | 🚫 | ✏️ |
| Decline supervisor assignment | 🚫 | 🚫 | ✏️ | 🚫 | ✏️ |
| View all project assignments | 👁️ | 👁️ | 👁️ | 🚫 | 👁️ |
| Remove supervisor from project | ✏️ | 🚫 | 🚫 | 🚫 | ✏️ |
| Request supervisor change | ✏️ | ✏️ | 🚫 | 🚫 | ✏️ |
| View supervisor performance history | 👁️ | 👁️ | 🔒 | 🚫 | 👁️ |

*🔒 Supervisor sees only their own performance history*

---

## MATRIX 12: System Configuration (Admin Only)

| Feature | Client | Engineer | Supervisor | Supplier | Admin |
|---------|:------:|:--------:|:----------:|:--------:|:-----:|
| Configure escrow release rules | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Set milestone auto-approval timeout | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Configure GPS verification radius | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Set dispute response SLA | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Manage integration API keys | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Configure email templates | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| Configure SMS templates | 🚫 | 🚫 | 🚫 | 🚫 | ✏️ |
| View system health metrics | 🚫 | 🚫 | 🚫 | 🚫 | 👁️ |
| Trigger manual backups | 🚫 | 🚫 | 🚫 | 🚫 | ✅ |
| Run data migration scripts | 🚫 | 🚫 | 🚫 | 🚫 | ✅ |

---

## Summary: Permission Count by Role

| Permission Type | Client | Engineer | Supervisor | Supplier | Admin |
|-----------------|--------|----------|------------|----------|-------|
| **Create (✅)** | 12 | 18 | 6 | 9 | 38 |
| **Read (👁️)** | 38 | 39 | 34 | 28 | 47 |
| **Update (✏️)** | 12 | 14 | 6 | 4 | 34 |
| **Delete (❌)** | 0 | 0 | 0 | 0 | 3 |
| **Conditional (🔒)** | 3 | 3 | 1 | 5 | 0 |
| **No Access (🚫)** | 27 | 18 | 45 | 46 | 0 |

---

## Rule of Thumb for Developers

1. **Client** = Owner of project → Full control over money and approvals
2. **Engineer** = Project manager → Can create milestones, request payments, manage RFQs
3. **Supervisor** = Quality checker → Can only inspect, never approve money
4. **Supplier** = External vendor → Can only see RFQs, submit quotes, confirm deliveries
5. **Admin** = God mode → Can do everything except delete critical financial records (audit requirement)

---

## How to Use This Matrix

### For Backend (Express + Better Auth)

```typescript
// Example: Protect milestone approval endpoint
app.post("/api/milestones/:id/approve", 
  requireAuth, 
  requirePermission("milestone.approve"),
  milestoneApprovalHandler
);
```

### For Frontend (React Native / React Web)

```javascript
// Example: Hide approve button for non-clients
{user.role === "client" && (
  <Button onPress={approveMilestone}>Approve Payment</Button>
)}
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | May 22, 2026 | Initial release with 12 permission matrices |

---

*This document is the authoritative source for RBAC in InkingiPro. Any deviation requires approved change request.*