# API Design

## Overview

The API layer combines Supabase's auto-generated REST API (PostgREST) for standard CRUD operations with custom Edge Functions for business logic that requires validation or orchestration.

---

## API Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Client (React PWA)                     │
└─────────────────────┬───────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌──────────────┐ ┌──────────┐ ┌──────────────┐
│ PostgREST    │ │ Realtime │ │ Edge         │
│ (Auto CRUD)  │ │ (WS)    │ │ Functions    │
└──────────────┘ └──────────┘ └──────────────┘
        │             │             │
        └─────────────┼─────────────┘
                      ▼
            ┌──────────────────┐
            │   PostgreSQL DB   │
            └──────────────────┘
```

---

## Authentication

All API requests require a valid JWT token in the `Authorization` header:

```
Authorization: Bearer <supabase_access_token>
```

### Auth Endpoints (Supabase Auth)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/v1/signup` | Register new user |
| POST | `/auth/v1/token?grant_type=password` | Login with email/password |
| POST | `/auth/v1/token?grant_type=refresh_token` | Refresh access token |
| POST | `/auth/v1/logout` | Logout (invalidate session) |
| POST | `/auth/v1/recover` | Send password reset email |
| PUT | `/auth/v1/user` | Update user (password change) |

---

## REST API Endpoints (PostgREST)

Base URL: `https://<project>.supabase.co/rest/v1`

### Profiles

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/profiles?id=eq.{id}` | Self/Manager/Admin | Get profile by ID |
| GET | `/profiles?organization_id=eq.{org_id}` | Admin | List org profiles |
| PATCH | `/profiles?id=eq.{id}` | Self/Admin | Update profile |
| GET | `/profiles?role=eq.employee&is_active=eq.true` | Manager/Admin | List active employees |

### Sites

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/sites?organization_id=eq.{org_id}` | All | List sites |
| GET | `/sites?id=eq.{id}` | All | Get site details |
| POST | `/sites` | Admin | Create new site |
| PATCH | `/sites?id=eq.{id}` | Admin | Update site |
| DELETE | `/sites?id=eq.{id}` | Admin | Deactivate site |

### Time Entries

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/time_entries?profile_id=eq.{id}` | Self | Get own entries |
| GET | `/time_entries?site_id=eq.{id}` | Manager | Get site entries |
| GET | `/time_entries?clock_out=is.null` | Manager/Admin | Get active sessions |
| POST | `/time_entries` | Employee | Create entry (via Edge Function preferred) |
| PATCH | `/time_entries?id=eq.{id}` | Self/Manager | Update entry |

### Timesheets

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/timesheets?profile_id=eq.{id}` | Self | Get own timesheets |
| GET | `/timesheets?site_id=eq.{id}&status=eq.pending` | Manager | Get pending approvals |
| POST | `/timesheets` | Employee | Create timesheet |
| PATCH | `/timesheets?id=eq.{id}` | Self/Manager | Update status |

### Notifications

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/notifications?profile_id=eq.{id}&is_read=eq.false` | Self | Get unread |
| PATCH | `/notifications?id=eq.{id}` | Self | Mark as read |

---

## Edge Functions (Custom Business Logic)

Base URL: `https://<project>.supabase.co/functions/v1`

### `clock-in`

Validates geofence and QR code, then creates a time entry.

```typescript
// POST /functions/v1/clock-in
// Request
{
  "site_id": "uuid",
  "latitude": -33.8688,
  "longitude": 151.2093,
  "qr_token": "optional-string",  // if QR method
  "method": "gps" | "qr_code"
}

// Response 200
{
  "success": true,
  "time_entry_id": "uuid",
  "clock_in_time": "2026-06-09T09:00:00Z",
  "within_geofence": true
}

// Response 403
{
  "success": false,
  "error": "outside_geofence",
  "message": "You are not within the site boundary",
  "distance_meters": 245
}
```

### `clock-out`

Validates and closes an active time entry.

