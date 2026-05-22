# INKINGIPRO - MILESTONE MANAGEMENT FEATURE DOCUMENTATION

**Feature:** Milestone Management (Create Milestone, Validate Budget, Client Approval)
**Feature IDs:** MIL-01, MIL-02, MIL-04 from MVP list
**Status:** Ready for Development
**Estimated Time:** 10-12 Hours
**Database Tables Used:** milestones, projects, users

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

This feature allows an Engineer to create payment milestones for a project and allows the Client to approve them.

**PART A: Create Milestone (MIL-01)**
- Engineer selects a project they are assigned to
- Engineer enters milestone title and budget percentage
- Engineer adds description and acceptance criteria
- Engineer submits to create milestone record

**PART B: Validate Budget Sum = 100% (MIL-02)**
- System checks that all milestone percentages add up to exactly 100%
- If sum is less than 100%, error message shows remaining percentage needed
- If sum is greater than 100%, error message shows overage amount
- Client cannot approve milestones until sum equals 100%

**PART C: Client Approves Milestone (MIL-04)**
- Client views list of milestones created by engineer
- Client reviews each milestone details
- Client clicks "Approve Milestones" button
- System changes milestone status from "pending" to "active"
- Payment requests can only be made on active milestones

### 1.2 Why This Feature is MVP Priority

| Reason | Explanation |
|--------|-------------|
| Payment Structure | Without milestones, there is no way to break project into payment stages |
| Client Control | Client must approve milestones before any money can be requested |
| Trust Mechanism | Clear milestones with percentages build trust between client and engineer |
| Prevents Disputes | Written acceptance criteria prevent disagreements about what "complete" means |

### 1.3 Sequential Flow Steps

**Create Milestone Flow:**
- Step 1: Engineer logs into app with verified account
- Step 2: Engineer selects project from their assigned projects list
- Step 3: Engineer taps "Create Milestones" button
- Step 4: Engineer enters milestone title (required)
- Step 5: Engineer enters budget percentage (number between 1 and 100)
- Step 6: Engineer enters description of work to be done (optional)
- Step 7: Engineer enters acceptance criteria (what defines completion)
- Step 8: Engineer taps "Add Milestone" button
- Step 9: System saves milestone with status = "pending"
- Step 10: System shows updated total percentage sum
- Step 11: Engineer repeats steps 4-10 until all milestones created
- Step 12: When total sum reaches 100%, "Submit for Client Approval" button appears

**Validate Budget Sum Flow:**
- Step 1: After each milestone added, system calculates sum of all milestone percentages
- Step 2: If sum < 100, system shows "Remaining: X% to allocate"
- Step 3: If sum > 100, system shows error "Total exceeds 100% by X%"
- Step 4: System prevents submission if sum does not equal 100
- Step 5: Only when sum = 100, system enables submission button

**Client Approval Flow:**
- Step 1: Client receives push notification "Engineer has set up milestones for your project"
- Step 2: Client opens app and navigates to project details
- Step 3: Client sees list of milestones with percentages and acceptance criteria
- Step 4: Client reviews each milestone
- Step 5: Client taps "Approve All Milestones" button
- Step 6: System updates all milestones status from "pending" to "active"
- Step 7: System sends push notification to engineer "Client approved milestones"
- Step 8: Project status changes from "draft" to "active"
- Step 9: Engineer can now request payments on active milestones

---

## 2. DATABASE TABLES USED (Names Only)

These tables are already defined in your database schema file. This feature uses ONLY these tables:

| Table Name | Purpose In This Feature |
|------------|-------------------------|
| projects | Check project exists and engineer is assigned |
| milestones | Store milestone title, percentage, description, acceptance criteria, status |
| users | Verify engineer role and client role |

**Tables accessed (read only):** projects, users

**Tables written to:** milestones

**Note:** The milestones table should already exist from your schema. If not, create migration with columns: id, project_id, title, budget_percentage, description, acceptance_criteria, status, created_by, created_at

