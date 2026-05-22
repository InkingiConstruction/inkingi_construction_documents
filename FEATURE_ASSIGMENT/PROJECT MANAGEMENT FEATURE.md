# INKINGIPRO - PROJECT MANAGEMENT FEATURE DOCUMENTATION

**Feature:** Project Management (Create Project, View List, View Details)
**Feature IDs:** PROJ-01, PROJ-05, PROJ-06 from MVP list
**Status:** Ready for Development
**Estimated Time:** 12-16 Hours
**Database Tables Used:** projects, project_members, users

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

This feature allows a verified Client user to create and manage construction projects.

**PART A: Create Project (PROJ-01)**
- Client enters project name and budget amount in RWF
- Client adds project description and site address
- Client submits form to create project record
- System saves project with status "draft"

**PART B: View Project List (PROJ-05)**
- Client sees all projects they have created
- Engineer sees all projects they are assigned to
- Each project shows name, budget, status, and progress
- List is sorted by most recently updated

**PART C: View Project Details (PROJ-06)**
- User taps on a project to see full details
- Shows project name, budget, address, description
- Shows current escrow balance (from escrow_accounts table)
- Shows list of milestones with their status
- Shows client and engineer names

### 1.2 Why This Feature is MVP Priority

| Reason | Explanation |
|--------|-------------|
| Core Action | Client cannot start construction without creating a project |
| Transparency | Viewing project details builds trust with diaspora investors |
| Foundation | All other features (milestones, payments, inspections) depend on projects |

### 1.3 Sequential Flow Steps

**Create Project Flow:**
- Step 1: Client logs into app with verified account (kyc_status = approved)
- Step 2: Client taps "Create New Project" button on dashboard
- Step 3: Client enters project name (required, min 3 characters)
- Step 4: Client enters budget amount in RWF (required, minimum 100,000)
- Step 5: Client enters site address (required, text description)
- Step 6: Client enters description (optional, max 500 characters)
- Step 7: Client taps "Create Project" button
- Step 8: System validates all fields
- Step 9: System inserts record into projects table with status = draft
- Step 10: System creates escrow_account record for this project with balance = 0
- Step 11: System returns success and shows project details screen

**View Project List Flow:**
- Step 1: User logs into app
- Step 2: System queries projects table for user's role
- Step 3: If Client: returns all projects where client_id = user.id
- Step 4: If Engineer: returns all projects where engineer_id = user.id
- Step 5: System displays list with project cards
- Step 6: Each card shows name, budget, status, progress percentage

**View Project Details Flow:**
- Step 1: User taps on project from list
- Step 2: System fetches project by ID from projects table
- Step 3: System fetches escrow balance from escrow_accounts table
- Step 4: System fetches milestones for this project from milestones table
- Step 5: System calculates progress percentage from completed milestones
- Step 6: System displays all information on detail screen

---

## 2. DATABASE TABLES USED (Names Only)

These tables are already defined in your database schema file. This feature uses ONLY these tables:

| Table Name | Purpose In This Feature |
|------------|-------------------------|
| users | Check client role and KYC status before allowing project creation |
| projects | Store project name, budget, address, description, client_id, status |
| escrow_accounts | Store escrow balance for each project (created when project is created) |
| milestones | Read milestone data for progress calculation (created by Engineer feature later) |
| project_members | Check if user has access to view this project |

**Tables accessed (read only):** users, milestones, project_members

**Tables written to:** projects, escrow_accounts

---

## 3. COMPLETE API ENDPOINTS

### 3.1 Project Endpoints (6 Endpoints)

