# Feature Specifications

## Overview

This document defines all features of the Tyrepower Workforce Management Platform, organized by module. Each feature includes user stories, acceptance criteria, and priority level.

---

## Priority Legend

| Priority | Label | Phase | Description |
|----------|-------|-------|-------------|
| P0 | Critical | Phase 1 (MVP) | Must have for initial testing |
| P1 | High | Phase 2 (Production) | Required for production deployment |
| P2 | Medium | Phase 3 (Enhancement) | Important for multi-site scaling |
| P3 | Low | Phase 4 (Rostering/AI) | Stretch goal / future |

---

## Module 1: Authentication & Security

### F1.1 - User Login (P0)

**User Story:** As a user, I want to log in securely so I can access my role-appropriate dashboard.

**Acceptance Criteria:**
- Email/password authentication
- JWT token stored securely
- Auto-refresh of expired tokens
- Redirect to role-appropriate dashboard after login
- Large touch-friendly login form (mobile-first)
- Tyrepower branded login page

### F1.2 - Password Reset (P0)

**User Story:** As a user, I want to reset my password if I forget it.

**Acceptance Criteria:**
- "Forgot Password" link on login page
- Email sent with secure reset link
- Link expires after 1 hour
- Password requirements enforced (min 8 chars)
- Reset confirmation screen

### F1.3 - Password Change (P1)

**User Story:** As a user, I want to change my password from my profile.

**Acceptance Criteria:**
- Requires current password
- New password must meet requirements
- Confirmation of success
- Audit log entry created

### F1.4 - Employee Invitation Onboarding (P1)

**User Story:** As an employee, I want to accept an invitation and create my account.

**Acceptance Criteria:**
- Site Manager sends invitation via email
- Employee receives branded invitation email
- Employee clicks link → lands on account creation page
- Employee creates password
- Account activated, linked to correct site
- Invitation token expires after configurable period (default 7 days)
- Used invitations cannot be reused

### F1.5 - OTP Verification (P2)

**User Story:** As a Super Admin/Manager, I want optional OTP for extra security.

**Acceptance Criteria:**
- Optional two-factor via email OTP
- Configurable per role (Admins/Managers only)
- Bypass for trusted devices (optional)

### F1.6 - Session Management (P1)

**User Story:** As an admin, I want session controls to prevent unauthorized access.

**Acceptance Criteria:**
- Configurable session timeout
- Auto-logout after inactivity
- Active sessions visible to user
- Force-logout capability for admins

### F1.7 - Login Activity History (P1)

**User Story:** As a user, I want to see my recent login activity.

**Acceptance Criteria:**
- Shows last 10 logins
- Displays: date/time, device/browser, IP (masked), location (approximate)
- Highlight suspicious activity

### F1.8 - Device Tracking (P1)

**User Story:** As an admin, I want to know what devices users log in from.

**Acceptance Criteria:**
- Capture user agent on login
- Store device type (mobile/tablet/desktop)
- Browser name and version
- Accessible in audit logs

---

## Module 2: Site Management

### F2.1 - Create Site (P0)

**User Story:** As a Super Admin, I want to add new Tyrepower locations to the system.

**Acceptance Criteria:**
- Input: site name, site code, address, GPS coordinates, geofence radius
- Map picker for setting coordinates
- Upload site logo
- Set welcome message
- Default geofence radius: 100 meters
- Site appears on admin map view
- Active/Inactive status toggle

### F2.2 - Configure Geofence (P0)

**User Story:** As a Super Admin, I want to set the geofence boundary for each site.

**Acceptance Criteria:**
- Set radius (meters) around site center point
- Visual preview on map showing boundary circle
- Can adjust radius without recreating site
- Minimum radius: 25 meters
- Maximum radius: 500 meters

### F2.3 - Edit & Archive Sites (P0)

**User Story:** As a Super Admin, I want to edit site details or archive inactive sites.

**Acceptance Criteria:**
- Edit all site fields (name, code, logo, address, contact, welcome message)
- Archive site (soft-delete, preserves history)
- Archived sites hidden from active lists
- Can reactivate archived sites

### F2.4 - Assign Site Managers (P1)

**User Story:** As a Super Admin, I want to assign managers to sites, including multiple sites per manager.

