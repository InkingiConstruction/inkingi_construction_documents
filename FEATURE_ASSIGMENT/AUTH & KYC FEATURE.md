# INKINGIPRO - AUTH & KYC FEATURE DOCUMENTATION

**Feature Bundle:** Authentication + KYC (Know Your Customer)
**Status:** Ready for Development
**Estimated Time:** 20-24 Hours
**Database Tables Used:** users, otp_codes, refresh_tokens, user_sessions, kyc_documents

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

---

## 1. FEATURE OVERVIEW

### 1.1 What This Feature Does

**PART A: Authentication**
- User registers with email, phone, full name, password, and role selection
- User verifies email via 6-digit OTP sent by Nodemailer
- User verifies phone via 6-digit OTP sent by Africa's Talking SMS
- User logs in with email + password
- System returns JWT access token (1 hour expiry) and refresh token (30 days)
- User can logout and invalidate tokens
- User can request password reset via email OTP

**PART B: KYC (Know Your Customer)**
- User uploads national ID or passport to Cloudinary
- Admin views pending KYC submissions in web portal
- Admin approves or rejects KYC with reason
- User receives email notification of KYC decision
- Engineer users must also upload IER license
- Supplier users must upload business registration and tax certificate

### 1.2 User Roles That Use This Feature

| Role | Registers | Email OTP | Phone OTP | KYC Uploads | Admin Review |
|------|-----------|-----------|-----------|-------------|--------------|
| Client | Yes | Yes | Yes | National ID | Yes |
| Engineer | Yes | Yes | Yes | National ID + IER License + Insurance | Yes |
| Supervisor | Yes | Yes | Yes | National ID + Certifications + Insurance | Yes |
| Supplier | Yes | Yes | Yes | National ID + Business Reg + Tax Cert | Yes |
| Admin | No (invited) | Yes | Yes | None | N/A |

---

## 2. DATABASE TABLES USED (Names Only)

These tables are already defined in your database schema file. This feature uses ONLY these tables:

| Table Name | Purpose In This Feature |
|------------|-------------------------|
| users | Stores user account data, role, verification flags, KYC status |
| otp_codes | Stores OTP codes for email and phone verification with expiry |
| refresh_tokens | Stores refresh tokens for JWT session management |
| user_sessions | Tracks active user sessions for logout and revocation |
| kyc_documents | Stores Cloudinary URLs of uploaded KYC documents |

**No new tables needed.** All tables above already exist in your schema.

---

## 3. COMPLETE API ENDPOINTS

### 3.1 Authentication Endpoints (8 Endpoints)

| Method | Endpoint | What It Does | Request Body | Response |
|--------|----------|--------------|--------------|----------|
| POST | /api/v1/auth/register | Creates new user account | email, phone, full_name, password, role | user_id, message |
| POST | /api/v1/auth/send-email-otp | Sends 6-digit code to email | email, purpose | message, expires_in |
| POST | /api/v1/auth/verify-email | Validates email OTP | email, otp | email_verified=true |
| POST | /api/v1/auth/send-phone-otp | Sends 6-digit code via SMS | phone, purpose | message, expires_in |
| POST | /api/v1/auth/verify-phone | Validates phone OTP | phone, otp | phone_verified=true |
| POST | /api/v1/auth/login | Authenticates user | email, password | access_token, refresh_token, user |
| POST | /api/v1/auth/refresh | Gets new access token | refresh_token | new_access_token |
| POST | /api/v1/auth/logout | Ends user session | refresh_token | message |

### 3.2 KYC Endpoints (6 Endpoints)

| Method | Endpoint | What It Does | Request Body | Response |
|--------|----------|--------------|--------------|----------|
| POST | /api/v1/kyc/upload-url | Gets signed Cloudinary upload URL | document_type | url, public_id, folder |
| POST | /api/v1/kyc/documents | Saves document record after upload | document_type, cloudinary_url, public_id | document_id |
| GET | /api/v1/kyc/status | Gets user's KYC status | None | kyc_status, documents[] |
| GET | /api/v1/admin/kyc/pending | Lists all pending KYC submissions | page, limit | users[], total |
| POST | /api/v1/admin/kyc/:userId/approve | Approves user KYC | notes | message |
| POST | /api/v1/admin/kyc/:userId/reject | Rejects user KYC | reason | message |