| Method | Endpoint | What It Does | Request Body | Response |
|--------|----------|--------------|--------------|----------|
| POST | /api/v1/projects | Creates new project | name, budget, address, description | project object with id |
| GET | /api/v1/projects | Lists user's projects | None (uses JWT user_id) | projects array |
| GET | /api/v1/projects/:id | Gets project details | None (project ID in URL) | project object with milestones and escrow |
| PUT | /api/v1/projects/:id | Updates project info | name, budget, address, description | updated project object |
| DELETE | /api/v1/projects/:id | Soft deletes project | None | success message |
| GET | /api/v1/projects/:id/progress | Gets progress percentage | None | progress percentage, completed milestones count |

### 3.2 Endpoint Details

**ENDPOINT 1: Create Project**

| Property | Value |
|----------|-------|
| Method | POST |
| URL | /api/v1/projects |
| Auth Required | Yes (JWT token) |
| Role Required | Client only |

**Request Body:**
```json
{
    "name": "Villa Nyarutarama Construction",
    "budget": 15000000,
    "address": "Nyarutarama, Kigali, Plot No. 123",
    "description": "3 bedroom luxury villa with swimming pool"
}
```

**Validation Rules:**
| Field | Rule |
|-------|------|
| name | Required, min 3 characters, max 100 characters |
| budget | Required, minimum 100,000 RWF, maximum 1,000,000,000 RWF |
| address | Required, min 5 characters, max 200 characters |
| description | Optional, max 500 characters |