**Acceptance Criteria:**
- Assign one or more managers per site
- A single manager can be assigned to multiple sites
- Managers only see data for their assigned sites
- Can reassign managers between sites
- Manager receives notification of assignment

### F2.5 - Invite Site Managers (P1)

**User Story:** As a Super Admin, I want to invite new Site Managers.

**Acceptance Criteria:**
- Send invitation email to new manager
- Manager creates account via invitation link
- Automatically assigned Site Manager role
- Can assign to sites during or after invitation

### F2.6 - Public Site Landing Page (P0)

**User Story:** As an employee, I want to access my site's page to clock in and see announcements.

**Acceptance Criteria:**
- Route: `/s/:siteSlug`
- **Requires authentication** — redirects to login if not signed in
- Displays: site logo, welcome message, announcements, contact details
- Employee features on page: Clock In, Clock Out, current shift status, today's hours
- Manager-editable content (logo, welcome message, announcements, contact)
- Mobile-optimized layout with large clock-in button

### F2.7 - Announcements (P2)

**User Story:** As a Site Manager, I want to post announcements for my site's employees.

**Acceptance Criteria:**
- Create/edit/delete announcements
- Display on site landing page
- Display on employee dashboard
- Support simple text with optional urgency level
- Most recent announcements shown first

---

## Module 3: Clock-In / Clock-Out

### F3.1 - GPS Clock-In (P0)

**User Story:** As an employee, I want to clock in using my phone's GPS so my location is verified.

**Acceptance Criteria:**
- Large single-tap "Clock In" button (fast access)
- Browser requests GPS permission
- GPS coordinates captured and sent to API
- Server validates coordinates against site geofence
- Success: shift entry created, confirmation shown with time
- Failure: clear error with distance from site shown
- Timestamp recorded in site's timezone

### F3.2 - QR Code Clock-In (P0)

**User Story:** As an employee, I want to scan a QR code at the site to clock in.

**Acceptance Criteria:**
- Camera opens QR scanner
- Scans site-specific QR code
- Validates QR token is active and matches a site
- GPS still captured for verification record
- Creates shift entry on successful scan
- Clear error if QR code is expired/invalid

### F3.3 - Clock-Out (P0)

**User Story:** As an employee, I want to clock out when I finish work.

**Acceptance Criteria:**
- "Clock Out" button visible when clocked in
- GPS captured on clock-out
- Geofence validation on clock-out (warning if outside, but still allows clock-out)
- Duration calculated and displayed
- Confirmation with total hours worked shown

### F3.4 - Active Shift Display (P0)

**User Story:** As an employee, I want to see my current active shift with a live timer.

**Acceptance Criteria:**
- Shows "Currently clocked in at [Site Name]"
- Displays clock-in time
- **Live timer counting up** (updates every second)
- Shows current hours worked today
- Clear "Clock Out" button

### F3.5 - Shift Notes (P0)

**User Story:** As an employee, I want to add notes to my shift.

**Acceptance Criteria:**
- Add notes during active shift
- Add/edit notes after clock-out (before timesheet submission)
- Notes visible to managers
- Optional field (not required)

### F3.6 - Missed Clock-Out Alert (P1)

**User Story:** As a Site Manager, I want to be alerted when employees forget to clock out.

**Acceptance Criteria:**
- Auto-detect if shift exceeds 12 hours
- Notification to site manager
- Appears in Daily Summary Report as "Missing Clock Out"
- Manager can manually close the shift

### F3.7 - Manual Shift Entry (P1)

**User Story:** As a Site Manager, I want to manually add shift entries for employees.

**Acceptance Criteria:**
- Manager enters: employee, date, start time, end time, notes
- Marked as "admin_override" method
- Requires notes/reason
- Appears in timesheet with override indicator
- Audit log entry created

---

## Module 4: Timesheets

### F4.1 - Auto-Generated Timesheets (P0)

**User Story:** As an employee, I want my shifts automatically grouped into weekly timesheets.

**Acceptance Criteria:**
- System creates timesheets for each week (Mon-Sun)
- Groups all shifts for the period
- Calculates total hours and overtime
- Shows breakdown by day
- Employee can select custom date range

### F4.2 - Submit Timesheet (P0)

**User Story:** As an employee, I want to submit my timesheet for manager review.

