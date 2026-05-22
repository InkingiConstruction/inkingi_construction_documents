# InkingiPro - Coding & Commenting Standards

**Version:** 1.0
**Status:** Final
**Date:** May 22, 2026
**Applies To:** React Native Mobile App + React Web Admin Portal + Node.js Backend API

---

## 🛡️ PART 1: GLOBAL CODING CORE (DRY, KISS, SOLID)

These three principles are **MANDATORY** for every line of code written in this project. No exceptions.

---

### 📌 DRY (Don't Repeat Yourself)

**Rule:** No duplicated business logic anywhere in the codebase.

**Violation Example (DO NOT DO THIS):**
```javascript
// BAD: Same logic repeated in two places

// In auth controller
function calculateOTPExpiry() {
  const now = new Date();
  const expiry = new Date(now.getTime() + 5 * 60000);
  return expiry;
}

// In payment controller
function getOTPExpiryTime() {
  const now = new Date();
  const expiry = new Date(now.getTime() + 5 * 60000);
  return expiry;
}
```

**Correct Example (DO THIS):**
```javascript
// GOOD: Reusable function in shared/utils.js
function getOTPExpiryTime(minutes = 5) {
  const now = new Date();
  const expiry = new Date(now.getTime() + minutes * 60000);
  return expiry;
}

// Used everywhere
const expiry = getOTPExpiryTime(5);
```

**Enforcement:**
- If you write the same logic twice → **CODE REJECTED**
- If you copy-paste code → **CODE REJECTED**
- All reusable functions go into `src/shared/` or `src/utils/`

---

### 📌 KISS (Keep It Simple, Stupid)

**Rule:** Write code so simple that a junior developer can understand it in 30 seconds.

**Violation Example (DO NOT DO THIS):**
```javascript
// BAD: Too clever, hard to read
const result = data?.map(x => x.val).filter(y => y > 0).reduce((a,b) => a + b, 0) ?? 0;
```

**Correct Example (DO THIS):**
```javascript
// GOOD: Clear, readable, obvious
let total = 0;
for (const item of data) {
  const value = item.val;
  if (value > 0) {
    total = total + value;
  }
}
```

**Readability Checklist:**
- [ ] Can a junior developer read it without asking questions?
- [ ] Are variable names obvious (`userEmail` not `ue`)?
- [ ] Is the logic broken into small, clear steps?
- [ ] Are there comments explaining WHY (not WHAT)?

**If you answer NO to any → REWRITE THE CODE**

---

### 📌 SOLID (Single Responsibility Principle)

**Rule:** Each file, class, or function must do exactly ONE job and do it well.

**Violation Example (DO NOT DO THIS):**
```javascript
// BAD: This file does 5 different things

// authController.js - but it also does user management and payments
function login() { ... }
function updateProfile() { ... }
function processPayment() { ... }
function sendEmail() { ... }
function logAudit() { ... }
```

**Correct Example (DO THIS):**
```javascript
// GOOD: One file, one job

// authController.js - ONLY authentication
function login() { ... }
function logout() { ... }
function refreshToken() { ... }

// userController.js - ONLY user management
function updateProfile() { ... }
function getUserById() { ... }

// paymentController.js - ONLY payments
function processPayment() { ... }
```

**File-to-Job Mapping (MANDATORY):**

| File Pattern | Single Responsibility |
|--------------|----------------------|
| `*Controller.js` | HTTP request handling only |
| `*Service.js` | Business logic only |
| `*Repository.js` | Database queries only |
| `*Model.js` | Data structure definition only |
| `*Middleware.js` | Request filtering only |
| `*Utils.js` | Helper functions only |

---

## 📝 PART 2: FILE HEADER COMMENT (MANDATORY)

**Every single file in the project MUST start with this exact structure.**

### Template:

```javascript
/**
 * ============================================================================
 * 📄 FILE HEADER COMMENT
 * ============================================================================
 * FILE NAME        : [filename.ext]
 * WHAT THIS FILE DOES : [One sentence describing the file's sole purpose]
 * HOW IT DOES IT      : [Brief description of the technical approach]
 * DATA SOURCE         : [Where inputs come from - API, database, user input, etc.]
 * DATA DESTINATION    : [Where outputs go - database, API response, file, etc.]
 * PRINCIPLE APPLIED   : [DRY / KISS / SOLID - which one is primary]
 * ============================================================================
 */
```