**Success Response (201 Created):**
```json
{
    "success": true,
    "message": "Project created successfully",
    "data": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "name": "Villa Nyarutarama Construction",
        "budget": 15000000,
        "address": "Nyarutarama, Kigali, Plot No. 123",
        "description": "3 bedroom luxury villa with swimming pool",
        "client_id": "client-user-id-123",
        "status": "draft",
        "created_at": "2024-01-15T10:30:00Z",
        "escrow_balance": 0
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| UNAUTHORIZED | 401 | No JWT token provided |
| FORBIDDEN | 403 | User role is not Client |
| KYC_NOT_APPROVED | 403 | Client kyc_status is not approved |
| INVALID_BUDGET | 400 | Budget below 100,000 or above 1,000,000,000 |
| MISSING_FIELDS | 400 | Name, budget, or address missing |
| PROJECT_LIMIT_REACHED | 403 | Client has more than 10 active projects |

---

**ENDPOINT 2: List User Projects**

| Property | Value |
|----------|-------|
| Method | GET |
| URL | /api/v1/projects |
| Auth Required | Yes (JWT token) |
| Role Required | Client or Engineer or Supervisor or Supplier |

**Query Parameters (optional):**
| Parameter | Values | Default | Description |
|-----------|--------|---------|-------------|
| status | draft, active, completed, all | all | Filter by project status |
| page | number | 1 | Page number for pagination |
| limit | number | 20 | Items per page |
| sort | created_at, updated_at, name | updated_at | Sort field |
| order | asc, desc | desc | Sort order |

**Success Response (200 OK):**
```json
{
    "success": true,
    "data": {
        "projects": [
            {
                "id": "project-id-1",
                "name": "Villa Nyarutarama",
                "budget": 15000000,
                "address": "Nyarutarama, Kigali",
                "status": "active",
                "progress_percentage": 65,
                "escrow_balance": 5250000,
                "last_updated": "2024-01-15T10:30:00Z"
            },
            {
                "id": "project-id-2",
                "name": "Kigali Heights Office",
                "budget": 25000000,
                "address": "Kacyiru, Kigali",
                "status": "draft",
                "progress_percentage": 0,
                "escrow_balance": 0,
                "last_updated": "2024-01-14T08:00:00Z"
            }
        ],
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

**ENDPOINT 3: Get Project Details**

| Property | Value |
|----------|-------|
| Method | GET |
| URL | /api/v1/projects/:id |
| Auth Required | Yes (JWT token) |
| Role Required | Client (if owner) OR Engineer (if assigned) OR Admin |

**Success Response (200 OK):**
```json
{
    "success": true,
    "data": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "name": "Villa Nyarutarama Construction",
        "budget": 15000000,
        "address": "Nyarutarama, Kigali, Plot No. 123",
        "description": "3 bedroom luxury villa with swimming pool",
        "status": "active",
        "client": {
            "id": "client-id",
            "full_name": "Jean Paul Ndayishimiye",
            "email": "client@example.com"
        },
        "engineer": {
            "id": "engineer-id",
            "full_name": "Marie Uwase",
            "email": "engineer@example.com"
        },
        "escrow": {
            "balance": 5250000,
            "locked_balance": 0,
            "total_deposited": 8000000,
            "total_released": 2750000
        },
        "milestones": [
            {
                "id": "milestone-1",
                "title": "Foundation",
                "budget_percentage": 25,
                "budget_amount": 3750000,
                "status": "paid",
                "completed_at": "2024-01-10T00:00:00Z"
            },
            {
                "id": "milestone-2",
                "title": "Framing",
                "budget_percentage": 35,
                "budget_amount": 5250000,
                "status": "active",
                "completed_at": null
            }
        ],
        "created_at": "2024-01-01T00:00:00Z",
        "updated_at": "2024-01-15T10:30:00Z"
    }
}
```

**Error Responses:**
| Error | HTTP Status | When It Happens |
|-------|-------------|-----------------|
| PROJECT_NOT_FOUND | 404 | Project ID does not exist |
| ACCESS_DENIED | 403 | User is not client, engineer, or admin for this project |

---

**ENDPOINT 4: Update Project**

| Property | Value |
|----------|-------|
| Method | PUT |
| URL | /api/v1/projects/:id |
| Auth Required | Yes (JWT token) |
| Role Required | Client (owner only) |

**Request Body:** (all fields optional)
```json
{
    "name": "Updated Villa Name",
    "budget": 18000000,
    "address": "Updated address",
    "description": "Updated description"
}
```

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Project updated successfully",
    "data": {
        "id": "project-id",
        "name": "Updated Villa Name",
        "budget": 18000000,
        "updated_at": "2024-01-15T11:00:00Z"
    }
}
```

**Note:** Budget can only be increased, never decreased. Budget decrease requires admin approval.

---

**ENDPOINT 5: Delete Project (Soft Delete)**

| Property | Value |
|----------|-------|
| Method | DELETE |
| URL | /api/v1/projects/:id |
| Auth Required | Yes (JWT token) |
| Role Required | Client (owner only) |

**Success Response (200 OK):**
```json
{
    "success": true,
    "message": "Project deleted successfully"
}
```

**Rules:**
- Only projects with status "draft" can be deleted
- Projects with active milestones cannot be deleted
- Deleted projects set status = "deleted" (not actually removed from database)

---

## 4. FRONTEND SCREENS REQUIRED

### 4.1 React Native Mobile App Screens (3 Screens)

| Screen Name | File Location | What User Does On This Screen |
|-------------|---------------|-------------------------------|
| ProjectListScreen | screens/project/ProjectListScreen.tsx | Views all projects, taps to see details, taps create new button |
| CreateProjectScreen | screens/project/CreateProjectScreen.tsx | Enters name, budget, address, description, submits form |
| ProjectDetailScreen | screens/project/ProjectDetailScreen.tsx | Views full project info, milestones, escrow balance |

### 4.2 Screen Details

**Screen 1: ProjectListScreen**

| Element | Description |
|---------|-------------|
| Header | Title "My Projects" + "+" button to create new |
| Empty State | Shows "No projects yet. Tap + to create your first project" |
| Project Card | Shows project name, budget formatted as RWF, status badge, progress bar |
| Status Badge Colors | Draft = Gray, Active = Green, Completed = Blue, Terminated = Red |
| Pull to Refresh | User can pull down to reload list |
| Search Bar | Filter projects by name |

**Screen 2: CreateProjectScreen**

| Field | Type | Validation |
|-------|------|------------|
| Project Name | Text input | Required, min 3 chars |
| Budget | Number input | Required, min 100,000, formatted as RWF |
| Site Address | Text input | Required, min 5 chars |
| Description | Text area | Optional, max 500 chars |
| Submit Button | Button | Disabled until all required fields filled |

**Screen 3: ProjectDetailScreen**

| Section | Content |
|---------|---------|
| Header | Project name, status badge, edit icon (if owner) |
| Info Card | Budget (formatted), address, description |
| Escrow Card | Balance, total deposited, total released |
| Milestones Section | List of milestones with progress bars |
| Actions | Invite Engineer button (if owner and no engineer assigned) |

---

## 5. BUSINESS RULES & VALIDATION

### 5.1 Project Creation Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| PROJ-01 | User must be logged in with valid JWT | 401 Unauthorized |
| PROJ-02 | User role must be "client" | 403 Forbidden |
| PROJ-03 | Client KYC status must be "approved" | 403 KYC_NOT_APPROVED |
| PROJ-04 | Project name must be at least 3 characters | 400 error |
| PROJ-05 | Project name must be at most 100 characters | 400 error |
| PROJ-06 | Budget must be at least 100,000 RWF | 400 error |
| PROJ-07 | Budget must be at most 1,000,000,000 RWF | 400 error |
| PROJ-08 | Address must be at least 5 characters | 400 error |
| PROJ-09 | Address must be at most 200 characters | 400 error |
| PROJ-10 | Description optional but max 500 characters | 400 error if exceeded |
| PROJ-11 | Client cannot have more than 10 active projects | 403 PROJECT_LIMIT_REACHED |
| PROJ-12 | When project created, escrow_account record auto-created | Automatic |
| PROJ-13 | New project status set to "draft" | Automatic |

### 5.2 Project View Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| PROJ-14 | User must be logged in | 401 Unauthorized |
| PROJ-15 | User can only see projects they own or are assigned to | Projects filtered by user_id |
| PROJ-16 | Admin can see ALL projects | No filter for admin |
| PROJ-17 | Project list sorted by updated_at desc (most recent first) | Automatic |
| PROJ-18 | Progress percentage calculated as sum of paid milestone budgets / total budget * 100 | Automatic |

### 5.3 Project Update Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| PROJ-19 | Only client who created project can update | 403 Forbidden |
| PROJ-20 | Budget can only be increased, never decreased | 400 BUDGET_CANNOT_DECREASE |
| PROJ-21 | Status can be changed from draft to active only after deposit | 400 INVALID_STATUS_TRANSITION |
| PROJ-22 | Cannot change name/budget/address if project has active milestones | 403 PROJECT_HAS_ACTIVE_MILESTONES |

### 5.4 Project Delete Rules

| Rule Number | Rule Description | What Happens If Violated |
|-------------|------------------|--------------------------|
| PROJ-23 | Only draft projects can be deleted | 403 CANNOT_DELETE_ACTIVE_PROJECT |
| PROJ-24 | Delete is soft delete (status = deleted, not removed) | Database record remains |
| PROJ-25 | Escrow_account is also marked inactive | Not deleted for audit |

---

## 6. TESTING CHECKLIST WITH CHECKBOXES

### 6.1 Create Project Tests (10 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 1 | Client with approved KYC creates valid project | 201 Created, project saved to database | ☐ |
| 2 | Client without KYC approval tries to create project | 403 KYC_NOT_APPROVED | ☐ |
| 3 | Engineer tries to create project | 403 Forbidden | ☐ |
| 4 | Create project with name too short (2 chars) | 400 error | ☐ |
| 5 | Create project with budget below 100,000 | 400 INVALID_BUDGET | ☐ |
| 6 | Create project with missing address | 400 MISSING_FIELDS | ☐ |
| 7 | Create project with no JWT token | 401 Unauthorized | ☐ |
| 8 | Escrow account created automatically | Check escrow_accounts table has record | ☐ |
| 9 | Project status defaults to "draft" | Check projects table status column | ☐ |
| 10 | Client with 10 active projects tries to create 11th | 403 PROJECT_LIMIT_REACHED | ☐ |

### 6.2 View Project List Tests (6 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 11 | Client views their projects | Returns only projects where client_id matches | ☐ |
| 12 | Engineer views assigned projects | Returns only projects where engineer_id matches | ☐ |
| 13 | Admin views all projects | Returns all projects in system | ☐ |
| 14 | Empty list when user has no projects | Returns empty array, shows empty state | ☐ |
| 15 | Pagination works with page and limit | Returns correct slice of data | ☐ |
| 16 | Sorting by updated_at desc works | Most recently updated appears first | ☐ |

### 6.3 View Project Details Tests (7 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 17 | Client views own project | Full project details returned | ☐ |
| 18 | Engineer views assigned project | Full project details returned | ☐ |
| 19 | User views project they don't own or aren't assigned to | 403 ACCESS_DENIED | ☐ |
| 20 | View non-existent project ID | 404 PROJECT_NOT_FOUND | ☐ |
| 21 | Escrow balance displays correctly | Matches escrow_accounts table | ☐ |
| 22 | Milestones list displays for project | Returns milestones from milestones table | ☐ |
| 23 | Progress percentage calculated correctly | Sum of paid milestone budgets / total budget | ☐ |

### 6.4 Update Project Tests (5 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 24 | Client updates project name | 200 OK, name changed in database | ☐ |
| 25 | Client updates budget to higher amount | 200 OK, budget increased | ☐ |
| 26 | Client updates budget to lower amount | 400 BUDGET_CANNOT_DECREASE | ☐ |
| 27 | Engineer tries to update project | 403 Forbidden | ☐ |
| 28 | Update project with active milestones | 403 PROJECT_HAS_ACTIVE_MILESTONES | ☐ |

### 6.5 Delete Project Tests (4 Tests)

| # | Test Case | Expected Result | Done? |
|---|-----------|-----------------|-------|
| 29 | Client deletes draft project | 200 OK, status changed to deleted | ☐ |
| 30 | Client deletes active project | 403 CANNOT_DELETE_ACTIVE_PROJECT | ☐ |
| 31 | Engineer tries to delete project | 403 Forbidden | ☐ |
| 32 | Deleted project no longer appears in list | GET /projects does not include deleted | ☐ |

---

## 7. COMMON PROBLEMS & SOLUTIONS

### Problem 1: Client creates project but KYC not approved

**Solution:** Check kyc_status in users table before allowing project creation. Return 403 with message "Please complete KYC verification before creating projects"

**How to verify:** Try to create project with client who has kyc_status = pending. Should see error.

### Problem 2: Escrow account not created when project created

**Solution:** Use database transaction. Create project and escrow_account in same transaction. If either fails, rollback both.

**How to verify:** Check escrow_accounts table after each project creation. Should always have matching record.

### Problem 3: Engineer sees projects they are not assigned to

**Solution:** In GET /projects endpoint, filter by engineer_id when role = engineer. Never return all projects.

**How to verify:** Login as Engineer, check projects list. Should only show projects where engineer_id matches.

### Problem 4: Budget display confusing for diaspora investors (USD vs RWF)

**Solution:** Store budget in RWF in database. On client app, show both RWF and approximate USD using saved exchange rate. Update exchange rate daily via background job.

**How to verify:** Client sees "15,000,000 RWF (approx $11,500 USD)"

### Problem 5: Progress percentage calculation wrong

**Solution:** Progress = (Sum of budget for milestones with status 'paid') / (Total project budget) * 100. Only count milestones that are fully paid.

**How to verify:** Create project with 2 milestones: 30% and 70%. Pay 30% milestone. Progress should show 30%, not 100%.

### Problem 6: Duplicate project names for same client

**Solution:** No restriction on duplicate names. Client can have "Villa A" and "Villa A" but show created_at dates to differentiate.

**How to verify:** Client creates two projects with same name. Both saved successfully.

---

## 8. TIME TRACKING & TASK BREAKDOWN

### 8.1 Developer Tasks (Single Developer)

| Task # | Task Description | Hours | Completed? | Actual Hours |
|--------|------------------|-------|------------|--------------|
| 1 | Create projects table migration (if not exists) | 0.5 | ☐ | |
| 2 | Add client_id foreign key to projects | 0.5 | ☐ | |
| 3 | Implement POST /projects endpoint | 2 | ☐ | |
| 4 | Implement project creation validation rules | 1 | ☐ | |
| 5 | Implement automatic escrow_account creation | 1 | ☐ | |
| 6 | Implement GET /projects list endpoint | 1.5 | ☐ | |
| 7 | Implement role-based filtering for list | 1 | ☐ | |
| 8 | Implement GET /projects/:id detail endpoint | 1.5 | ☐ | |
| 9 | Implement progress percentage calculation | 1 | ☐ | |
| 10 | Implement PUT /projects/:id update endpoint | 1.5 | ☐ | |
| 11 | Implement DELETE /projects/:id soft delete | 1 | ☐ | |
| 12 | Create ProjectListScreen (React Native) | 2 | ☐ | |
| 13 | Create CreateProjectScreen (React Native) | 2 | ☐ | |
| 14 | Create ProjectDetailScreen (React Native) | 2.5 | ☐ | |
| 15 | Write API integration for all screens | 2 | ☐ | |
| 16 | Run all 32 test cases | 2 | ☐ | |
| 17 | Fix bugs found during testing | 2 | ☐ | |
| **Total** | | **24** | | |

### 8.2 Timeline (Single Developer)

| Day | Tasks | Hours |
|-----|-------|-------|
| Day 1 | Database setup + POST /projects endpoint (Tasks 1-5) | 5 |
| Day 1 | GET /projects list + filtering (Tasks 6-7) | 2.5 |
| Day 2 | GET /projects/:id detail (Tasks 8-9) | 2.5 |
| Day 2 | PUT + DELETE endpoints (Tasks 10-11) | 2.5 |
| Day 3 | React Native screens (Tasks 12-15) | 8.5 |
| Day 4 | Testing + bug fixes (Tasks 16-17) | 4 |

### 8.3 Feature Completion Checklist

| # | Item | Status |
|---|------|--------|
| 1 | All 6 API endpoints working | ☐ |
| 2 | All 3 mobile screens built | ☐ |
| 3 | All 32 test cases passed | ☐ |
| 4 | Escrow account created automatically with each project | ☐ |
| 5 | Role-based filtering works (Client sees own, Engineer sees assigned) | ☐ |
| 6 | Progress percentage calculates correctly | ☐ |
| 7 | Cannot delete active projects | ☐ |
| 8 | Budget cannot be decreased after creation | ☐ |
| 9 | Deployed to staging | ☐ |
| 10 | Client can create project → view list → view details → update → delete | ☐ |

---

## 9. PREREQUISITES (What Must Be Done First)

Before starting this feature, the following MUST be complete:

| Prerequisite | Status | Notes |
|--------------|--------|-------|
| Auth & KYC Feature | ☐ Pending | User must be able to register, login, and get KYC approved |
| JWT authentication middleware | ☐ Pending | All project endpoints require JWT |
| Role-based access control | ☐ Pending | Must identify if user is Client or Engineer |
| users table with kyc_status | ☐ Pending | To check KYC approval before project creation |
| escrow_accounts table | ☐ Pending | Must exist before project creation (auto-create) |

**Do NOT start this feature until Auth & KYC is complete and tested.**

---

**END OF PROJECT MANAGEMENT FEATURE DOCUMENTATION**

**Next Feature Ready When You Are:** Milestone Management (MIL-01, MIL-02, MIL-04)