**Acceptance Criteria:**
- Review timesheet before submitting
- See all shifts with times, durations, notes
- Submit changes status from "Draft" to "Submitted"
- Cannot modify shifts after submission
- Manager receives notification
- Employee can track submission status

### F4.3 - Timesheet Statuses (P0)

**Statuses:**
| Status | Description |
|--------|-------------|
| Draft | Auto-generated, employee hasn't submitted |
| Submitted | Employee has submitted for review |
| Reviewed | Manager has looked at it (intermediate state) |
| Approved | Manager has approved |
| Rejected | Manager has rejected (reason required) |

### F4.4 - Approve/Reject Timesheet (P0)

**User Story:** As a Site Manager, I want to review and approve or reject timesheets.

**Acceptance Criteria:**
- View list of pending/submitted timesheets
- See detailed breakdown (daily hours, methods, geofence status, notes)
- Mark as "Reviewed" (intermediate acknowledgement)
- Approve with one click
- Reject with **required** reason/notes
- Employee notified of decision
- Manager notes field for either action

### F4.5 - Bulk Approval (P0)

**User Story:** As a Site Manager, I want to approve multiple timesheets at once.

**Acceptance Criteria:**
- Select individual timesheets (checkboxes)
- "Select All" option for pending timesheets
- Bulk Approve / Bulk Reject buttons
- **Confirmation Dialog** shows:
  - Number of selected timesheets
  - Employee names
  - Total hours across all selected
  - Optional manager notes field
  - Buttons: Approve / Reject / Cancel

### F4.6 - Undo Bulk Action (P0)

**User Story:** As a Site Manager, I want to undo a bulk approval if I made a mistake.

**Acceptance Criteria:**
- After bulk approve/reject, display notification: "X timesheets approved"
- Notification includes "Undo" button
- Undo available for **30 seconds** (configurable)
- Undo reverts all timesheets back to previous status
- After timeout, action is permanent

### F4.7 - Timesheet History (P1)

**User Story:** As an employee, I want to view my past timesheets and their statuses.

**Acceptance Criteria:**
- List of all timesheets sorted by date (newest first)
- Filter by status (all, pending, approved, rejected)
- See approval/rejection comments from manager
- Expandable detail view per timesheet

### F4.8 - Overtime Calculation (P1)

**User Story:** As the system, I want to flag overtime hours on timesheets.

**Acceptance Criteria:**
- Default threshold: **38 hours/week**
- Configurable per organization
- Hours above threshold marked as overtime
- Visual indicator on timesheet
- Overtime hours shown separately

---

## Module 5: QR Code Management

### F5.1 - Generate Site QR Code (P0)

**User Story:** As a Super Admin, I want to generate a unique QR code for each site.

**Acceptance Criteria:**
- Generate button in site settings
- Unique cryptographic token per QR code
- QR image downloadable and printable
- Stored in Supabase Storage
- Can regenerate (old code invalidated)
- Shows site name on QR image for identification

### F5.2 - QR Code Display (P1)

**User Story:** As a Site Manager, I want to display the current QR code for employees to scan.

**Acceptance Criteria:**
- Full-screen QR display mode (for tablets at reception)
- Print-friendly layout
- Shows site name and basic instructions
- Auto-updates if code rotates

### F5.3 - QR Code Rotation (P2)

**User Story:** As an Admin, I want QR codes to rotate periodically for security.

**Acceptance Criteria:**
- Configurable rotation schedule (daily, weekly, monthly, never)
- Auto-generate new code before old expires
- Grace period for old code (15 minutes)
- Managers notified to display/print new code

---

## Module 6: Employee Management

### F6.1 - Invite Employees (P1)

**User Story:** As a Site Manager, I want to invite employees to join the system.

**Acceptance Criteria:**
- Enter employee email to send invitation
- Invitation email includes site name and instructions
- Token-based link (secure, expires after 7 days)
- Manager sees pending invitations list
- Can resend or cancel pending invitations

### F6.2 - Employee List (P1)

**User Story:** As a Site Manager, I want to view and manage employees at my site.

**Acceptance Criteria:**
- List shows: name, email, mobile, employee code, status, weekly hours
- Filter by status (active, suspended, removed)
- Search by name or email
- Sort by name, hours, status

### F6.3 - Suspend/Reactivate Employee (P1)