### Examples:

**Backend Controller Example:**
```javascript
/**
 * ============================================================================
 * 📄 FILE HEADER COMMENT
 * ============================================================================
 * FILE NAME        : milestoneController.js
 * WHAT THIS FILE DOES : Handles all milestone-related HTTP requests and responses
 * HOW IT DOES IT      : Receives REST API calls, validates input, calls milestone service, returns JSON
 * DATA SOURCE         : HTTP request body (JSON) + JWT from Authorization header
 * DATA DESTINATION    : Returns JSON response to client + passes data to milestone service
 * PRINCIPLE APPLIED   : SOLID (This file handles HTTP concerns ONLY - no business logic)
 * ============================================================================
 */

const milestoneService = require('../services/milestoneService');

async function createMilestone(req, res) {
  // Function code here
}
```

**Mobile Screen Example (React Native):**
```javascript
/**
 * ============================================================================
 * 📄 FILE HEADER COMMENT
 * ============================================================================
 * FILE NAME        : ProjectListScreen.tsx
 * WHAT THIS FILE DOES : Displays list of projects for logged-in user
 * HOW IT DOES IT      : Fetches projects from API, renders FlatList, handles pull-to-refresh
 * DATA SOURCE         : GET /api/projects endpoint (JWT from secure storage)
 * DATA DESTINATION    : Renders UI components to mobile screen
 * PRINCIPLE APPLIED   : KISS (Simple list rendering with no complex state)
 * ============================================================================
 */

import React, { useState, useEffect } from 'react';
import { View, FlatList, Text } from 'react-native';
```

**Database Model Example:**
```javascript
/**
 * ============================================================================
 * 📄 FILE HEADER COMMENT
 * ============================================================================
 * FILE NAME        : UserModel.js
 * WHAT THIS FILE DOES : Defines User database schema and validation rules
 * HOW IT DOES IT      : Uses Prisma/Sequelize schema definition with field types and constraints
 * DATA SOURCE         : None - this is a pure schema definition
 * DATA DESTINATION    : Used by database migration system and repositories
 * PRINCIPLE APPLIED   : DRY (Centralized schema used by all other files)
 * ============================================================================
 */
```

**Utility Function Example:**
```javascript
/**
 * ============================================================================
 * 📄 FILE HEADER COMMENT
 * ============================================================================
 * FILE NAME        : dateHelpers.js
 * WHAT THIS FILE DOES : Provides reusable date formatting and calculation functions
 * HOW IT DOES IT      : Exports pure functions that take Date objects and return formatted strings
 * DATA SOURCE         : JavaScript Date objects from calling functions
 * DATA DESTINATION    : Returns formatted date strings (RWF localized format)
 * PRINCIPLE APPLIED   : DRY (Prevents date logic duplication across 20+ files)
 * ============================================================================
 */

function formatRwandanDate(date) {
  // Function code here
}
```

---

## 🧱 PART 3: CODE BLOCK COMMENTS (Mandatory for Every Logical Block)

**Every logical block of code (5+ lines) MUST have a block comment explaining it.**

### Template:

```javascript
/**
 * 🧱 CODE BLOCK: [Name of this block]
 * WHAT IT IS DOING: [One sentence describing the action]
 * WHY IT IS HERE  : [Business/technical reason this block exists]
 * PRINCIPLE       : [DRY / KISS / SOLID - which one this block follows]
 * DATA SOURCE     : [Where this block gets its input data]
 * DATA DESTINATION: [Where this block sends its output]
 */
```

### Examples:

**1. Input Validation Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Payment Input Validation
 * WHAT IT IS DOING: Checking that amount is positive and payment method exists
 * WHY IT IS HERE  : Prevent invalid payments from reaching MTN API (saves API costs)
 * PRINCIPLE       : KISS (Simple if statements, no regex)
 * DATA SOURCE     : req.body.amount and req.body.paymentMethod from client
 * DATA DESTINATION: Early return with 400 error if validation fails
 */