---

## 4. FRONTEND SCREENS REQUIRED

### 4.1 React Native Mobile App Screens (8 Screens)

| Screen Name | File Location | What User Does On This Screen |
|-------------|---------------|-------------------------------|
| RoleSelectionScreen | screens/auth/RoleSelectionScreen.tsx | Taps one of 4 role cards (Client, Engineer, Supervisor, Supplier) |
| RegisterScreen | screens/auth/RegisterScreen.tsx | Enters email, phone, full name, password, confirm password |
| EmailVerificationScreen | screens/auth/EmailVerificationScreen.tsx | Enters 6-digit OTP from email, can request resend |
| PhoneVerificationScreen | screens/auth/PhoneVerificationScreen.tsx | Enters 6-digit OTP from SMS, can request resend |
| KYCUploadScreen | screens/kyc/KYCUploadScreen.tsx | Takes photo of ID or uploads from gallery |
| LoginScreen | screens/auth/LoginScreen.tsx | Enters email and password, taps login |
| ForgotPasswordScreen | screens/auth/ForgotPasswordScreen.tsx | Enters email, receives OTP, sets new password |
| ProfileScreen | screens/user/ProfileScreen.tsx | Views KYC status, uploads additional docs if rejected |

### 4.2 React Web Admin Portal Screens (2 Screens)

| Screen Name | File Location | What Admin Does On This Screen |
|-------------|---------------|--------------------------------|
| KYCPendingQueue | pages/admin/KYCPendingQueue.tsx | Views list of users with pending KYC, clicks on user to review |
| KYCReviewDetail | pages/admin/KYCReviewDetail.tsx | Views uploaded ID image, clicks Approve or Reject with reason |

---

## 5. BUSINESS RULES & VALIDATION

### 5.1 Registration Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| REG-01 | Email must be valid format (name@domain.com) | Registration rejected, show error |
| REG-02 | Email must not already exist in users table | Registration rejected, suggest login |
| REG-03 | Phone must be Rwanda format (+250XXXXXXXXX) | Registration rejected, show format example |
| REG-04 | Phone must not already exist in users table | Registration rejected |
| REG-05 | Full name must be at least 3 characters | Registration rejected |
| REG-06 | Full name must be at most 100 characters | Registration rejected |
| REG-07 | Password must be at least 8 characters | Registration rejected |
| REG-08 | Password must contain at least 1 uppercase letter | Registration rejected |
| REG-09 | Password must contain at least 1 number | Registration rejected |
| REG-10 | Password must contain at least 1 special character | Registration rejected |
| REG-11 | Role must be one of: client, engineer, supervisor, supplier | Registration rejected |
| REG-12 | Account created with email_verified = false | User cannot login until verified |
| REG-13 | Account created with phone_verified = false | User cannot use SMS features until verified |
| REG-14 | Account created with kyc_status = 'pending' | User cannot create projects or request payments |

### 5.2 OTP Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| OTP-01 | OTP code is exactly 6 digits | Validation fails |
| OTP-02 | OTP expires after 5 minutes (300 seconds) | User must request new code |
| OTP-03 | User has maximum 3 attempts to enter correct OTP | After 3 failures, code invalidated |
| OTP-04 | User can request new OTP after 30 seconds | Resend button disabled until cooldown |
| OTP-05 | Each OTP can be used only once | Second attempt fails with "Code already used" |
| OTP-06 | OTP for email verification requires email_verified = false | Cannot verify already verified email |

### 5.3 Login Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| LOG-01 | Email must exist in users table | Show "Invalid email or password" (generic) |
| LOG-02 | Password must match stored hash | Show "Invalid email or password" (generic) |
| LOG-03 | After 5 failed login attempts, account locked for 15 minutes | Show "Too many attempts. Try again in X minutes" |
| LOG-04 | User cannot login if email_verified = false | Show "Please verify your email first" |
| LOG-05 | User cannot login if is_active = false | Show "Account suspended. Contact support" |
| LOG-06 | User cannot login if is_suspended = true and suspended_until > now | Show "Account suspended until [date]" |
| LOG-07 | On successful login, last_login_at updated to current time | Database updated |
| LOG-08 | On successful login, new session created in user_sessions table | Session tracked |
| LOG-09 | Access token expires in 1 hour (3600 seconds) | Client must use refresh token |
| LOG-10 | Refresh token expires in 30 days (2,592,000 seconds) | User must login again after 30 days |

