# InkingiPro - User Journey & Workflow Documentation
## Complete Step-by-Step Flows for Every User Role

**Version:** 1.0
**Status:** Final
**Date:** May 22, 2026
**Auth Framework:** Better Auth (passwordless + OTP)

---

## Document Purpose

This document provides **text-based step-by-step workflows** for every user role in the InkingiPro system. Each flow is written as sequential steps from user arrival to task completion. No diagrams — pure sequential instructions for developers to implement.

---

## Part 1: CLIENT - Complete End-to-End Flow

### 1.1 Client First-Time Onboarding (Registration to First Project Funded)

**Step 1: Download and Open App**
- User downloads InkingiPro from App Store (iOS) or Play Store (Android)
- User taps app icon to open
- Splash screen displays for 3 seconds while checking for existing session (none exists)

**Step 2: Select Role**
- System displays "Choose Your Role" screen with 5 role cards: Client, Engineer, Supervisor, Supplier, Admin
- User taps "Client" card
- System highlights selected role and enables "Continue" button

**Step 3: Enter Registration Information**
- System displays registration form with fields:
  - Full Name (text input, required)
  - Email Address (email format validation, required)
  - Phone Number (Rwanda format +250XXXXXXXXX, required)
  - Password (minimum 8 characters, 1 number, 1 uppercase)
  - Confirm Password (must match password)
- User fills all fields and taps "Sign Up with Email"

**Step 4: Verify Email Address**
- System generates 6-digit OTP and sends to user's email via Nodemailer
- System displays "Enter Verification Code" screen with 6 input boxes
- System starts 5-minute countdown timer
- User checks email, retrieves code, enters digits
- User taps "Verify Email"
- System validates OTP from Redis storage
- If valid: Proceeds to step 5
- If expired: System offers "Resend Code" button (max 3 attempts)