if (!amount || amount <= 0) {
  return res.status(400).json({ error: "Amount must be greater than 0" });
}
if (!paymentMethod || paymentMethod !== 'MTN') {
  return res.status(400).json({ error: "Only MTN Mobile Money supported in MVP" });
}
```

**2. Database Query Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Fetch Project with Milestones
 * WHAT IT IS DOING: Retrieves project from database and joins associated milestones
 * WHY IT IS HERE : Client dashboard needs both project details AND progress data in one view
 * PRINCIPLE       : DRY (Uses shared repository pattern, not raw SQL in controller)
 * DATA SOURCE     : projectId from URL parameter
 * DATA DESTINATION: Returns combined project+milestones object to service layer
 */
const project = await projectRepository.findByIdWithMilestones(projectId);
if (!project) {
  return res.status(404).json({ error: "Project not found" });
}
```

**3. API Call Block:**
```javascript
/**
 * 🧱 CODE BLOCK: MTN Mobile Money API Call
 * WHAT IT IS DOING: Sends deposit request to MTN MoMo sandbox API
 * WHY IT IS HERE  : External payment gateway integration - must be isolated for error handling
 * PRINCIPLE       : SOLID (Single responsibility - this block only handles MTN comms)
 * DATA SOURCE     : amount, phoneNumber, transactionId from previous validation
 * DATA DESTINATION: MTN API response (success/failure) with transaction reference
 */
try {
  const mtnResponse = await axios.post(
    'https://sandbox.mtn.co.rw/momo/v1/request-to-pay',
    {
      amount: amount,
      currency: 'RWF',
      externalId: transactionId,
      payer: { partyIdType: 'MSISDN', partyId: phoneNumber }
    },
    {
      headers: {
        'Authorization': `Bearer ${mtnAccessToken}`,
        'X-Reference-Id': uuidv4()
      }
    }
  );
  return { success: true, mtnReference: mtnResponse.data.referenceId };
} catch (error) {
  // Log and return failure
}
```

**4. Error Handling Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Cloudinary Upload Error Recovery
 * WHAT IT IS DOING: Catches upload failure and stores photo locally for retry
 * WHY IT IS HERE  : Network may fail on site; prevents data loss
 * PRINCIPLE       : KISS (Simple try-catch with fallback to AsyncStorage)
 * DATA SOURCE     : Cloudinary SDK error object
 * DATA DESTINATION: Saves photo to local queue + shows user retry option
 */
catch (cloudinaryError) {
  await AsyncStorage.setItem(`pending_photo_${Date.now()}`, photoBase64);
  console.error('Cloudinary upload failed, saved locally:', cloudinaryError.message);
  return { success: false, queued: true, message: "Photo saved locally. Will retry when online." };
}
```

**5. Permission Check Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Role-Based Access Control
 * WHAT IT IS DOING: Verifies user has permission to release escrow funds
 * WHY IT IS HERE  : Only client (project owner) can approve payments
 * PRINCIPLE       : SOLID (Security check isolated from business logic)
 * DATA SOURCE     : req.user.id from JWT + project.client_id from database
 * DATA DESTINATION: Returns 403 if unauthorized, proceeds if authorized
 */
const project = await projectRepository.findById(projectId);
if (req.user.role !== 'admin' && project.client_id !== req.user.id) {
  return res.status(403).json({ error: "Only project owner can approve payments" });
}
```

**6. Data Transformation Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Format Response for Mobile App
 * WHAT IT IS DOING: Converts database camelCase to mobile-friendly format
 * WHY IT IS HERE  : Mobile expects different date format and currency string
 * PRINCIPLE       : KISS (Simple mapping, no complex logic)
 * DATA SOURCE     : Raw database query results (snake_case)
 * DATA DESTINATION: Transformed JSON object sent to React Native client
 */