### 5.4 KYC Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| KYC-01 | User must have email_verified = true before uploading KYC | Show "Verify email first" |
| KYC-02 | User must have phone_verified = true before uploading KYC | Show "Verify phone first" |
| KYC-03 | National ID document is required for ALL roles | Upload button disabled until document added |
| KYC-04 | Engineer must also upload IER license | Cannot submit KYC without license |
| KYC-05 | Engineer must also upload professional indemnity insurance | Cannot submit KYC without insurance |
| KYC-06 | Supplier must also upload business registration certificate | Cannot submit KYC without registration |
| KYC-07 | Supplier must also upload tax compliance certificate | Cannot submit KYC without tax cert |
| KYC-08 | Document upload uses Cloudinary signed URL | URL expires after 60 minutes |
| KYC-09 | Admin can approve KYC only if all required documents uploaded | Approve button disabled until complete |
| KYC-10 | Admin must provide reason when rejecting KYC | Reject button disabled without reason |
| KYC-11 | User receives email notification when KYC approved or rejected | Email sent via Nodemailer |
| KYC-12 | After KYC approved, user kyc_status changes to 'approved' | User can now create projects |
| KYC-13 | After KYC rejected, user can upload new documents | Old documents remain for audit |

### 5.5 Token Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| TOK-01 | Access token must be sent in Authorization header | 401 Unauthorized response |
| TOK-02 | Access token format: Bearer <token> | 401 Unauthorized response |
| TOK-03 | Access token signature must be valid | 401 Unauthorized response |
| TOK-04 | Access token must not be expired | 401 with "Token expired" |
| TOK-05 | Access token JTI must not be in revoked list | 401 with "Token revoked" |
| TOK-06 | Refresh token must be stored in refresh_tokens table | 401 with "Invalid refresh token" |
| TOK-07 | Refresh token must not be revoked | 401 with "Token revoked" |
| TOK-08 | Refresh token must not be expired | 401 with "Session expired, login again" |
| TOK-09 | When logout, both access and refresh tokens revoked | Tokens added to denylist |
| TOK-10 | When password changed, all user tokens revoked | User must login again on all devices |

---

## 6. TESTING CHECKLIST WITH CHECKBOXES

### 6.1 Registration Testing (9 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 1 | Register new client with valid data | User created, email_verified=false, phone_verified=false, kyc_status=pending | ☐ |
| 2 | Register with existing email | Error: EMAIL_ALREADY_REGISTERED | ☐ |
| 3 | Register with existing phone | Error: PHONE_ALREADY_REGISTERED | ☐ |
| 4 | Register with invalid email format | Error: INVALID_EMAIL_FORMAT | ☐ |
| 5 | Register with wrong phone format | Error: INVALID_PHONE_FORMAT | ☐ |
| 6 | Register with weak password (no uppercase) | Error: WEAK_PASSWORD | ☐ |
| 7 | Register with weak password (too short) | Error: WEAK_PASSWORD | ☐ |
| 8 | Register with all 4 roles | Each role saved correctly in database | ☐ |
| 9 | Password stored as bcrypt hash | Cannot read plaintext password from database | ☐ |