**User Story:** As a Site Manager, I want to suspend employees temporarily.

**Acceptance Criteria:**
- Suspend: employee cannot log in, data preserved
- Reactivate: employee can log in again
- Reason field for suspension
- Audit log entry created
- Different from "Remove" (which fully deactivates)

### F6.4 - Remove Employee (P1)

**User Story:** As a Site Manager, I want to remove employees who have left.

**Acceptance Criteria:**
- Soft-delete (is_active = false, removed_at timestamp)
- Employee cannot log in
- Historical data (shifts, timesheets) preserved
- Removed from active employee counts
- Can be reactivated by admin if needed

### F6.5 - Employee Profile (P1)

**User Story:** As an employee, I want to update my profile information.

**Acceptance Criteria:**
- View/edit: name, phone, avatar
- View (read-only): email, employee code, site, role
- Password change (requires current password)
- Avatar upload to Supabase Storage

### F6.6 - Employee History (P1)

**User Story:** As a Site Manager, I want to view an employee's complete history.

**Acceptance Criteria:**
- Shift history with filters
- Timesheet submission history
- Status changes (active, suspended, reactivated)
- Total hours over custom periods

---

## Module 7: Overtime Monitoring

### F7.1 - Overtime Threshold (P1)

**User Story:** As the system, I want to monitor employees approaching overtime.

**Acceptance Criteria:**
- Default threshold: **38 hours/week**
- Configurable per organization
- Calculated in real-time based on current week shifts

### F7.2 - Overtime Dashboard Widget (P1)

**User Story:** As a Site Manager, I want to see overtime risks on my dashboard.

**Acceptance Criteria:**
- Widget shows employees approaching/exceeding threshold
- Columns: Employee Name, Weekly Hours, Overtime Hours, Status
- Status badges: "At Risk" (>35h), "Overtime" (>38h)
- Sorted by hours descending

### F7.3 - Overtime Warning Badges (P1)

**User Story:** As a Site Manager, I want employees flagged when they exceed hours.

**Acceptance Criteria:**
- Orange badge when approaching (>35h current week)
- Red badge when exceeded (>38h current week)
- Visible on employee list, dashboard, and timesheet views
- Included in overtime report

### F7.4 - Overtime Alerts (P1)

**User Story:** As a Site Manager, I want to be notified when employees hit overtime.

**Acceptance Criteria:**
- In-app notification when employee exceeds 38h in a week
- Email alert (configurable)
- Grouped into weekly overtime summary email

---

## Module 8: Dashboards

### F8.1 - Employee Dashboard (P0)

**User Story:** As an employee, I want a simple dashboard for managing my work.

**Widgets:**
- Clock In / Clock Out (large, prominent button)
- Live Shift Timer (when clocked in)
- Today's Hours
- This Week's Hours
- Current Timesheet Status

**Actions:**
- Clock In / Clock Out
- Submit Timesheet
- View Shift History
- Update Profile

### F8.2 - Site Manager Dashboard (P0)

**User Story:** As a Site Manager, I want an overview of my site(s) workforce.

**Widgets:**
- Employees Clocked In (count + names)
- Pending Timesheets (count with action button)
- Hours Logged Today
- Overtime Alerts
- Labour Cost Summary (P2)

**Actions:**
- Invite Employee
- Review Timesheets
- Generate Reports

### F8.3 - Super Admin Dashboard (P1)

**User Story:** As a Super Admin, I want a bird's-eye view of all locations.

**Widgets:**
- Total Sites (active)
- Total Active Employees
- Hours Worked Today (across all sites)
- Pending Timesheets (across all sites)
- Overtime Alerts (system-wide)

**Actions:**
- Create Site
- Invite Manager
- View Reports
- System Settings

---

## Module 9: Reporting

### F9.1 - Employee Hours Report (P1)

**Columns:** Employee, Site, Total Hours, Overtime Hours
**Filters:** Date range, site, employee
**Export:** CSV, Excel, PDF

### F9.2 - Timesheet Status Report (P1)

**Columns:** Employee, Period, Status (Pending/Reviewed/Approved/Rejected), Hours
**Filters:** Date range, site, status
**Export:** CSV, Excel, PDF

### F9.3 - Attendance Report (P1)