```typescript
// POST /functions/v1/clock-out
// Request
{
  "time_entry_id": "uuid",
  "latitude": -33.8688,
  "longitude": 151.2093,
  "method": "gps" | "qr_code"
}

// Response 200
{
  "success": true,
  "time_entry_id": "uuid",
  "clock_out_time": "2026-06-09T17:00:00Z",
  "total_hours": 8.0,
  "within_geofence": true
}
```

### `submit-timesheet`

Calculates totals and submits timesheet for approval.

```typescript
// POST /functions/v1/submit-timesheet
// Request
{
  "site_id": "uuid",
  "period_start": "2026-06-01",
  "period_end": "2026-06-07"
}

// Response 200
{
  "success": true,
  "timesheet_id": "uuid",
  "total_hours": 38.5,
  "overtime_hours": 0,
  "entries_count": 5,
  "status": "pending"
}
```

### `approve-timesheet`

Manager approves or rejects a timesheet.

```typescript
// POST /functions/v1/approve-timesheet
// Request
{
  "timesheet_id": "uuid",
  "action": "approve" | "reject",
  "rejection_reason": "optional string"  // required if rejecting
}

// Response 200
{
  "success": true,
  "timesheet_id": "uuid",
  "status": "approved",
  "approved_at": "2026-06-09T14:30:00Z"
}
```

### `bulk-approve-timesheets`

Manager approves multiple timesheets at once.

```typescript
// POST /functions/v1/bulk-approve-timesheets
// Request
{
  "timesheet_ids": ["uuid1", "uuid2", "uuid3"],
  "action": "approve"
}

// Response 200
{
  "success": true,
  "approved_count": 3,
  "failed_count": 0,
  "results": [
    { "id": "uuid1", "status": "approved" },
    { "id": "uuid2", "status": "approved" },
    { "id": "uuid3", "status": "approved" }
  ]
}
```

### `generate-qr-code`

Generates a new QR code for a site.

```typescript
// POST /functions/v1/generate-qr-code
// Request
{
  "site_id": "uuid",
  "expires_in_days": 30  // optional, null = never expires
}

// Response 200
{
  "success": true,
  "qr_code_id": "uuid",
  "token": "hex-string",
  "image_url": "https://storage.supabase.co/...",
  "expires_at": "2026-07-09T00:00:00Z"
}
```

### `validate-qr-token`

Validates a scanned QR code token.

```typescript
// POST /functions/v1/validate-qr-token
// Request
{
  "token": "hex-string"
}

// Response 200
{
  "valid": true,
  "site_id": "uuid",
  "site_name": "Tyrepower Sydney CBD"
}
```

### `get-dashboard-stats`

Aggregated statistics for manager/admin dashboards.

```typescript
// GET /functions/v1/get-dashboard-stats?site_id={id}&period=today|week|month
// Response 200
{
  "currently_clocked_in": 12,
  "total_employees": 45,
  "total_hours_today": 96.5,
  "pending_timesheets": 3,
  "late_arrivals_today": 2,
  "overtime_alerts": 1,
  "attendance_rate": 0.93
}
```

### `send-notification`

Internal function for sending push/email notifications.

```typescript
// POST /functions/v1/send-notification (internal only)
// Request
{
  "profile_id": "uuid",
  "title": "Timesheet Approved",
  "body": "Your timesheet for Jun 1-7 has been approved",
  "type": "timesheet_approved",
  "channels": ["push", "email", "in_app"]
}
```

---

## Realtime Subscriptions

### Channel Structure

```typescript
// Manager subscribes to site activity
supabase
  .channel('site-activity')
  .on('postgres_changes', {
    event: 'INSERT',
    schema: 'public',
    table: 'time_entries',
    filter: `site_id=eq.${siteId}`
  }, (payload) => {
    // Handle new clock-in at this site
  })
  .subscribe();

// Employee subscribes to their notifications
supabase
  .channel('user-notifications')
  .on('postgres_changes', {
    event: 'INSERT',
    schema: 'public',
    table: 'notifications',
    filter: `profile_id=eq.${profileId}`
  }, (payload) => {
    // Show notification toast
  })
  .subscribe();

// Admin subscribes to pending timesheets
supabase
  .channel('pending-timesheets')
  .on('postgres_changes', {
    event: 'UPDATE',
    schema: 'public',
    table: 'timesheets',
    filter: `status=eq.pending`
  }, (payload) => {
    // Update pending count badge
  })
  .subscribe();
```