---

## 3. COMPLETE API ENDPOINTS

### 3.1 Milestone Endpoints (7 Endpoints)

| Method | Endpoint | What It Does | Request Body | Response |
|--------|----------|--------------|--------------|----------|
| POST | /api/v1/projects/:projectId/milestones | Creates new milestone | title, budget_percentage, description, acceptance_criteria | milestone object |
| GET | /api/v1/projects/:projectId/milestones | Lists all milestones for project | None (project ID in URL) | milestones array with total sum |
| GET | /api/v1/milestones/:id | Gets single milestone details | None | milestone object |
| PUT | /api/v1/milestones/:id | Updates milestone (engineer only, before approval) | title, budget_percentage, description, acceptance_criteria | updated milestone |
| DELETE | /api/v1/milestones/:id | Deletes milestone (engineer only, before approval) | None | success message |
| POST | /api/v1/projects/:projectId/milestones/validate | Validates total percentage sum | None | validation result with sum and missing/overage |
| POST | /api/v1/projects/:projectId/milestones/approve | Client approves all milestones | None | success message, updated milestones |

### 3.2 Endpoint Details

**ENDPOINT 1: Create Milestone**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/projects/:projectId/milestones |
| Auth Required | Yes (JWT token) |
| Role Required | Engineer assigned to this project |

**Request Body:**
```json
{
    "title": "Foundation Construction",
    "budget_percentage": 25,
    "description": "Excavation, footing, foundation walls, and waterproofing",
    "acceptance_criteria": "Foundation passed structural inspection, concrete cured for 7 days, waterproofing intact"
}
```

**Validation Rules:**
| Field | Rule |
|-------|------|
| title | Required, min 5 characters, max 100 characters |
| budget_percentage | Required, integer between 1 and 100 |
| description | Optional, max 500 characters |
| acceptance_criteria | Required, min 20 characters, max 500 characters |