**Columns:** Employee, Date, Clock-In Time, Clock-Out Time, Late Start, Missed Clock-Out, Absent
**Filters:** Date range, site, employee
**Export:** CSV, Excel, PDF

### F9.4 - Overtime Report (P1)

**Columns:** Employee, Site, Weekly Hours, Overtime Hours, Status
**Filters:** Week, site
**Export:** CSV, Excel, PDF

### F9.5 - Labour Utilisation Report (P1)

**Columns:** Site, Total Hours, Employee Count, Avg Hours/Employee
**Filters:** Date range
**Export:** CSV, Excel, PDF

### F9.6 - Daily Summary Report (P1)

**Manager report showing:**
- Date
- Total Hours Worked
- Total Employees Worked
- Average Hours Per Employee
- Overtime Hours
- Missing Clock Outs

**Filters:** Day, Week, Month
**Export:** CSV, PDF

### F9.7 - Export Formats (P1)

| Format | Library | Notes |
|--------|---------|-------|
| CSV | Native | Simple, works everywhere |
| Excel (.xlsx) | `xlsx` | Formatted workbook with headers |
| PDF | `jsPDF` | Tyrepower branded header/footer |

---

## Module 10: Notifications

### F10.1 - In-App Notifications (P0)

**User Story:** As a user, I want real-time notifications for important events.

**Acceptance Criteria:**
- Bell icon with unread count badge
- Notification dropdown list
- Mark individual as read
- Mark all as read
- Auto-dismiss after viewing

**Notification Types:**
- Timesheet approved
- Timesheet rejected
- Shift reminder
- Overtime alert
- New announcement

### F10.2 - Email Notifications (P1)

**Manager receives:**
- New timesheet submissions
- Overtime alerts (weekly summary)

**Employee receives:**
- Invitation email
- Timesheet approved notification
- Timesheet rejected notification (with reason)

### F10.3 - Push Notifications (P1)

**User Story:** As a user, I want push notifications even when the app is closed.

**Acceptance Criteria:**
- PWA push via Firebase Cloud Messaging
- Configurable per user (opt-in/out per type)
- Notifications for: timesheet decisions, shift reminders, overtime

---

## Module 11: Audit & Security

### F11.1 - Audit Logs (P1)

**User Story:** As a Super Admin, I want to see a log of all important actions.

**Acceptance Criteria:**
- Track: logins, clock events, timesheet actions, setting changes, user management
- Fields: user_id, action, timestamp, details (JSON), device_info, IP
- Filterable by user, action type, date range
- Admin-only access
- Retained for 12 months minimum

### F11.2 - Login Activity (P1)

**User Story:** As a user, I want to see where my account was accessed from.

**Acceptance Criteria:**
- Last 10 login events
- Shows: date/time, device type, browser, approximate location
- Accessible from profile settings

---

## Module 12: PWA & Mobile

### F12.1 - Install as App (P1)

**User Story:** As an employee, I want to install the app on my phone.

**Acceptance Criteria:**
- PWA install prompt
- Tyrepower branded home screen icon
- Full-screen mode (no browser chrome)
- Tyrepower branded splash screen

### F12.2 - Offline Clock-In (P2)

**User Story:** As an employee, I want to clock in even without internet.

**Acceptance Criteria:**
- Detect offline status
- Store clock-in locally (IndexedDB)
- Capture GPS coordinates
- Show "pending sync" indicator
- Auto-sync when back online
- Manager sees "offline entry" tag

### F12.3 - Responsive Design (P0)

**User Story:** As a user, I want an identical experience on any device.

**Acceptance Criteria:**
- Mobile-first design
- Responsive at: mobile (<640px), tablet (640-1024px), desktop (>1024px)
- Large touch targets for clock-in (min 48x48px)
- Fast access to clock-in from any screen
- Clean dashboard layouts, minimal visual clutter
- High contrast for readability

---

## Module 13: Rostering (Phase 4)

### F13.1 - Create Roster (P3)

**User Story:** As a Site Manager, I want to create weekly rosters for my site.

**Acceptance Criteria:**
- Select week and site
- Add shift slots (date, start time, end time)
- Assign employees to shifts
- Save as draft or publish immediately
- View weekly calendar layout

### F13.2 - Publish Roster (P3)

**User Story:** As a Site Manager, I want to publish a roster so employees can see their schedule.