---

## Error Response Format

All Edge Functions return consistent error responses:

```typescript
// Standard error format
{
  "success": false,
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {}  // optional additional context
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `unauthorized` | 401 | Missing or invalid JWT |
| `forbidden` | 403 | Insufficient role/permissions |
| `not_found` | 404 | Resource doesn't exist |
| `outside_geofence` | 403 | Employee not within site boundary |
| `invalid_qr_token` | 400 | QR code expired or invalid |
| `already_clocked_in` | 409 | Employee has active session |
| `not_clocked_in` | 409 | No active session to clock out |
| `timesheet_conflict` | 409 | Overlapping timesheet period |
| `validation_error` | 400 | Invalid request body |
| `rate_limited` | 429 | Too many requests |

---

## Rate Limiting

| Endpoint | Limit | Window |
|----------|-------|--------|
| Auth endpoints | 30 requests | per minute |
| Clock-in/out | 10 requests | per minute per user |
| General API | 100 requests | per minute per user |
| Dashboard stats | 30 requests | per minute |

---

## Pagination

PostgREST supports range-based pagination:

```
GET /rest/v1/time_entries?profile_id=eq.{id}&order=clock_in.desc
Range: 0-24
```

Response headers include:
```
Content-Range: 0-24/150
```

---

## Filtering & Sorting

PostgREST operators used:

| Operator | Example | Description |
|----------|---------|-------------|
| `eq` | `?status=eq.pending` | Equals |
| `neq` | `?status=neq.draft` | Not equals |
| `gt` | `?clock_in=gt.2026-06-01` | Greater than |
| `gte` | `?total_hours=gte.8` | Greater than or equal |
| `lt` | `?clock_in=lt.2026-06-08` | Less than |
| `is` | `?clock_out=is.null` | IS NULL |
| `in` | `?status=in.(pending,approved)` | IN list |
| `order` | `?order=clock_in.desc` | Sort |
| `select` | `?select=id,clock_in,profiles(first_name)` | Field selection + joins |

---

## Additional REST Endpoints

### Invitations

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/invitations?site_id=eq.{id}` | Manager | List pending invitations |
| POST | `/invitations` | Manager/Admin | Create invitation (via Edge Function) |
| DELETE | `/invitations?id=eq.{id}` | Manager | Cancel pending invitation |

### Announcements

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/announcements?site_id=eq.{id}&is_active=eq.true` | All (authenticated) | List active announcements |
| POST | `/announcements` | Manager | Create announcement |
| PATCH | `/announcements?id=eq.{id}` | Manager | Update/deactivate announcement |
| DELETE | `/announcements?id=eq.{id}` | Manager | Delete announcement |

### Audit Logs

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/audit_logs?order=created_at.desc` | Admin | Browse all audit logs |
| GET | `/audit_logs?user_id=eq.{id}` | Admin/Self | Logs for specific user |

### Rosters (Phase 4)

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/rosters?site_id=eq.{id}` | Manager/Employee | List rosters for site |
| POST | `/rosters` | Manager | Create new roster |
| PATCH | `/rosters?id=eq.{id}` | Manager | Update roster status |
| GET | `/roster_shifts?roster_id=eq.{id}` | Manager/Employee | Get shifts in roster |
| POST | `/roster_shifts` | Manager | Add shift to roster |
| PATCH | `/roster_shifts?id=eq.{id}` | Manager | Update shift |
| DELETE | `/roster_shifts?id=eq.{id}` | Manager | Remove shift |

---

## Additional Edge Functions

### `invite-employee`

Send an invitation email to a new employee.

```typescript
// POST /functions/v1/invite-employee
// Request
{
  "site_id": "uuid",
  "email": "employee@example.com",
  "role": "employee"  // or "site_manager"
}