**Success Response (201 Created):**
```json
{
    "success": true,
    "message": "Milestone created successfully",
    "data": {
        "id": "milestone-id-123",
        "project_id": "project-id-456",
        "title": "Foundation Construction",
        "budget_percentage": 25,
        "budget_amount": 3750000,
        "description": "Excavation, footing, foundation walls, and waterproofing",
        "acceptance_criteria": "Foundation passed structural inspection, concrete cured for 7 days, waterproofing intact",
        "status": "pending",
        "created_at": "2024-01-15T10:30:00Z",
        "current_total_percentage": 25,
        "remaining_percentage": 75
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| UNAUTHORIZED | 401 | No JWT token provided |
| FORBIDDEN | 403 | User is not engineer assigned to this project |
| PROJECT_NOT_FOUND | 404 | Project ID does not exist |
| INVALID_PERCENTAGE | 400 | budget_percentage less than 1 or greater than 100 |
| TOTAL_EXCEEDS_100 | 400 | Adding this milestone would make total sum exceed 100% |
| MISSING_TITLE | 400 | Title is required |
| MISSING_ACCEPTANCE_CRITERIA | 400 | Acceptance criteria is required |

---

**ENDPOINT 2: List Project Milestones**

| Property | Value |
|----------|-------|
| Method | GET |
| URL | /api/v1/projects/:projectId/milestones |
| Auth Required | Yes (JWT token) |
| Role Required | Client (owner) OR Engineer (assigned) OR Admin |

**Success Response (200 OK):**
```json
{
    "success": true,
    "data": {
        "milestones": [
            {
                "id": "milestone-1",
                "title": "Site Preparation",
                "budget_percentage": 10,
                "budget_amount": 1500000,
                "status": "pending",
                "acceptance_criteria": "Site cleared, utilities marked, excavation ready"
            },
            {
                "id": "milestone-2",
                "title": "Foundation",
                "budget_percentage": 25,
                "budget_amount": 3750000,
                "status": "pending",
                "acceptance_criteria": "Foundation passed inspection, concrete cured"
            },
            {
                "id": "milestone-3",
                "title": "Framing",
                "budget_percentage": 35,
                "budget_amount": 5250000,
                "status": "pending",
                "acceptance_criteria": "All walls framed, roof structure complete"
            }
        ],
        "summary": {
            "total_percentage": 70,
            "remaining_percentage": 30,
            "is_complete": false,
            "can_submit": false,
            "message": "Remaining 30% to allocate. Add one more milestone."
        }
    }
}
```

---

**ENDPOINT 3: Validate Milestone Total**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/projects/:projectId/milestones/validate |
| Auth Required | Yes (JWT token) |
| Role Required | Engineer assigned to this project |

**Request Body:** (empty, uses existing milestones)

**Success Response (200 OK):**
```json
{
    "success": true,
    "data": {
        "current_total": 70,
        "is_valid": false,
        "needs": 30,
        "message": "Need 30% more to reach 100%. Please add additional milestones.",
        "can_approve": false
    }
}
```

**When total equals 100:**
```json
{
    "success": true,
    "data": {
        "current_total": 100,
        "is_valid": true,
        "needs": 0,
        "message": "Milestone total is 100%. Ready for client approval.",
        "can_approve": true
    }
}
```

---

**ENDPOINT 4: Client Approve Milestones**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/projects/:projectId/milestones/approve |
| Auth Required | Yes (JWT token) |
| Role Required | Client (project owner only) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "All milestones approved successfully",
    "data": {
        "approved_count": 3,
        "project_status": "active",
        "milestones": [
            {
                "id": "milestone-1",
                "status": "active",
                "activated_at": "2024-01-15T11:00:00Z"
            }
        ]
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| FORBIDDEN | 403 | User is not the client who owns this project |
| VALIDATION_FAILED | 400 | Milestone total percentage does not equal 100% |
| ALREADY_APPROVED | 400 | Milestones already approved previously |
| NO_MILESTONES | 400 | Project has no milestones created |

---

**ENDPOINT 5: Update Milestone**

| Property | Value |
|----------|-------|
| Method | PUT |
| URL | /api/v1/milestones/:id |
| Auth Required | Yes (JWT token) |
| Role Required | Engineer assigned to project (only before client approval) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Milestone updated successfully",
    "data": {
        "id": "milestone-id",
        "title": "Updated Foundation Work",
        "budget_percentage": 30,
        "updated_at": "2024-01-15T11:30:00Z"
    }
}
```

**Note:** After client approves milestones, update and delete are blocked.

---

**ENDPOINT 6: Delete Milestone**