**Step 5: Verify Phone Number**
- System generates new 6-digit OTP and sends via SMS (Africa's Talking)
- System displays "Verify Your Phone" screen
- User receives SMS, enters code
- User taps "Verify Phone"
- System validates OTP from Redis
- Both email and phone now marked as verified in database

**Step 6: Upload KYC Documents**
- System displays "Identity Verification - Upload Documents" screen
- System lists required documents for Client role:
  - National ID or Passport (photo upload)
  - Proof of Address (optional but recommended)
  - Proof of Property Ownership (if applicable)
- For each document:
  - User taps "Upload" button
  - System opens camera or file picker
  - User captures or selects image
  - System uploads directly to Cloudinary (signed URL preset)
  - Cloudinary returns public URL
  - System saves URL to kyc_documents table with status "pending"
- User sees progress indicator for each upload
- After all documents uploaded, user taps "Submit for Verification"
- System updates user.kyc_status to "submitted"
- System creates Bull job to notify admins (email + push)

**Step 7: Wait for KYC Approval**
- System displays "KYC Under Review" screen with message:
  - "Your identity is being verified by our team. This typically takes less than 24 hours. You will receive an email confirmation once approved."
- System sends confirmation email: "KYC Documents Received - Under Review"
- User closes app or waits

**Step 8: Receive KYC Approval Email**
- Admin reviews documents in web portal (within 24 hours)
- Admin clicks "Approve" button
- System updates user.kyc_status to "approved"
- System triggers Bull job to send email via Nodemailer
- User receives email: "Your identity has been verified - Start Building"
- User opens app

**Step 9: Login After Approval**
- User opens app
- System displays login screen (session expired)
- User enters phone number
- System sends OTP via SMS
- User enters OTP
- System validates and creates Better Auth session
- System redirects to Client Dashboard (now with full access)

**Step 10: Create First Project**
- System displays "Create New Project" button prominently
- User taps button
- System displays project creation form with sections:

**Section A: Basic Information**
- Project Name (text, required)
- Project Description (multi-line text, required)
- Project Category dropdown (Residential/Commercial/Industrial/Infrastructure)
- Start Date (date picker)
- End Date (date picker)

**Section B: Budget**
- Total Budget (number input, RWF format, required)
- Currency selector (RWF default, USD toggle for diaspora)

**Section C: Site Location**
- "Draw GPS Boundary on Map" button
- User taps → Opens Google Maps integration (react-native-maps)
- User taps to add polygon points around property
- System saves polygon as GeoJSON in projects.gps_boundary
- User confirms boundary

**Section D: Site Documents**
- "Upload Site Photos" button (minimum 3 photos)
- "Upload Architectural Plans" button (PDF or DWG files)
- Each upload goes directly to Cloudinary
- System stores URLs in projects table

- User taps "Create Project" at bottom
- System validates all fields
- System inserts project record with status "draft"
- System redirects to Project Detail screen

**Step 11: Fund Escrow Account**
- System displays "Fund Your Project" banner (empty escrow)
- User taps "Deposit Funds"
- System displays payment options:
  - MTN Mobile Money
  - Airtel Money
  - Bank Transfer (wire)
- User selects MTN Mobile Money
- System displays amount input with minimum 100,000 RWF
- User enters amount (e.g., 5,000,000 RWF)
- User taps "Confirm Deposit"
- System redirects to MTN MoMo payment gateway (webview)
- User completes payment on MTN interface
- MTN sends webhook to Render API endpoint
- System validates webhook signature
- System creates transaction record (type: deposit)
- System updates escrow_accounts.balance (+amount)
- System sends receipt email via Nodemailer
- System sends push notification: "Funds received! Ready to start construction."
- User returns to app and sees updated balance

**Step 12: Invite Engineer**
- System displays "No Engineer Assigned" warning
- User taps "Invite Engineer"
- System displays search/email input
- Options:
  - Search verified engineers by name (from users table where role=engineer, kyc_status=approved)
  - Enter engineer's email address directly
- User selects engineer or enters email
- User taps "Send Invitation"
- System creates invitation record (type: engineer_assignment)
- System sends email via Nodemailer to engineer:
  - Subject: "Project Invitation: [Project Name]"
  - Body: "You have been invited to manage this project. Click to accept."
- System sends push notification to engineer's device (if registered)
- User sees "Invitation Sent - Pending Acceptance" status

**Step 13: Engineer Accepts (waiting period)**
- Engineer opens invitation email
- Engineer clicks "Accept"
- Engineer logs into app
- System updates project with engineer_id
- System sends push notification to client: "Engineer has accepted your invitation"
- User sees "Engineer Assigned - Ready to Start"

**Step 14: Monitor Milestone Progress**
- Engineer creates milestones and BoQ (see Engineer Flow)
- System sends push notification to client when milestones ready
- User opens app, sees active project with milestones
- User taps project to view timeline
- System displays:
  - Gantt-style progress bars
  - Each milestone with status (pending/active/complete)
  - Budget breakdown vs actual

**Step 15: Receive Payment Request**
- Engineer completes work on milestone
- Engineer taps "Request Payment" and uploads completion photos
- System sends push notification to client: "Milestone [X] ready for approval"
- User opens app, sees notification badge
- User taps notification → opens Milestone Review screen

**Step 16: Review Milestone Payment Request**
- System displays Milestone Review screen with:
  - Milestone name and description
  - Amount requested (from milestone budget_pct * total budget)
  - Engineer's completion photos (loaded from Cloudinary)
  - Supervisor inspection report (if submitted)
  - Quality rating (if inspected)
  - "Approve", "Request Revision", or "Dispute" buttons

**Step 17: Case A - Approve Payment**
- User reviews all evidence
- User taps "Approve Payment"
- System displays confirmation dialog: "Release [amount] RWF from escrow?"
- User taps "Confirm"
- System updates escrow_accounts.balance (-amount)
- System updates escrow_accounts.locked_balance (-amount if was locked)
- System creates transaction record (type: release, actor_id: client)
- System sends email receipt to engineer via Nodemailer
- System sends push notification to engineer: "Payment approved! Funds released."
- Milestone status updates to "paid"
- User returns to dashboard

**Step 18: Case B - Request Revision**
- User taps "Request Revision"
- System displays text input: "Explain what needs to be fixed"
- User enters detailed revision requirements
- User taps "Submit Revision Request"
- System updates milestone status to "revision_required"
- System sends push notification to engineer: "Client requested changes on milestone [X]"
- System sends email with revision notes
- User returns to dashboard, funds remain in escrow

**Step 19: Case C - Initiate Dispute**
- User taps "Dispute"
- System displays dispute form with:
  - Category dropdown (Quality/Timeline/Cost/Other)
  - Description (required, min 50 characters)
  - Evidence upload button (photos/documents to Cloudinary)
- User fills form, uploads evidence
- User taps "Submit Dispute"
- System creates dispute record with status "open"
- System updates escrow_accounts.locked_balance (+amount)
- System sends push notifications to:
  - Engineer (within 60 seconds)
  - Supervisor (if assigned)
  - Admin team
- System sends emails to all parties via Nodemailer
- System redirects user to Dispute Tracking screen

**Step 20: Track Dispute Resolution**
- System displays Dispute Tracking screen with:
  - Dispute ID and status
  - Timeline of events
  - Evidence submitted
  - Mediator name (once assigned)
  - Resolution ETA (14 days average)
- Admin mediates and issues decision (see Admin Flow)
- System sends push notification: "Dispute resolution issued"
- User reviews decision, accepts or appeals (once)

**Step 21: View Project Completion**
- After all milestones paid, project status becomes "completed"
- System displays "Project Complete" certificate (PDF generated)
- User can download all project documents as ZIP from Cloudinary
- User can rate engineer (1-5 stars)
- User can leave review comment

**CLIENT FLOW END** — Client has completed full project lifecycle from registration to project completion.

---

## Part 2: ENGINEER - Complete End-to-End Flow

### 2.1 Engineer First-Time Onboarding

**Step 1-5:** Same as Client steps 1-5 (registration, email OTP, phone OTP)

**Step 6: Upload Engineer KYC Documents**
- System displays KYC upload screen for Engineer role with specific requirements:
  - National ID or Passport (required)
  - IER Professional License (required) - with license number field
  - Professional Indemnity Insurance Certificate (required, min 10M RWF)
  - CV / Professional Certifications (required)
  - Portfolio of past projects (optional)
- For IER License:
  - User enters license number
  - System calls IER API endpoint automatically
  - System displays "License Valid" or "License Invalid - Please verify"
  - If invalid, user cannot submit until corrected
- User uploads all documents to Cloudinary
- User taps "Submit for Verification"

**Step 7: Wait for KYC Approval**
- IER license already auto-verified instantly
- Admin manually reviews insurance and credentials (24h)
- System sends email when approved: "Your engineer profile is verified"

**Step 8: Login After Approval**
- Same as Client step 9

**Step 9: View Available Project Invitations**
- System displays Engineer Dashboard with sections:
  - **Pending Invitations** (projects where client invited you)
  - **Active Projects** (projects you are managing)
  - **Completed Projects** (history)

**Step 10: Accept Project Invitation**
- User sees invitation card for "Villa Nyarutarama"
- User taps "View Details"
- System displays project summary:
  - Client name and contact
  - Budget amount
  - Location (map preview)
  - Site photos from Cloudinary
- User taps "Accept Invitation"
- System updates project with engineer_id
- System sends push notification to client: "Engineer accepted project"
- System redirects to Project Management screen

**Step 11: Create Milestones**
- System displays "No Milestones Created" warning
- User taps "Create Milestone Structure"
- System displays milestone builder:
  - Milestone 1: Name, budget percentage, duration days, acceptance criteria
  - User adds milestones until total percentage = 100%
  - System validates sum before allowing submit
  - Example milestones:
    - Site Preparation (10%, 5 days)
    - Foundation (25%, 15 days)
    - Framing (30%, 20 days)
    - Roofing (20%, 10 days)
    - Finishing (15%, 15 days)
- User taps "Save Milestones"
- System inserts milestone records linked to project
- System sends push notification to client: "Engineer has defined project milestones"

**Step 12: Create Bill of Quantities (BoQ)**
- For each milestone, user can add BoQ items
- User selects milestone, taps "Add BoQ Item"
- System displays BoQ form:
  - Category (dropdown: Concrete/Steel/Timber/Finishes/Labor/Equipment)
  - Material name
  - Quantity
  - Unit (bags/cubic meters/pieces/lumpsum)
  - Unit price (RWF)
- System auto-calculates total
- User can import market prices from historical data (system suggests avg)
- User taps "Save Item"
- System stores in boq_items table
- Repeat for all required materials

**Step 13: Daily Progress Upload**
- User visits site daily
- User opens app, taps active project
- User taps "Upload Daily Progress"
- System opens camera (react-native-camera)
- User takes 5-10 photos of site progress
- User can optionally record 2-minute video
- System automatically embeds GPS coordinates and timestamp
- System uploads directly to Cloudinary to folder /projects/{id}/progress/
- Cloudinary returns URLs
- System stores URLs in JSONB array in progress table
- System sends push notification to client: "New progress photos uploaded"
- System also notifies supervisor if milestone near completion

**Step 14: Request Milestone Payment**
- When milestone work is complete:
- User navigates to milestone, taps "Request Payment"
- System displays completion form:
  - Confirm work 100% complete (checkbox)
  - Upload completion photos (minimum 5, Cloudinary upload)
  - Add completion notes (optional)
- User taps "Submit for Review"
- System updates milestone status to "pending_supervisor"
- System sends push notification to supervisor: "Inspection required for milestone [X]"
- System sends email to supervisor via Nodemailer
- System also sends notification to client: "Milestone complete - awaiting inspection"

**Step 15: Wait for Supervisor Inspection**
- Supervisor has 48 hours to complete inspection (see Supervisor Flow)
- User sees "Pending Inspection" status in dashboard
- User receives push notification when inspection submitted:
  - If approved: "Inspection passed! Waiting for client payment approval."
  - If revision needed: "Inspection failed. Check supervisor comments."

**Step 16: Receive Payment (after client approval)**
- Client approves payment (see Client Flow steps 16-17)
- System sends push notification to engineer: "Payment approved! [amount] RWF released."
- User views transaction in Payment History
- Funds are available in engineer's linked payment method (MTN/Airtel/Bank)
- System generates payment receipt PDF, stores on Cloudinary, emails to engineer

**Step 17: Manage Supply Chain (RFQ to Suppliers)**
- When materials needed:
- User taps "Create RFQ"
- System displays RFQ form:
  - Select milestone
  - Material specifications (JSONB structured)
  - Quantity needed
  - Delivery deadline
  - Preferred suppliers (optional, else system matches)
- User submits RFQ
- System matches suppliers by:
  - Category of materials
  - Geographic area (project location within supplier service area)
- System sends push notifications + emails to matched suppliers (within 10 minutes)
- User receives quotes over next 1-3 days
- System ranks quotes by price, delivery time, supplier rating
- User reviews quotes, taps "Select" on best quote
- System generates purchase order PDF (pdfkit)
- System sends PO to supplier
- System creates delivery tracking record

**Step 18: Confirm Material Delivery**
- Supplier delivers materials and confirms via app (GPS + photos)
- System sends push notification to engineer: "Materials delivered - confirm receipt"
- User visits site, inspects delivered materials
- User taps "Confirm Delivery" or "Reject Delivery"
- If confirm: System triggers auto-payment to supplier (within 48 hours)
- If reject: User adds reason, supplier must re-deliver or refund

**Step 19: Handle Revision Requests**
- If client requests revision (Step 18 from Client Flow):
- User receives push notification: "Client requested changes on milestone [X]"
- User opens app, reads revision notes
- User performs required fixes
- User taps "Resubmit for Review"
- System notifies supervisor again for re-inspection
- Cycle continues until client approves

**Step 20: Complete Project**
- After all milestones paid:
- System updates project status to "completed"
- User receives "Project Complete" notification
- User can view client rating (1-5 stars)
- User adds project to portfolio within profile

**ENGINEER FLOW END** — Engineer has completed full project management cycle from invitation to completion.

---

## Part 3: SUPERVISOR - Complete End-to-End Flow

### 3.1 Supervisor First-Time Onboarding

**Step 1-5:** Same registration steps as Client (email + phone OTP)

**Step 6: Upload Supervisor KYC Documents**
- System displays KYC upload screen for Supervisor role:
  - National ID or Passport (required)
  - Professional Certifications (construction inspection, required)
  - Professional Indemnity Insurance (required)
  - Experience letters from previous clients (recommended)
  - Quality control training certificates (optional)
- User uploads all documents to Cloudinary
- User taps "Submit for Verification"

**Step 7: Wait for KYC Approval**
- Admin reviews certifications (24-48 hours)
- System sends email when approved: "Your supervisor profile is verified. You can now accept inspection assignments."

**Step 8: Login After Approval**
- Same login flow as other roles

**Step 9: Receive Inspection Invitation**
- Engineer requests payment on milestone
- System automatically matches available supervisors
- Client can also manually invite specific supervisor
- System sends push notification: "New inspection request - [Project Name] - [Milestone Name]"
- System sends email with details
- User opens app, sees Pending Inspections section

**Step 10: Accept or Decline Inspection**
- User taps inspection request
- System displays:
  - Project name and location
  - Engineer name and contact
  - Milestone details and acceptance criteria
  - Deadline (48 hours from request)
  - Map showing project location
- User taps "Accept Inspection"
- System updates inspection record with supervisor_id
- System sends notifications:
  - To client: "Supervisor assigned to inspect milestone"
  - To engineer: "Supervisor [name] will inspect your work"
- If decline: User taps "Decline" with reason, system finds another supervisor

**Step 11: Travel to Project Site**
- User navigates to project address
- User opens app at site

**Step 12: Start Inspection (GPS Verification)**
- User taps "Start Inspection" button
- System checks device GPS coordinates
- System compares with project.gps_boundary polygon
- **If outside boundary (more than 50 meters):**
  - System displays error: "You must be at the project site to conduct inspection."
  - System shows distance to site
  - User cannot proceed until physically at site
- **If inside boundary:**
  - System proceeds to inspection checklist

**Step 13: Complete Inspection Checklist**
- System displays digital inspection form with sections:

**Section A: Structural Quality**
- Question 1: "Foundation depth meets specification?" (Yes/No)
- Question 2: "Reinforcement bars correctly spaced?" (Yes/No)
- Question 3: "Concrete mixture matches BoQ?" (Rating 1-5)
- Question 4: "Waterproofing properly applied?" (Yes/No/Partial)

**Section B: Workmanship**
- Question 5: "Wall alignment is straight and level?" (Rating 1-5)
- Question 6: "Surface finishing is smooth?" (Rating 1-5)
- Question 7: "Tile/flooring installation is even?" (Rating 1-5)

**Section C: Materials & Safety**
- Question 8: "Materials match approved specifications?" (Yes/No)
- Question 9: "Safety equipment present on site?" (Yes/No/Partial)
- Question 10: "Site is clean and organized?" (Rating 1-5)

**Section D: Custom Observations**
- Text area for additional notes (required if any rating below 3)
- User can add custom checklist items

**Step 14: Capture Inspection Photos**
- System requires minimum 5 photos (mandatory)
- For each critical area, user taps camera icon
- System opens camera, user captures photo
- System uploads to Cloudinary (/projects/{id}/inspections/)
- Each photo gets GPS coordinates and timestamp embedded
- User can add captions to photos

**Step 15: Rate Overall Quality**
- System displays star rating (1-5) for overall project quality
- User selects rating
- Rating criteria:
  - 5 stars: Exceptional, exceeds standards
  - 4 stars: Good, meets all requirements
  - 3 stars: Satisfactory, minor issues noted
  - 2 stars: Needs improvement, major issues
  - 1 star: Unacceptable, requires rework

**Step 16: Add Digital Signature**
- System displays signature pad (react-native-signature-canvas)
- User draws signature with finger or stylus
- User taps "Confirm Signature"
- System converts signature to base64 PNG
- System uploads signature to Cloudinary
- Signature represents legally binding inspection certification

**Step 17: Submit Inspection Report**
- User taps "Submit Report" button
- System validates all required fields completed
- System creates inspection record in inspections table:
  - Checklist answers stored as JSONB
  - Photos URLs stored as JSONB array
  - Rating stored
  - Digital signature URL stored
  - Signed_at timestamp recorded
- System determines overall decision:
  - **If all critical checks = Yes and rating >= 3:** Decision = "approved"
  - **If any critical check = No or rating < 3:** Decision = "revision_required"

**Step 18: Notifications Sent**
- **If approved:**
  - System sends push notification to engineer: "Inspection passed! Milestone approved."
  - System sends push notification to client: "Supervisor approved milestone. Ready for payment release."
  - System sends email to both parties via Nodemailer
  - Milestone status updates to "awaiting_client_payment"

- **If revision required:**
  - System sends push notification to engineer: "Inspection failed. See report for required fixes."
  - System sends push notification to client: "Supervisor found issues with milestone."
  - System sends detailed report email with checklist failures highlighted
  - Milestone status updates to "revision_required"
  - Engineer has 7 days to fix and request re-inspection

**Step 19: View Inspection History**
- User can view all past inspections in "My Reports" section
- Each report shows:
  - Project name
  - Date of inspection
  - Decision (approved/revision)
  - Client name
  - Engineer name
- User can download report as PDF (generated from inspection data + Cloudinary images)

**Step 20: Receive Re-inspection Request (if applicable)**
- Engineer fixes issues and requests re-inspection
- System sends push notification to supervisor: "Re-inspection requested for [Project] - [Milestone]"
- User repeats Steps 11-18 for re-inspection
- System tracks that this is re-inspection #1, #2, etc.

**Step 21: Get Rated by Client**
- After milestone payment released
- Client can rate supervisor (1-5 stars) on:
  - Punctuality (arrived on time)
  - Thoroughness (caught all issues)
  - Professionalism (communication, attitude)
- Ratings affect supervisor's visibility in client search results

**SUPERVISOR FLOW END** — Supervisor has completed full inspection lifecycle.

---

## Part 4: SUPPLIER - Complete End-to-End Flow

### 4.1 Supplier First-Time Onboarding

**Step 1-5:** Same registration steps (email + phone OTP, but use business email)

**Step 6: Upload Supplier KYC Documents**
- System displays KYC upload screen for Supplier role:
  - Business Registration Certificate (RDB) - required
  - Tax Compliance Certificate (RRA) - required
  - Director/Owner National ID - required
  - Company profile/brochure - required
  - Product catalog with prices - required
  - Bank account details (for auto-payments) - required
  - Previous client references - optional
- User uploads all documents to Cloudinary
- User taps "Submit for Verification"

**Step 7: Wait for KYC Approval**
- Admin reviews business documents (24-48 hours)
- Admin verifies business registration with RDB (manual or API)
- System sends email when approved: "Your supplier account is verified. You can now receive RFQs."

**Step 8: Login After Approval**
- Same login flow
- System displays Supplier Dashboard

**Step 9: Receive API Key (Optional Integration)**
- System displays API key section (for automated integration)
- User can generate API key for:
  - Connecting own inventory system
  - Automated quote submission
  - Delivery confirmation from warehouse
- User taps "Generate API Key"
- System creates API key record with scoped permissions
- User copies key and stores securely

**Step 10: View Available RFQs**
- Engineer creates RFQ broadcast
- System matches supplier based on:
  - Product categories (from supplier profile)
  - Service area (geographic zones supplier covers)
- System sends push notification: "New RFQ matching your products - [material name]"
- System sends email with RFQ summary
- User opens app, sees "Matching RFQs" section

**Step 11: Review RFQ Details**
- User taps RFQ
- System displays:
  - Project name and location (map)
  - Material specifications (JSONB structured)
  - Quantity required
  - Delivery deadline
  - Engineer contact info
  - Number of other suppliers viewing (anonymized)
- User assesses if they can fulfill order

**Step 12: Submit Quote**
- User taps "Submit Quote"
- System displays quote form:
  - Unit price (RWF) - required
  - Total price (auto-calculated)
  - Delivery time (days from order confirmation) - required
  - Warranty period (months) - optional
  - Additional terms and conditions - text area
  - Upload product certification (optional, to Cloudinary)
- User fills all fields
- User taps "Submit Quote"
- System stores quote in quotes table with status "pending_selection"
- System sends notification to engineer: "New quote received for RFQ [ID]"

**Step 13: Wait for Selection**
- Engineer reviews all quotes (1-3 days typical)
- Engineer may select competitor's quote
- User receives notification:
  - **If selected:** "Your quote has been selected for [Project] - Purchase order attached"
  - **If not selected:** "Your quote was not selected for this RFQ"

**Step 14: Receive Purchase Order (if selected)**
- Engineer selects user's quote
- System generates purchase order PDF (pdfkit library)
- PO includes:
  - PO number
  - Supplier name and contact
  - Material descriptions and quantities
  - Agreed price
  - Delivery deadline
  - Payment terms (auto-payment on delivery confirmation)
- System attaches PO to quote record
- System sends email with PO PDF to supplier
- System sends push notification: "Purchase order received - Prepare for delivery"

**Step 15: Prepare Materials for Delivery**
- User views PO in app
- User taps "Acknowledge PO"
- System updates order status to "preparing"
- User prepares materials

**Step 16: Start Delivery (GPS Tracking)**
- When ready to deliver:
- User opens app, selects order
- User taps "Start Delivery"
- System records start time and start GPS location
- System begins tracking delivery progress (optional real-time for client)
- User loads materials onto truck

**Step 17: Arrive at Project Site**
- User arrives at project location
- User opens app, taps "Mark Arrived"
- System checks GPS against project boundary
- **If outside 50-meter radius:**
  - System warns: "You are not at the project site. GPS shows you are [distance] meters away."
  - User cannot confirm delivery until at correct location
- **If inside boundary:**
  - System proceeds

**Step 18: Unload and Capture Proof of Delivery (PoD)**
- User unloads materials
- User taps "Capture Delivery Proof"
- System opens camera
- User takes minimum 5 photos:
  - Photo 1: Materials on truck before unloading
  - Photo 2: Unloading in progress
  - Photo 3: Materials stacked at site
  - Photo 4: Close-up of material quality
  - Photo 5: Engineer/supervisor receiving materials (optional)
- Each photo uploads to Cloudinary with GPS metadata
- User can add delivery notes

**Step 19: Confirm Delivery**
- User taps "Confirm Delivery"
- System creates delivery record with:
  - Delivery timestamp
  - GPS coordinates (array of start and end)
  - Photos URLs (JSONB array)
  - Status = "pending_confirmation"
- System sends push notification to engineer: "Materials delivered - Please confirm"
- System sends push notification to supervisor: "Materials delivered - Inspect quality"

**Step 20: Receive Delivery Confirmation**
- Engineer or supervisor visits site to inspect delivered materials
- Engineer taps "Confirm Delivery" or "Reject Delivery"
- **If confirmed:**
  - System updates delivery status to "confirmed"
  - System triggers Bull job for auto-payment
  - Within 48 hours, payment released from escrow to supplier's bank/MTN
  - User receives push notification: "Payment received - [amount] RWF"
  - User receives email receipt
- **If rejected:**
  - User receives notification: "Delivery rejected - Reason: [engineer's reason]"
  - User must arrange for pick-up or re-delivery
  - No payment released

**Step 21: View Payment History**
- User can view all payments in "Earnings" section
- Each payment shows:
  - Project name
  - PO number
  - Amount received
  - Date received
  - Status (pending/cleared)
- User can download payment receipts as PDF

**Step 22: Receive Rating**
- After delivery confirmed, engineer rates supplier (1-5 stars on):
  - Product quality
  - Delivery speed
  - Communication
  - Value for money
- Average rating displayed on supplier profile
- Higher-rated suppliers appear higher in RFQ matching

**SUPPLIER FLOW END** — Supplier has completed full RFQ-to-payment cycle.

---

## Part 5: ADMIN - Complete End-to-End Flow

### 5.1 Admin First-Time Onboarding (Internal)

**Step 1: Receive Invitation Email**
- Super Admin creates admin account via Better Auth admin panel
- System sends invitation email to admin's work email
- Email contains unique invitation link

**Step 2: Click Invitation Link**
- User clicks link in email
- System opens React Web Admin Portal (Vercel-hosted)
- System displays account setup screen

**Step 3: Set Up Account**
- User enters:
  - Full name
  - Password (minimum 12 characters, 1 number, 1 uppercase, 1 special character)
  - Confirm password
- User taps "Create Account"

**Step 4: Enable Two-Factor Authentication (2FA) - Mandatory**
- System displays QR code (Google Authenticator / TOTP)
- User scans QR code with authenticator app
- User enters 6-digit code from authenticator
- System validates code
- System generates 10 backup codes for account recovery
- User downloads backup codes to secure location
- 2FA now active on admin account

**Step 5: Access Admin Portal Dashboard**
- User logs in with email + password + 2FA code
- System displays Admin Dashboard with widgets:
  - Pending KYC reviews: X users
  - Active disputes: X cases
  - System health: All green (API, DB, Redis, Cloudinary)
  - Today's transactions: X RWF processed
  - New users this week: X

---

### 5.2 Admin Daily Operations Flow

**MODULE A: KYC Review**

**Step 1: Access KYC Queue**
- User clicks "KYC Verification" in left sidebar
- System displays list of pending submissions:
  - Sorted by oldest first (FIFO)
  - Each row shows: Name, Role, Submitted time, Document count
- User clicks on first pending user

**Step 2: Review Documents**
- System displays user information:
  - Full name, email, phone number
  - Role (Client/Engineer/Supervisor/Supplier)
  - Registered date
- System displays each uploaded document from Cloudinary
- Admin can:
  - View document (opens Cloudinary preview with signed URL)
  - Zoom in on document details
  - Download document for offline review
  - Flag document as suspicious

**Step 3: Verify Against External Databases (if applicable)**
- **For Engineer:** System automatically shows IER verification result
  - License valid: Yes/No
  - Expiry date
  - License category
- **For Supplier:** Admin manually checks RDB and RRA portals
  - Business registration number matches
  - Tax compliance valid
- Admin checks documents for:
  - Expiry dates (future expiry required)
  - Name consistency across all docs
  - Photo matches live photo (if applicable)

**Step 4: Make Decision**
- System displays decision buttons:
  - **Approve** - User becomes verified
  - **Reject** - User must re-submit
  - **Request More Info** - User uploads additional documents

**Case A: Approve**
- User clicks "Approve"
- System opens confirmation dialog
- User adds optional internal note (not visible to user)
- User clicks "Confirm Approval"
- System updates user.kyc_status = "approved"
- System triggers Bull job to send email via Nodemailer:
  - Subject: "Your identity has been verified"
  - Body: "You now have full access to InkingiPro"
- System logs action to audit_logs table

**Case B: Reject**
- User clicks "Reject"
- System displays reason input (required)
- User selects rejection reason from dropdown:
  - "Document illegible - please upload clearer copy"
  - "Document expired - please upload current version"
  - "Name mismatch between documents"
  - "Invalid license number - please verify"
  - "Business registration not found"
  - "Other (specify)"
- User adds specific instructions (required)
- User clicks "Confirm Rejection"
- System updates user.kyc_status = "rejected"
- System stores rejection reason
- System triggers email to user with rejection reason and instructions
- System logs action to audit_logs

**Case C: Request More Info**
- User clicks "Request More Info"
- System displays text input for specific request
- User types: "Please upload your professional indemnity insurance certificate"
- User clicks "Send Request"
- System sends email to user requesting additional documents
- System updates user.kyc_status = "additional_info_requested"
- User will upload new docs, admin reviews again

**Step 5: Move to Next Pending**
- After decision, system automatically loads next pending KYC
- Admin repeats steps 2-4

**Target:** 95% of KYC submissions processed within 24 hours

---

**MODULE B: Dispute Mediation**

**Step 1: Access Dispute Queue**
- User clicks "Dispute Resolution" in sidebar
- System displays list of open disputes:
  - Priority: Oldest first (SLA = 14 days)
  - Color coding: Red (overdue), Yellow (approaching deadline), Green (on track)
  - Each row shows: Dispute ID, Project, Amount, Days open, Status
- User clicks on dispute

**Step 2: Review Dispute Details**
- System displays dispute information:
  - Raised by: [User name and role]
  - Against: [Counterparty name and role]
  - Category: Quality/Timeline/Cost/Other
  - Amount in dispute: X RWF (locked in escrow)
  - Description: [User's written statement]
  - Timeline of events leading to dispute
  - Milestone status (percentage complete)

**Step 3: Review Evidence**
- System displays all uploaded evidence:
  - Photos from engineer (progress photos)
  - Inspection reports from supervisor
  - Client correspondence (in-app messages)
  - Contract documents (uploaded by either party)
  - Delivery confirmations (if materials dispute)
- Admin can view each piece of evidence from Cloudinary
- Admin can request additional evidence from either party

**Step 4: Communicate with Parties**
- System has integrated chat for mediation:
  - Admin can message client privately
  - Admin can message engineer privately
  - Admin can message both parties (group)
  - All messages logged for audit
- Admin gathers facts and attempts to mediate

**Step 5: Issue Resolution Decision**
- After investigation (average 14 days):
- User clicks "Issue Decision"
- System displays decision form:

**Decision Type A: Full Payment Release**
- Admin selects "Release full payment to engineer"
- System requires justification text
- Admin clicks "Confirm"
- System updates escrow_accounts.balance (-amount)
- System releases locked balance
- System creates transaction record
- System sends notifications to all parties
- Dispute status = "resolved_full_payment"

**Decision Type B: Partial Payment + Remediation**
- Admin selects "Partial payment to engineer + remediation required"
- Admin enters percentage to release (e.g., 60%)
- Admin specifies remediation work required from engineer (text)
- System releases partial payment, keeps remainder locked
- Milestone status = "revision_required"
- Engineer must complete remediation within 14 days
- Dispute status = "resolved_partial"

**Decision Type C: Payment Withheld + Penalty**
- Admin selects "No payment - client refund"
- Admin specifies penalty amount (deducted from escrow, goes to platform)
- Rest refunded to client
- System updates escrow_accounts.balance (-refund)
- System creates refund transaction to client's payment method
- Dispute status = "resolved_refund"

**Decision Type D: Project Termination**
- Admin selects "Terminate project"
- System freezes all remaining escrow
- System initiates refund workflow to client (minus platform fees)
- Project status = "terminated"
- Dispute status = "resolved_termination"
- Both parties notified

**Step 6: Handle Appeal (if filed)**
- Either party can file one appeal within 7 days of decision
- System assigns appeal to senior admin
- Senior admin reviews original decision + new evidence
- Senior admin either:
  - Upholds original decision
  - Reverses or modifies decision
- Appeal decision is final

---

**MODULE C: User Management**

**Step 1: View All Users**
- User clicks "User Management"
- System displays paginated table with:
  - Name, email, role, KYC status, account status (active/suspended), joined date
  - Search bar for name/email/phone
  - Filters for role and KYC status

**Step 2: View User Details**
- User clicks on any user row
- System displays full user profile:
  - All KYC documents (from Cloudinary)
  - All projects user is involved in
  - All transactions user made
  - Login history (last 10 logins with IP and device)
  - Notification preferences

**Step 3: Suspend User Account**
- If user violates terms:
- User clicks "Suspend Account"
- System displays reason input
- Admin selects suspension reason:
  - "Fraudulent activity"
  - "Multiple disputes without resolution"
  - "Harassment of other users"
  - "Payment default"
  - "Other"
- Admin sets suspension duration:
  - Temporary (7/14/30 days)
  - Permanent
- Admin clicks "Suspend"
- System updates user.account_status = "suspended"
- System sends email notification to user
- User cannot log in during suspension

**Step 4: Activate Suspended Account**
- User requests reinstatement (email or in-app)
- Admin reviews request
- Admin clicks "Activate Account"
- System updates user.account_status = "active"
- System sends email: "Your account has been reinstated"

---

**MODULE D: Audit & Compliance**

**Step 1: View Audit Log**
- User clicks "Audit Logs"
- System displays append-only log table:
  - Timestamp, Actor (user), Action, Entity Type, Entity ID, IP Address, Result
  - Searchable and filterable
  - Export to CSV button

**Step 2: Generate Compliance Report**
- User clicks "Reports" → "Compliance"
- System displays date range picker
- User selects "Last Quarter"
- User clicks "Generate Report"
- System runs PostgreSQL queries:
  - KYC completion rate by role
  - Average KYC processing time
  - Number of active licensed engineers
  - Number of verified suppliers
  - Dispute rate (% of projects)
- System generates PDF report (pdfkit)
- User downloads or emails to regulators (RBA/IER)

**Step 3: Financial Reconciliation**
- User clicks "Reports" → "Financial"
- System displays:
  - Total escrow balance across all projects
  - Total deposits this month
  - Total payments released this month
  - Platform fees collected
  - Any unmatched transactions (reconciliation exceptions)
- User can export full transaction ledger to Excel (exceljs)

---

**MODULE E: System Configuration**

**Step 1: Configure Escrow Rules**
- User clicks "System Settings" → "Escrow"
- System displays editable fields:
  - Auto-release timeout (days client has to approve, default 14)
  - Dispute response SLA (days, default 14)
  - Minimum deposit amount (RWF, default 100,000)
  - GPS verification radius (meters, default 50)
- Admin edits values
- Admin clicks "Save"
- System updates configuration table
- Changes apply to new projects only (not existing)

**Step 2: Configure Notification Templates**
- User clicks "System Settings" → "Email Templates"
- System lists all email types:
  - KYC Approval
  - KYC Rejection
  - Payment Receipt
  - Milestone Approval Request
  - Dispute Opened
  - Password Reset
- Admin selects template, edits HTML (with inline CSS)
- Admin clicks "Preview" to see rendered email
- Admin clicks "Save" to update template
- System stores template in database
- Nodemailer uses updated template for future emails

**Step 3: Configure Fees**
- User clicks "System Settings" → "Pricing"
- System displays:
  - Platform fee percentage (default 2.5%)
  - Who pays fee (client/engineer/split)
  - Minimum platform fee (RWF)
- Admin edits values
- Admin clicks "Save"
- New fee structure applies to new projects

---

**ADMIN FLOW END** — Admin has completed all daily operational tasks.

---

## Summary: User Journey Completion Checklist

| Role | Total Steps | Key Completion Milestone |
|------|-------------|--------------------------|
| **Client** | 21 steps | Project fully funded and completed |
| **Engineer** | 20 steps | Project managed from invitation to completion |
| **Supervisor** | 21 steps | Inspection submitted and client rated |
| **Supplier** | 22 steps | Delivery confirmed and payment received |
| **Admin** | Multiple modules | KYC reviewed, dispute resolved, system configured |

---

*This document replaces all diagram-based workflows and provides sequential implementation guidance for the development team.*