### 6.2 Email OTP Testing (7 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 10 | Send OTP to valid email | 200 OK, email received within 30 seconds | ☐ |
| 11 | Send OTP to non-existent email | 200 OK (for security, don't reveal email exists) | ☐ |
| 12 | Verify with correct OTP | 200 OK, email_verified changes to true | ☐ |
| 13 | Verify with wrong OTP | 400 Error, INVALID_OTP | ☐ |
| 14 | Verify with expired OTP (after 5 minutes) | 400 Error, OTP_EXPIRED | ☐ |
| 15 | Verify with OTP after 3 wrong attempts | MAX_ATTEMPTS_EXCEEDED, OTP invalidated | ☐ |
| 16 | Resend OTP after 30 seconds | New OTP sent, old OTP invalidated | ☐ |

### 6.3 Phone OTP Testing (6 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 17 | Send OTP to valid Rwandan phone (+250788123456) | SMS received within 15 seconds | ☐ |
| 18 | Send OTP to invalid phone format | 400 Error, INVALID_PHONE_FORMAT | ☐ |
| 19 | Verify with correct OTP | 200 OK, phone_verified changes to true | ☐ |
| 20 | Verify with wrong OTP | 400 Error, INVALID_OTP | ☐ |
| 21 | Verify with expired OTP | 400 Error, OTP_EXPIRED | ☐ |
| 22 | Africa's Talking API fails | System retries 3 times, logs error, returns error to user | ☐ |

### 6.4 Login Testing (8 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 23 | Login with correct credentials | 200 OK, access_token and refresh_token returned | ☐ |
| 24 | Login with wrong password | 401 Error, INVALID_CREDENTIALS | ☐ |
| 25 | Login with non-existent email | 401 Error, INVALID_CREDENTIALS (generic message) | ☐ |
| 26 | Login 5 times with wrong password | ACCOUNT_LOCKED for 15 minutes | ☐ |
| 27 | Login with unverified email | 403 Error, EMAIL_NOT_VERIFIED | ☐ |
| 28 | Login with suspended account | 403 Error, ACCOUNT_SUSPENDED | ☐ |
| 29 | Refresh token after access token expires | New access_token returned | ☐ |
| 30 | Logout with valid token | 200 OK, token revoked, cannot use again | ☐ |

### 6.5 KYC Upload Testing (8 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 31 | Get signed Cloudinary upload URL | Returns URL, public_id, folder name | ☐ |
| 32 | Upload national ID to Cloudinary | File appears in Cloudinary dashboard, URL returned | ☐ |
| 33 | Save document record to kyc_documents table | Record created with correct user_id and URL | ☐ |
| 34 | Engineer uploads IER license | Additional document saved, linked to user | ☐ |
| 35 | Supplier uploads business registration | Additional document saved, linked to user | ☐ |
| 36 | User without email verification tries to upload KYC | 403 Error, VERIFY_EMAIL_FIRST | ☐ |
| 37 | User without phone verification tries to upload KYC | 403 Error, VERIFY_PHONE_FIRST | ☐ |
| 38 | User uploads document after KYC rejected | New document saved, kyc_status resets to submitted | ☐ |

### 6.6 Admin KYC Review Testing (7 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 39 | Admin views pending KYC queue | List of users with kyc_status='submitted' | ☐ |
| 40 | Admin clicks on user to review | User details and uploaded documents displayed | ☐ |
| 41 | Admin approves KYC | user.kyc_status changes to 'approved' | ☐ |
| 42 | Admin rejects KYC with reason | user.kyc_status changes to 'rejected', reason saved | ☐ |
| 43 | User receives email on KYC approval | Email delivered via Nodemailer | ☐ |
| 44 | User receives email on KYC rejection | Email includes rejection reason | ☐ |
| 45 | Admin cannot approve if required documents missing | Approve button disabled | ☐ |

### 6.7 Password Reset Testing (5 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 46 | Request password reset with valid email | OTP sent to email | ☐ |
| 47 | Request password reset with non-existent email | Same success message (security, don't reveal) | ☐ |
| 48 | Reset password with correct OTP | Password changed, all sessions revoked | ☐ |
| 49 | Reset password with wrong OTP | 400 Error, INVALID_OTP | ☐ |
| 50 | Login with new password after reset | Successful login | ☐ |

---

## 7. COMMON PROBLEMS & SOLUTIONS

### Problem 1: Email OTP goes to spam folder

**Solution:** Configure SPF, DKIM, and DMARC DNS records for your email domain. Use a trusted SMTP provider like Brevo or SendGrid instead of Gmail for production.

**How to verify it's fixed:** Check email headers for "spf=pass" and "dkim=pass"

---

### Problem 2: SMS OTP not arriving on MTN or Airtel networks

**Solution:** Implement retry logic with exponential backoff. Try sending again after 2 seconds, then 4 seconds, then 8 seconds. Also log failed deliveries to monitor which networks have issues.

**How to verify it's fixed:** Send 10 test SMS to different network providers (MTN, Airtel). All should arrive within 30 seconds.

---

### Problem 3: Cloudinary upload URL expires before user uploads

**Solution:** Generate signed URL with 60 minute expiry. Show countdown timer on KYC upload screen. If time runs out, automatically request new URL without user noticing.

**How to verify it's fixed:** Generate URL, wait 61 minutes, attempt upload. Should fail with 401. Then request new URL, upload should succeed.

---

### Problem 4: Two users register with same email at exact same time

**Solution:** Database unique constraint on email column prevents duplicate. Wrap registration in transaction with SELECT FOR UPDATE to lock the email check.

**How to verify it's fixed:** Send 10 concurrent registration requests with same email. Only 1 succeeds, 9 get EMAIL_ALREADY_REGISTERED error.

---

### Problem 5: JWT token stolen from device

**Solution:** Keep access token expiry short (1 hour). Use refresh token rotation - each refresh generates new refresh token and invalidates old one. Store tokens in secure storage (react-native-encrypted-storage, not AsyncStorage).

**How to verify it's fixed:** Extract token from device, try to use on different device. Token works but expires in 1 hour. Refresh token can only be used once.

---

### Problem 6: Admin approves KYC for user who hasn't uploaded all documents

**Solution:** Backend checks required documents based on user role before allowing approval. Client role needs only national ID. Engineer needs ID + IER license + insurance. Supplier needs ID + business registration + tax certificate.

**How to verify it's fixed:** Try to approve engineer who only uploaded ID. API returns 400 error "Missing required documents: IER License"

---

### Problem 7: User loses phone and cannot receive SMS OTP

**Solution:** Provide backup verification methods. User can verify via email instead of phone for login. For critical actions, support team can manually verify identity using KYC documents.

**How to verify it's fixed:** Login screen has "Verify another way" link that sends OTP to email instead.

---

### Problem 8: Africa's Talking API rate limit exceeded

**Solution:** Queue SMS requests using Bull queue. Process maximum 10 SMS per second. Retry failed sends with exponential backoff. Monitor queue length and alert if backlog grows.

**How to verify it's fixed:** Send 100 OTP requests in 1 second. All are queued and delivered within 10 seconds. No API rate limit errors.

---

## 8. TIME TRACKING & TASK BREAKDOWN

### 8.1 Developer 1 Tasks (Authentication Focus)

| Task # | Task Description | Hours | Completed? | Actual Hours |
|--------|------------------|-------|------------|--------------|
| 1 | Create users table migration | 1 | ☐ | |
| 2 | Create otp_codes table migration | 0.5 | ☐ | |
| 3 | Create refresh_tokens table migration | 0.5 | ☐ | |
| 4 | Create user_sessions table migration | 0.5 | ☐ | |
| 5 | Implement POST /auth/register endpoint | 2 | ☐ | |
| 6 | Implement password hashing with bcrypt | 1 | ☐ | |
| 7 | Implement POST /auth/send-email-otp endpoint | 2 | ☐ | |
| 8 | Integrate Nodemailer with SMTP | 1.5 | ☐ | |
| 9 | Implement POST /auth/verify-email endpoint | 1.5 | ☐ | |
| 10 | Implement POST /auth/send-phone-otp endpoint | 2 | ☐ | |
| 11 | Integrate Africa's Talking SMS SDK | 1.5 | ☐ | |
| 12 | Implement POST /auth/verify-phone endpoint | 1.5 | ☐ | |
| 13 | Implement POST /auth/login endpoint with JWT | 2 | ☐ | |
| 14 | Implement POST /auth/refresh endpoint | 1.5 | ☐ | |
| 15 | Implement POST /auth/logout endpoint | 1 | ☐ | |
| 16 | Implement POST /auth/forgot-password endpoint | 1.5 | ☐ | |
| 17 | Implement POST /auth/reset-password endpoint | 1.5 | ☐ | |
| 18 | Write unit tests for all auth endpoints | 3 | ☐ | |
| **Total** | | **24.5** | | |

### 8.2 Developer 2 Tasks (KYC + Admin Focus)

| Task # | Task Description | Hours | Completed? | Actual Hours |
|--------|------------------|-------|------------|--------------|
| 1 | Create kyc_documents table migration | 0.5 | ☐ | |
| 2 | Configure Cloudinary SDK | 1 | ☐ | |
| 3 | Implement POST /kyc/upload-url endpoint | 2 | ☐ | |
| 4 | Implement POST /kyc/documents endpoint | 1.5 | ☐ | |
| 5 | Implement GET /kyc/status endpoint | 1 | ☐ | |
| 6 | Implement role-based document requirements logic | 2 | ☐ | |
| 7 | Implement GET /admin/kyc/pending endpoint | 1.5 | ☐ | |
| 8 | Implement POST /admin/kyc/:userId/approve endpoint | 1.5 | ☐ | |
| 9 | Implement POST /admin/kyc/:userId/reject endpoint | 1.5 | ☐ | |
| 10 | Implement email notification on KYC decision | 2 | ☐ | |
| 11 | Create RoleSelectionScreen (React Native) | 2 | ☐ | |
| 12 | Create RegisterScreen (React Native) | 2 | ☐ | |
| 13 | Create EmailVerificationScreen (React Native) | 1.5 | ☐ | |
| 14 | Create PhoneVerificationScreen (React Native) | 1.5 | ☐ | |
| 15 | Create KYCUploadScreen (React Native) | 2.5 | ☐ | |
| 16 | Create LoginScreen (React Native) | 1.5 | ☐ | |
| 17 | Create KYCPendingQueue (React Admin) | 2 | ☐ | |
| 18 | Create KYCReviewDetail (React Admin) | 2 | ☐ | |
| **Total** | | **28.5** | | |

### 8.3 Testing & Integration Tasks (Both Developers)

| Task # | Task Description | Hours | Completed? | Actual Hours |
|--------|------------------|-------|------------|--------------|
| 1 | Run all 50 test cases from Section 6 | 4 | ☐ | |
| 2 | Fix bugs found during testing | 4 | ☐ | |
| 3 | Deploy to Render staging environment | 1 | ☐ | |
| 4 | Test complete user journey from registration to KYC approval | 2 | ☐ | |
| 5 | Performance test: 50 concurrent registrations | 1 | ☐ | |
| 6 | Security test: SQL injection on all endpoints | 1 | ☐ | |
| 7 | Security test: JWT tampering | 1 | ☐ | |
| 8 | Document API with Swagger | 2 | ☐ | |
| **Total** | | **16** | | |

### 8.4 Summary Timeline

| Day | Developer 1 Tasks | Developer 2 Tasks | Both Together |
|-----|-------------------|-------------------|---------------|
| Day 1 | Auth tables + register endpoint (Tasks 1-6) | KYC tables + Cloudinary setup (Tasks 1-4) | - |
| Day 2 | Email OTP endpoints (Tasks 7-9) | KYC endpoints + role requirements (Tasks 5-6) | - |
| Day 3 | Phone OTP + Africa's Talking (Tasks 10-12) | Admin KYC endpoints (Tasks 7-10) | - |
| Day 4 | Login + JWT endpoints (Tasks 13-15) | Mobile screens (Tasks 11-16) | - |
| Day 5 | Password reset (Tasks 16-17) | Admin screens (Tasks 17-18) | - |
| Day 6 | Unit tests (Task 18) | - | - |
| Day 7 | - | - | Testing all 50 cases (4 hours) + Bug fixes (4 hours) |
| Day 8 | - | - | Deploy + Final testing (4 hours) + Swagger docs (2 hours) |

### 8.5 Feature Completion Checklist

Before marking this feature as DONE, verify:

| # | Item | Status |
|---|------|--------|
| 1 | All 8 auth API endpoints working | ☐ |
| 2 | All 6 KYC API endpoints working | ☐ |
| 3 | All 8 mobile screens built and navigable | ☐ |
| 4 | All 2 admin screens built | ☐ |
| 5 | All 50 test cases passed | ☐ |
| 6 | No console.log or debug code in production | ☐ |
| 7 | All environment variables documented | ☐ |
| 8 | Swagger/OpenAPI docs updated | ☐ |
| 9 | Deployed to staging and tested with real phone numbers | ☐ |
| 10 | Client can register → verify → upload KYC → admin approves → user logs in | ☐ |

---

**END OF AUTH & KYC FEATURE DOCUMENTATION**

**Next Feature Ready When You Are:** Project Management (PROJ-01, PROJ-05, PROJ-06)