// Response 200
{
  "success": true,
  "invitation_id": "uuid",
  "expires_at": "2026-06-16T00:00:00Z"
}
```

### `accept-invitation`

Employee accepts invitation and creates their account.

```typescript
// POST /functions/v1/accept-invitation
// Request
{
  "token": "hex-invitation-token",
  "password": "new-password",
  "first_name": "John",
  "last_name": "Smith",
  "phone": "0412345678"
}

// Response 200
{
  "success": true,
  "profile_id": "uuid",
  "site_name": "Tyrepower Sydney CBD",
  "message": "Account created. Please log in."
}

// Response 400
{
  "success": false,
  "error": "invitation_expired",
  "message": "This invitation has expired. Ask your manager for a new one."
}
```

### `suspend-employee`

Manager suspends an employee (cannot log in).

```typescript
// POST /functions/v1/suspend-employee
// Request
{
  "profile_id": "uuid",
  "reason": "Extended leave"
}

// Response 200
{
  "success": true,
  "status": "suspended"
}
```

### `reactivate-employee`

Manager reactivates a suspended employee.

```typescript
// POST /functions/v1/reactivate-employee
// Request
{
  "profile_id": "uuid"
}

// Response 200
{
  "success": true,
  "status": "active"
}
```

### `get-overtime-alerts`

Get employees approaching or exceeding overtime threshold.

```typescript
// GET /functions/v1/get-overtime-alerts?site_id={id}
// Response 200
{
  "threshold_hours": 38,
  "alerts": [
    {
      "profile_id": "uuid",
      "name": "Jane Doe",
      "weekly_hours": 42.5,
      "overtime_hours": 4.5,
      "status": "overtime"
    },
    {
      "profile_id": "uuid",
      "name": "Bob Smith",
      "weekly_hours": 36.5,
      "overtime_hours": 0,
      "status": "at_risk"
    }
  ]
}
```

### `generate-report`

Generate a report with filters and return data for export.

```typescript
// POST /functions/v1/generate-report
// Request
{
  "report_type": "employee_hours" | "timesheet_status" | "attendance" | "overtime" | "labour_utilisation" | "daily_summary",
  "site_id": "uuid",           // optional, omit for all sites
  "start_date": "2026-06-01",
  "end_date": "2026-06-07",
  "employee_id": "uuid",       // optional filter
  "format": "json"             // json for display, csv/xlsx/pdf for download
}

// Response 200
{
  "success": true,
  "report_type": "employee_hours",
  "period": { "start": "2026-06-01", "end": "2026-06-07" },
  "data": [...],
  "summary": {
    "total_hours": 1250.5,
    "total_employees": 35,
    "overtime_hours": 45.0
  }
}
```

### `publish-roster` (Phase 4)

Publish a roster and notify all assigned employees.

```typescript
// POST /functions/v1/publish-roster
// Request
{
  "roster_id": "uuid"
}

// Response 200
{
  "success": true,
  "roster_id": "uuid",
  "published_at": "2026-06-09T10:00:00Z",
  "employees_notified": 12
}
```

### `request-shift-swap` (Phase 4)

Employee requests to swap a shift.

```typescript
// POST /functions/v1/request-shift-swap
// Request
{
  "roster_shift_id": "uuid",
  "target_employee_id": "uuid",  // optional - specific person or open request
  "reason": "Medical appointment"
}

// Response 200
{
  "success": true,
  "swap_request_id": "uuid",
  "status": "pending"
}
```

---

## Site Landing Page API

The authenticated site landing page (`/s/:siteSlug`) uses standard PostgREST queries:

```typescript
// Get site info by slug (requires auth)
GET /rest/v1/sites?slug=eq.{siteSlug}&select=*

// Get active announcements for site
GET /rest/v1/announcements?site_id=eq.{id}&is_active=eq.true&order=created_at.desc

// Get current user's active shift at this site
GET /rest/v1/time_entries?profile_id=eq.{userId}&site_id=eq.{siteId}&clock_out=is.null
```