const formattedResponse = {
  projectId: dbResult.project_id,
  projectName: dbResult.project_name,
  formattedBudget: `${dbResult.budget.toLocaleString()} RWF`,
  createdAt: new Date(dbResult.created_at).toLocaleDateString('en-RW')
};
return res.json(formattedResponse);
```

**7. Async Background Job Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Bull Queue Email Sender
 * WHAT IT IS DOING: Processes email queue and sends via Nodemailer
 * WHY IT IS HERE  : Email sending should not block HTTP response
 * PRINCIPLE       : SOLID (Background job separated from request/response cycle)
 * DATA SOURCE     : Job data from Redis queue (to, subject, html)
 * DATA DESTINATION: SMTP server (Brevo/Gmail) with delivery status logged
 */
const emailJob = await emailQueue.add('send-kyc-approval', {
  to: user.email,
  subject: 'Your KYC has been approved',
  html: `<h1>Welcome to InkingiPro</h1><p>You can now create projects.</p>`
});
await emailQueue.process(async (job) => {
  const transporter = nodemailer.createTransport(smtpConfig);
  await transporter.sendMail(job.data);
});
```

**8. Offline Storage Block:**
```javascript
/**
 * 🧱 CODE BLOCK: Save Progress Photo to Local Queue
 * WHAT IT IS DOING: Stores photo in AsyncStorage when internet is unavailable
 * WHY IT IS HERE  : Engineers work on sites with poor connectivity
 * PRINCIPLE       : KISS (Simple key-value storage, no complex sync yet)
 * DATA SOURCE     : Camera capture (base64 string)
 * DATA DESTINATION: AsyncStorage queue array with timestamp + project ID
 */
const pendingQueue = await AsyncStorage.getItem('pending_photos');
const queue = pendingQueue ? JSON.parse(pendingQueue) : [];
queue.push({
  id: Date.now(),
  projectId: projectId,
  photoBase64: photoData,
  timestamp: new Date().toISOString()
});
await AsyncStorage.setItem('pending_photos', JSON.stringify(queue));
```

---

## 🔍 PART 4: FUNCTION COMMENTING (Mandatory for Every Function)

**Every function (exported or not) MUST have a comment block.**

### Template for Functions:

```javascript
/**
 * ============================================================================
 * 🔧 FUNCTION: [functionName]
 * ============================================================================
 * WHAT IT DOES: [One sentence describing the function's purpose]
 * PARAMETERS:
 *   - [param1] ([type]) : [description]
 *   - [param2] ([type]) : [description]
 * RETURNS: [type] - [description of what is returned]
 * WHO CALLS IT: [Which files/components call this function]
 * PRINCIPLE: [DRY / KISS / SOLID]
 * ============================================================================
 */
```

### Example:

```javascript
/**
 * ============================================================================
 * 🔧 FUNCTION: calculateMilestoneBudget
 * ============================================================================
 * WHAT IT DOES: Calculates the actual RWF amount for a milestone based on project budget
 * PARAMETERS:
 *   - projectBudget (number) : Total project budget in RWF
 *   - milestonePercentage (number) : Milestone percentage (1-100)
 * RETURNS: number - The RWF amount allocated to this milestone
 * WHO CALLS IT: milestoneService.js, paymentController.js
 * PRINCIPLE: DRY (Used by both milestone creation and payment release)
 * ============================================================================
 */
function calculateMilestoneBudget(projectBudget, milestonePercentage) {
  return (projectBudget * milestonePercentage) / 100;
}
```

---

## 📋 PART 5: VARIABLE NAMING STANDARDS

| Type | Convention | Example |
|------|------------|---------|
| **Variables** | camelCase | `userEmail`, `projectBudget`, `isActive` |
| **Constants** | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `OTP_EXPIRY_MINUTES` |
| **Functions** | camelCase (verb first) | `getUserById()`, `calculateTotal()`, `sendEmail()` |
| **Classes** | PascalCase | `UserService`, `PaymentGateway`, `MilestoneRepository` |
| **Files** | camelCase or PascalCase | `userService.js`, `ProjectListScreen.tsx` |
| **Database Columns** | snake_case | `user_id`, `created_at`, `kyc_status` |
| **API Endpoints** | kebab-case | `/api/user-profile`, `/milestone-payment` |

---

## 🚫 PART 6: WHAT IS FORBIDDEN

### Absolutely NOT Allowed:

1. **Magic Numbers**
```javascript
// FORBIDDEN
if (timeout > 300000) { ... }

// ALLOWED
const OTP_EXPIRY_MS = 5 * 60 * 1000; // 5 minutes
if (timeout > OTP_EXPIRY_MS) { ... }
```