**Acceptance Criteria:**
- Publish action sends notifications to all assigned employees
- Published rosters visible to employees
- Can unpublish/edit and re-publish
- Shows in employee dashboard as "Upcoming Shifts"

### F13.3 - Shift Swap Requests (P3)

**User Story:** As an employee, I want to request a shift swap with a colleague.

**Acceptance Criteria:**
- Employee selects shift to swap
- Selects colleague or posts as "available for swap"
- Colleague accepts/declines
- Manager approves final swap
- Both employees notified of outcome

### F13.4 - Roster Conflict Detection (P3)

**User Story:** As a Site Manager, I want warnings if I create conflicting shifts.

**Acceptance Criteria:**
- Warn if employee double-booked
- Warn if shift would exceed weekly hours threshold
- Warn if employee is on leave/suspended
- Show conflicts before publishing

---

## Module 14: AI Features (Phase 4)

### F14.1 - Overtime Risk Prediction (P3)

**User Story:** As a Manager, I want to see which employees are likely to hit overtime this week.

**Acceptance Criteria:**
- Based on current week hours + historical patterns
- Shows predicted end-of-week hours
- Colour-coded risk level
- Recommendations to adjust schedule

### F14.2 - Labour Cost Forecasting (P3)

**User Story:** As a Manager, I want to forecast labour costs for the week/month.

**Acceptance Criteria:**
- Based on historical patterns + current roster
- Shows estimated cost per site
- Compares to previous periods
- Highlights cost anomalies

### F14.3 - Attendance Anomaly Detection (P3)

**User Story:** As a Manager, I want to be alerted to unusual attendance patterns.

**Acceptance Criteria:**
- Detect: frequent late starts, short shifts, unusual clock-in times
- Highlight employees with anomalies
- Configurable sensitivity
- Privacy-respectful (no over-surveillance)

### F14.4 - Optimal Staffing Suggestions (P3)

**User Story:** As a Manager, I want recommendations for staffing levels.

**Acceptance Criteria:**
- Based on historical demand patterns
- Suggest number of employees needed per day/time
- Factor in seasonal variations
- Available as insight in dashboard

### F14.5 - AI-Assisted Rostering (P3)

**User Story:** As a Manager, I want the system to auto-generate roster suggestions.

**Acceptance Criteria:**
- Generate roster based on: employee availability, hours constraints, historical patterns
- Respects overtime thresholds
- Manager reviews and edits before publishing
- Learns from manager adjustments over time

---

## Feature Delivery by Phase

### Phase 0: Design (Weeks 1-2)
- Wireframes for all screens in Google Stitch
- Tyrepower branding applied
- Mobile/tablet/desktop responsive layouts

### Phase 1: MVP (Weeks 3-8)
- F1.1, F1.2 (Auth)
- F2.1, F2.2, F2.3, F2.6 (Sites + Landing Page)
- F3.1, F3.2, F3.3, F3.4, F3.5 (Clock-In/Out)
- F4.1, F4.2, F4.3, F4.4, F4.5, F4.6 (Timesheets + Bulk Approval + Undo)
- F5.1 (QR Generation)
- F8.1, F8.2 (Dashboards)
- F10.1 (In-App Notifications)
- F12.3 (Responsive Design)

### Phase 2: Production Ready (Weeks 9-14)
- F1.3, F1.4, F1.6, F1.7, F1.8 (Auth Advanced)
- F2.4, F2.5 (Site Managers)
- F3.6, F3.7 (Alerts & Manual Entry)
- F4.7, F4.8 (History & Overtime Calc)
- F5.2 (QR Display)
- F6.1-F6.6 (Employee Management)
- F7.1-F7.4 (Overtime Monitoring)
- F8.3 (Admin Dashboard)
- F9.1-F9.7 (All Reports + Export)
- F10.2, F10.3 (Email + Push)
- F11.1, F11.2 (Audit & Security)
- F12.1 (PWA Install)

### Phase 3: Enhancement (Weeks 15-18)
- F1.5 (OTP)
- F2.7 (Announcements)
- F5.3 (QR Rotation)
- F12.2 (Offline Clock-In)
- Performance, accessibility, cross-device polish

### Phase 4: Rostering & AI (Weeks 19-24+)
- F13.1-F13.4 (Rostering)
- F14.1-F14.5 (AI Features)