| Property | Value |
|----------|-------|
| Method | DELETE |
| URL | /api/v1/milestones/:id |
| Auth Required | Yes (JWT token) |
| Role Required | Engineer assigned to project (only before client approval) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Milestone deleted successfully"
}
```

---

## 4. FRONTEND SCREENS REQUIRED

### 4.1 React Native Mobile App Screens (3 Screens)

| Screen Name | File Location | What User Does On This Screen |
|-------------|---------------|-------------------------------|
| MilestoneListScreen | screens/milestone/MilestoneListScreen.tsx | Engineer views existing milestones, sees total percentage, adds new milestones |
| CreateMilestoneScreen | screens/milestone/CreateMilestoneScreen.tsx | Engineer enters title, percentage, description, acceptance criteria |
| ApproveMilestonesScreen | screens/milestone/ApproveMilestonesScreen.tsx | Client reviews all milestones, sees total, clicks approve button |

### 4.2 Screen Details

**Screen 1: MilestoneListScreen (Engineer View)**

| Element | Description |
|---------|-------------|
| Header | "Milestones" + "+ Add Milestone" button |
| Total Percentage Card | Shows current sum, remaining needed, progress bar |
| Milestone List | Each milestone shows title, percentage, status badge, edit/delete icons |
| Status Badge | Pending (yellow), Active (green), Paid (blue) |
| Submit Button | Shows "Submit for Client Approval" only when total = 100% |
| Warning Message | Shows if total < 100 or total > 100 |

**Screen 2: CreateMilestoneScreen**

| Field | Type | Validation |
|-------|------|------------|
| Milestone Title | Text input | Required, min 5 chars |
| Budget Percentage | Number input | Required, 1-100 |
| Description | Text area | Optional |
| Acceptance Criteria | Text area | Required, min 20 chars |
| Current Total Display | Text | Shows current sum of all milestones |
| Warning Display | Text | Shows if adding would exceed 100% |
| Submit Button | Button | Disabled if validation fails |

**Screen 3: ApproveMilestonesScreen (Client View)**

| Element | Description |
|---------|-------------|
| Header | "Review Milestones" |
| Project Name | Shows project name and total budget |
| Total Confirmation | "Total: 100% of 15,000,000 RWF" |
| Milestone Cards | Each card shows title, percentage, amount in RWF, acceptance criteria |
| Read-only Mode | Client cannot edit, only view |
| Approve Button | Large green button "Approve All Milestones" |
| Warning | "Once approved, milestones cannot be changed by engineer" |

---

## 5. BUSINESS RULES & VALIDATION

### 5.1 Milestone Creation Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| MIL-01 | User must be engineer assigned to project | 403 Forbidden |
| MIL-02 | Project must have status "draft" (not yet active) | 403 PROJECT_ALREADY_ACTIVE |
| MIL-03 | Milestone title required, min 5 characters | 400 error |
| MIL-04 | Budget percentage between 1 and 100 | 400 error |
| MIL-05 | Acceptance criteria required, min 20 characters | 400 error |
| MIL-06 | Total of all milestone percentages cannot exceed 100 | 400 TOTAL_EXCEEDS_100 |
| MIL-07 | Engineer can create multiple milestones | No limit |
| MIL-08 | Milestone status defaults to "pending" | Automatic |
| MIL-09 | Budget amount calculated as (percentage / 100) * project budget | Automatic |

### 5.2 Milestone Update Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| MIL-10 | Only engineer who created milestone can update | 403 Forbidden |
| MIL-11 | Cannot update milestone after client approval | 403 ALREADY_APPROVED |
| MIL-12 | Changing percentage cannot make total exceed 100 | 400 TOTAL_EXCEEDS_100 |

### 5.3 Client Approval Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| MIL-13 | Only client who owns project can approve | 403 Forbidden |
| MIL-14 | Total percentage must equal exactly 100 | 400 VALIDATION_FAILED |
| MIL-15 | Project must have at least 1 milestone | 400 NO_MILESTONES |
| MIL-16 | Cannot approve twice | 400 ALREADY_APPROVED |
| MIL-17 | After approval, all milestone status changes to "active" | Automatic |
| MIL-18 | After approval, project status changes from "draft" to "active" | Automatic |
| MIL-19 | After approval, engineer cannot edit or delete milestones | Blocked by API |

---

## 6. TESTING CHECKLIST WITH CHECKBOXES

### 6.1 Create Milestone Tests (10 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 1 | Engineer creates valid milestone | 201 Created, milestone in database | ☐ |
| 2 | Client tries to create milestone | 403 Forbidden | ☐ |
| 3 | Engineer not assigned to project tries to create | 403 Forbidden | ☐ |
| 4 | Create milestone with title too short (4 chars) | 400 error | ☐ |
| 5 | Create milestone with percentage 0 | 400 INVALID_PERCENTAGE | ☐ |
| 6 | Create milestone with percentage 101 | 400 INVALID_PERCENTAGE | ☐ |
| 7 | Create milestone without acceptance criteria | 400 MISSING_ACCEPTANCE_CRITERIA | ☐ |
| 8 | Create second milestone that makes total exceed 100 | 400 TOTAL_EXCEEDS_100 | ☐ |
| 9 | Create milestone for project that is already active | 403 PROJECT_ALREADY_ACTIVE | ☐ |
| 10 | Budget amount calculates correctly | 25% of 10M = 2.5M RWF | ☐ |

### 6.2 Total Validation Tests (5 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 11 | Milestones sum to 100 | Validation returns is_valid=true | ☐ |
| 12 | Milestones sum to 70 | Validation returns needs=30, can_approve=false | ☐ |
| 13 | Milestones sum to 105 | Validation returns exceeds=5, can_approve=false | ☐ |
| 14 | No milestones created | Validation returns needs=100 | ☐ |
| 15 | Single milestone with 100% | Validation returns is_valid=true | ☐ |

### 6.3 Client Approval Tests (8 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 16 | Client approves valid milestones (sum=100) | 200 OK, all statuses become active | ☐ |
| 17 | Client approves when sum not 100 | 400 VALIDATION_FAILED | ☐ |
| 18 | Client approves when no milestones exist | 400 NO_MILESTONES | ☐ |
| 19 | Client approves already approved milestones | 400 ALREADY_APPROVED | ☐ |
| 20 | Engineer tries to approve | 403 Forbidden | ☐ |
| 21 | Different client tries to approve | 403 Forbidden | ☐ |
| 22 | After approval, engineer cannot edit milestone | PUT returns 403 | ☐ |
| 23 | After approval, project status changes to active | Check projects table status column | ☐ |

### 6.4 Update and Delete Tests (6 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 24 | Engineer updates milestone before approval | 200 OK, changes saved | ☐ |
| 25 | Engineer updates milestone after approval | 403 ALREADY_APPROVED | ☐ |
| 26 | Update milestone percentage that causes exceed | 400 TOTAL_EXCEEDS_100 | ☐ |
| 27 | Engineer deletes milestone before approval | 200 OK, milestone removed | ☐ |
| 28 | Engineer deletes milestone after approval | 403 ALREADY_APPROVED | ☐ |
| 29 | Delete milestone and verify total recalculated | GET /validate shows new total | ☐ |

---

## 7. COMMON PROBLEMS & SOLUTIONS

### Problem 1: Engineer creates milestones that sum to 100 but client sees different total due to rounding

**Solution:** Store percentages as integers (1-100). Calculate budget amount as (percentage / 100) * total budget. Use integer math for RWF (no decimals). Example: 33% of 10,000,000 = 3,300,000, not 3,333,333.

**How to verify:** Create milestones 33%, 33%, 34% = 100%. Each amount is integer with no decimals.

### Problem 2: Client approves milestones but engineer already started working on unapproved milestones

**Solution:** Milestone status must be "active" before engineer can request payment. API blocks payment requests on pending milestones. Show visual indicator on engineer dashboard which milestones are active.

**How to verify:** Try to request payment on pending milestone. Get 403 error "Milestone not yet approved by client".

### Problem 3: Engineer creates 10 milestones but client only sees first 5 due to pagination

**Solution:** Show all milestones on one screen without pagination for MVP. Use FlatList with scroll. Maximum reasonable milestones per project is 10-15.

**How to verify:** Create 12 milestones, scroll through list, all visible.

### Problem 4: Client accidentally approves wrong milestone structure

**Solution:** Add confirmation dialog before approval. "Are you sure? Once approved, milestones cannot be changed by engineer." Also show total budget and each milestone amount.

**How to verify:** Tap approve, see dialog, must confirm before API call.

### Problem 5: Engineer deletes milestone that already has payment requests

**Solution:** Block deletion of milestones that have any transactions or payment requests. Check transactions table for milestone_id before allowing delete.

**How to verify:** Create milestone, request payment, try to delete. Get 403 "Cannot delete milestone with existing payment requests"

---

## 8. TIME TRACKING & TASK BREAKDOWN

### 8.1 Developer Tasks (Single Developer)

| Task # | Task Description | Hours | Completed? | Actual Hours |
|--------|------------------|-------|------------|--------------|
| 1 | Create milestones table migration (if not exists) | 0.5 | ☐ | |
| 2 | Add foreign key to projects table | 0.5 | ☐ | |
| 3 | Implement POST /milestones endpoint | 1.5 | ☐ | |
| 4 | Implement total percentage validation logic | 1 | ☐ | |
| 5 | Implement GET /milestones list endpoint | 1 | ☐ | |
| 6 | Implement GET /milestones/:id detail endpoint | 0.5 | ☐ | |
| 7 | Implement POST /milestones/validate endpoint | 1 | ☐ | |
| 8 | Implement POST /milestones/approve endpoint | 1.5 | ☐ | |
| 9 | Implement PUT /milestones/:id update endpoint | 1 | ☐ | |
| 10 | Implement DELETE /milestones/:id endpoint | 0.5 | ☐ | |
| 11 | Create MilestoneListScreen (React Native) | 2 | ☐ | |
| 12 | Create CreateMilestoneScreen (React Native) | 2 | ☐ | |
| 13 | Create ApproveMilestonesScreen (React Native) | 1.5 | ☐ | |
| 14 | Add milestone section to ProjectDetailScreen | 1 | ☐ | |
| 15 | Write API integration for all screens | 1.5 | ☐ | |
| 16 | Run all 29 test cases | 1.5 | ☐ | |
| 17 | Fix bugs found during testing | 1.5 | ☐ | |
| **Total** | | **19.5** | | |

### 8.2 Timeline (Single Developer)

| Day | Tasks | Hours |
|-----|-------|-------|
| Day 1 AM | Database + POST /milestones (Tasks 1-4) | 3.5 |
| Day 1 PM | GET endpoints + validate endpoint (Tasks 5-7) | 2.5 |
| Day 2 AM | Approve + update + delete endpoints (Tasks 8-10) | 3 |
| Day 2 PM | Mobile screens part 1 (Tasks 11-13) | 5.5 |
| Day 3 AM | Integration + testing (Tasks 14-17) | 4 |

### 8.3 Feature Completion Checklist

| # | Item | Status |
|---|------|--------|
| 1 | All 7 API endpoints working | ☐ |
| 2 | All 3 mobile screens built | ☐ |
| 3 | All 29 test cases passed | ☐ |
| 4 | Total percentage validation prevents sum ≠ 100 | ☐ |
| 5 | Only engineer assigned to project can create milestones | ☐ |
| 6 | Only client who owns project can approve | ☐ |
| 7 | After approval, engineer cannot edit or delete | ☐ |
| 8 | Project status changes to active after approval | ☐ |
| 9 | Budget amount calculates correctly from percentage | ☐ |
| 10 | Client can review all milestones before approving | ☐ |

---

## 9. PREREQUISITES (What Must Be Done First)

Before starting this feature, the following MUST be complete:

| Prerequisite | Status | Notes |
|--------------|--------|-------|
| Auth & KYC Feature | ☐ Pending | User must be able to register and login |
| Project Management Feature | ☐ Pending | Project must exist before creating milestones |
| Engineer assigned to project | ☐ Pending | Engineer must be invited and accept project |
| JWT authentication middleware | ☐ Pending | All milestone endpoints require JWT |
| projects table with engineer_id | ☐ Pending | To verify engineer assignment |

**Do NOT start this feature until Project Management is complete and tested.**

---

**END OF MILESTONE MANAGEMENT FEATURE DOCUMENTATION**

**Next Feature Ready When You Are:** Escrow & Payments (ESCROW-01, ESCROW-04, ESCROW-05, ESCROW-06, ESCROW-07, ESCROW-09)