2. **Console.log in Production Code** (Use logger)
```javascript
// FORBIDDEN
console.log("User logged in");

// ALLOWED
logger.info("User logged in", { userId: user.id });
```

3. **Hardcoded Secrets**
```javascript
// FORBIDDEN
const apiKey = "sk_live_123456789";

// ALLOWED
const apiKey = process.env.MTN_API_KEY;
```

4. **Comments that Lie** (Comments must match code exactly)
5. **Functions Longer Than 50 Lines** (Refactor into smaller functions)
6. **Nested Callbacks More Than 3 Levels Deep** (Use async/await)
7. **Empty Catch Blocks**
```javascript
// FORBIDDEN
try { ... } catch (e) { }

// ALLOWED
try { ... } catch (error) {
  logger.error("Payment failed", { error: error.message });
  throw new Error("Payment processing failed");
}
```

---

## ✅ PART 7: CODE REVIEW CHECKLIST

Before submitting any Pull Request, verify:

### File Header (MANDATORY)
- [ ] Every file has the `📄 FILE HEADER COMMENT` at the very top
- [ ] WHAT, HOW, DATA SOURCE, DATA DESTINATION, PRINCIPLE are filled

### Code Blocks (MANDATORY)
- [ ] Every logical block (5+ lines) has `🧱 CODE BLOCK` comment
- [ ] Block explains WHAT, WHY, PRINCIPLE, DATA SOURCE, DATA DESTINATION

### Functions (MANDATORY)
- [ ] Every function has `🔧 FUNCTION` comment block
- [ ] Parameters documented
- [ ] Return type documented

### Principles
- [ ] DRY: No duplicate code (checked with grep)
- [ ] KISS: Code is readable by junior developer
- [ ] SOLID: Each file does exactly ONE job

### Clean Code
- [ ] No console.log (use logger)
- [ ] No magic numbers (use constants)
- [ ] No hardcoded secrets (use env vars)
- [ ] No functions > 50 lines
- [ ] No nested callbacks > 3 levels

---

## 📚 PART 8: QUICK REFERENCE CARD

### File Header Template (Copy-Paste This)

```javascript
/**
 * ============================================================================
 * 📄 FILE HEADER COMMENT
 * ============================================================================
 * FILE NAME        : [filename]
 * WHAT THIS FILE DOES : [One sentence]
 * HOW IT DOES IT      : [Brief description]
 * DATA SOURCE         : [Where inputs come from]
 * DATA DESTINATION    : [Where outputs go]
 * PRINCIPLE APPLIED   : [DRY / KISS / SOLID]
 * ============================================================================
 */
```

### Code Block Template (Copy-Paste This)

```javascript
/**
 * 🧱 CODE BLOCK: [Block Name]
 * WHAT IT IS DOING: [Action description]
 * WHY IT IS HERE  : [Business/technical reason]
 * PRINCIPLE       : [DRY / KISS / SOLID]
 * DATA SOURCE     : [Input source]
 * DATA DESTINATION: [Output destination]
 */
```

### Function Template (Copy-Paste This)

```javascript
/**
 * ============================================================================
 * 🔧 FUNCTION: [functionName]
 * ============================================================================
 * WHAT IT DOES: [Description]
 * PARAMETERS:
 *   - [param1] ([type]) : [description]
 *   - [param2] ([type]) : [description]
 * RETURNS: [type] - [description]
 * WHO CALLS IT: [Calling files]
 * PRINCIPLE: [DRY / KISS / SOLID]
 * ============================================================================
 */
```

---

## 🎯 PART 9: Enforcement

| Violation | Consequence |
|-----------|-------------|
| Missing file header | PR rejected automatically |
| Missing code block comment | PR rejected, must add comments |
| DRY violation | PR rejected, must refactor |
| KISS violation | PR rejected, must simplify |
| SOLID violation | PR rejected, must split files |
| Magic numbers | PR rejected, must use constants |
| console.log in production | PR rejected, must use logger |

---

**This document is the single source of truth for coding standards. No code merges without following these rules.**

*Last Updated: May 22, 2026*