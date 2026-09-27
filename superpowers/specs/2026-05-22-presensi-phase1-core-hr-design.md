# Presensi App Phase 1: Core HR — Design Spec

## Overview

Phase 1 upgrades the existing attendance app with core HR features: profile management, geofence validation, leave management, and selfie capture on check-in. Built on existing Laravel 8 + Sanctum backend and Flutter 3 MVC mobile app.

Phase 2 (Shift Scheduling, Overtime, Notifications) and Phase 3 (Reimbursement, Reports) will follow as separate specs.

## Goal

Production-ready attendance app deployable to any company. Configurable per deployment (office locations, leave types, quotas).

## Key Decisions

| # | Topic | Decision |
|---|-------|----------|
| 1 | Goal | Production — deploy to clients |
| 2 | Target | Ready for any company (configurable) |
| 3 | Face recognition | Selfie capture MVP, face comparison later |
| 4 | Leave approval | 1 level (Employee → HRD) |
| 5 | Architecture | Keep MVC, improve incrementally |
| 6 | Leave backdate | Allowed — start_date can be in the past |

## Tech Stack (Existing)

- **Backend:** Laravel 8, Sanctum, MySQL, Livewire (admin panel)
- **Mobile:** Flutter 3, MVC pattern, location package, Syncfusion maps
- **Auth:** Token-based (Sanctum personal access tokens)
- **Roles:** superadmin, hrd, admin, employee

---

## 1. Profile Management

### Database Migration

Add columns to existing `users` table:

```
users (new columns)
+-- phone (varchar, nullable)
+-- position (varchar, nullable)         -- jabatan
+-- department (varchar, nullable)       -- divisi
+-- avatar_url (varchar, nullable)       -- foto profil
+-- employee_id (varchar, nullable)      -- nomor karyawan
```

### API Endpoints

```
GET    /api/profile              -- get profile (add new fields to response)
PUT    /api/profile              -- update name, phone, position, department, employee_id
POST   /api/profile/avatar       -- upload avatar (multipart, max 2MB, JPG/PNG)
PUT    /api/change-password      -- old_password, new_password, new_password_confirmation
```

### Avatar Storage

- Store: `storage/app/public/avatars/{user_id}.jpg`
- Serve via `Storage::url()` → `/storage/avatars/{user_id}.jpg`
- Overwrite on re-upload (1 avatar per user)

### Validation Rules

- `phone`: nullable, max 15 chars
- `position`: nullable, max 100 chars
- `department`: nullable, max 100 chars
- `employee_id`: nullable, max 50 chars
- `avatar`: file, image, max 2MB, mimes: jpg,jpeg,png
- `old_password`: required for change-password, must match current
- `new_password`: required, min 8, confirmed

### Flutter Screens

**ProfileScreen** — replace placeholder tab:
- Display: avatar (circular, tap to change), name, email (read-only), phone, position, department, employee_id
- Edit mode: form fields, save button
- Change password button → separate screen

**ChangePasswordScreen:**
- 3 fields: current password, new password, confirm new password
- Validation: min 8 chars, passwords match
- Success → logout and redirect to login

### Admin Panel

- UserManagement Livewire: add new fields to user create/edit forms
- Display avatar thumbnail in user list

---

## 2. Geofence Validation

### Database

```
office_locations
+-- id (bigint, PK, auto-increment)
+-- name (varchar -- "Kantor Pusat", "Cabang BSD")
+-- latitude (decimal 10,7)
+-- longitude (decimal 10,7)
+-- radius_meters (integer, default 100)
+-- is_active (boolean, default true)
+-- created_at (timestamp)
+-- updated_at (timestamp)
```

### Geofence Check Logic

On check-in, backend validates employee location against ALL active office locations:

```
1. Fetch all active office_locations
2. For each location, calculate distance using Haversine formula
3. If employee is within ANY location's radius → ALLOW
4. If outside ALL radiuses → REJECT with error message + nearest office distance
```

Haversine formula (PHP):
```php
function haversineDistance($lat1, $lon1, $lat2, $lon2) {
    $earthRadius = 6371000; // meters
    $dLat = deg2rad($lat2 - $lat1);
    $dLon = deg2rad($lon2 - $lon1);
    $a = sin($dLat/2) * sin($dLat/2) +
         cos(deg2rad($lat1)) * cos(deg2rad($lat2)) *
         sin($dLon/2) * sin($dLon/2);
    $c = 2 * atan2(sqrt($a), sqrt(1-$a));
    return $earthRadius * $c;
}
```

### Updated Check-in Flow

```
POST /api/save-presensi
  body: { latitude, longitude, selfie }

  1. Validate geofence → reject if outside
  2. Validate selfie → store file
  3. Determine check-in or check-out (same as existing logic)
  4. Save presensi record
  5. Return success with distance info
```

### API Endpoints

```
# Admin (role middleware)
GET    /api/admin/office-locations          -- list all
POST   /api/admin/office-locations          -- create
PUT    /api/admin/office-locations/{id}     -- update
DELETE /api/admin/office-locations/{id}     -- delete

# Employee (authenticated)
GET    /api/office-locations                -- list active (for map display)
```

### Flutter Changes

**AttendanceFormView updates:**
- Fetch office locations on load
- Show distance to nearest office
- Within radius: green indicator "Anda dalam area kantor (50m)"
- Outside radius: red indicator "Anda di luar area kantor (250m)", button disabled
- Show office markers on map (in addition to user marker)

### Admin Panel

**OfficeLocationManagement** Livewire component:
- Table: name, lat, lng, radius, active status, actions
- Create/edit form: name, latitude, longitude, radius_meters, is_active
- Search + pagination

---

## 3. Leave Management

### Database

```
leave_types
+-- id (bigint, PK, auto-increment)
+-- name (varchar -- "Cuti Tahunan", "Sakit", "Izin")
+-- quota_per_year (integer, nullable -- null = unlimited)
+-- requires_attachment (boolean, default false)
+-- is_active (boolean, default true)
+-- created_at (timestamp)
+-- updated_at (timestamp)

leave_balances
+-- id (bigint, PK, auto-increment)
+-- user_id (FK -> users)
+-- leave_type_id (FK -> leave_types)
+-- year (integer -- 2026)
+-- total_quota (integer)
+-- used (integer, default 0)
+-- created_at (timestamp)
+-- updated_at (timestamp)

leave_requests
+-- id (bigint, PK, auto-increment)
+-- user_id (FK -> users)
+-- leave_type_id (FK -> leave_types)
+-- start_date (date)
+-- end_date (date)
+-- total_days (integer)
+-- reason (text)
+-- attachment_url (varchar, nullable)
+-- status (enum: pending, approved, rejected)
+-- approved_by (FK -> users, nullable)
+-- approved_at (timestamp, nullable)
+-- reject_reason (text, nullable)
+-- created_at (timestamp)
+-- updated_at (timestamp)
```

### Business Rules

- **Backdate allowed** — `start_date` can be in the past
- `end_date` >= `start_date`
- `total_days` = business days between start-end (exclude Saturday + Sunday)
- If leave_type has quota: `leave_balance.remaining` >= `total_days`
- No overlapping approved/pending leave requests for same user
- If `leave_type.requires_attachment` = true and `total_days` > 2: attachment required
- Employee can cancel own pending requests (DELETE)
- Only HRD can approve/reject

### Approval Flow

```
Employee submits leave request
    |
    +-- Validation passes → status: PENDING
    |
    +-- HRD reviews
    |
    +-- APPROVE:
    |       +-- status = approved
    |       +-- approved_by = hrd_user_id
    |       +-- approved_at = now
    |       +-- leave_balance.used += total_days
    |
    +-- REJECT:
            +-- status = rejected
            +-- reject_reason = "..."
```

### Leave Balance Management

- Admin/HRD can manually set quotas per employee per year
- When leave_type has `quota_per_year`: auto-create leave_balance on first request or via yearly seeder
- `remaining` = `total_quota - used` (computed, not stored)

### API Endpoints

```
# Employee
GET    /api/leave-types                              -- list active types
GET    /api/leave-balances                           -- my balances (all types, current year)
POST   /api/leave-requests                           -- submit (leave_type_id, start_date, end_date, reason, attachment)
GET    /api/leave-requests                           -- my requests (paginated, ?status=pending)
GET    /api/leave-requests/{id}                      -- detail
DELETE /api/leave-requests/{id}                      -- cancel pending request

# HRD (role middleware)
GET    /api/admin/leave-requests                     -- all requests (?status, ?user_id, ?date_from, ?date_to)
PUT    /api/admin/leave-requests/{id}/approve        -- approve
PUT    /api/admin/leave-requests/{id}/reject         -- reject (body: reject_reason)
GET    /api/admin/leave-balances                     -- all balances (?user_id, ?year)
PUT    /api/admin/leave-balances/{id}                -- manually adjust quota

# Admin
GET    /api/admin/leave-types                        -- list all
POST   /api/admin/leave-types                        -- create
PUT    /api/admin/leave-types/{id}                   -- update
DELETE /api/admin/leave-types/{id}                   -- soft delete (is_active = false)
```

### Flutter Screens

**LeaveListScreen** — replace "Leave Request (coming soon)" placeholder:
- Tab bar: All / Pending / Approved / Rejected
- Card per request: type badge, date range, total days, status badge, reason preview
- Pull-to-refresh
- FAB → apply new leave

**ApplyLeaveScreen:**
- Leave type dropdown
- Date range picker (start + end)
- Total days auto-calculated (exclude weekends) — displayed in real-time
- Reason text area
- Attachment upload button (file picker / camera)
- Remaining quota displayed for selected type
- Submit button

**LeaveBalanceCard** — on dashboard:
- Per leave type: "Cuti Tahunan: 4/12 terpakai" with progress bar
- Compact view, tap to see detail

**HRD Approval Screen** (if user role = hrd/superadmin):
- Pending requests list
- Tap → detail view with employee name, dates, reason, attachment
- Approve / Reject buttons (reject shows reason input dialog)

### Admin Panel (Livewire)

**LeaveTypeManagement:**
- CRUD leave types: name, quota_per_year, requires_attachment, is_active

**LeaveRequestManagement:**
- Table: employee name, type, dates, days, status, actions
- Filter by status, user, date range
- Approve/reject actions

**LeaveBalanceManagement:**
- Table: employee, type, year, quota, used, remaining
- Edit quota per employee

---

## 4. Selfie Capture on Check-in

### Database Change

```
presensis (add column)
+-- selfie_url (varchar, nullable)
```

### Updated Check-in Endpoint

```
POST /api/save-presensi
  Content-Type: multipart/form-data
  body: {
    latitude (required),
    longitude (required),
    selfie (file, required for check-in, optional for check-out)
  }

  Flow:
    1. Geofence validation (Section 2)
    2. Store selfie → storage/app/public/selfies/{user_id}/{Y-m-d}.jpg
    3. Determine check-in or check-out:
       - No record today → check-in (masuk)
       - Record exists, pulang is null → check-out (pulang)
    4. Save presensi with selfie_url
    5. Return response
```

### File Constraints

- Max size: 2MB
- Formats: jpg, jpeg, png
- Flutter compresses image before upload (max 800px width, 70% quality)

### Flutter Changes

**AttendanceFormView updates:**
1. After geofence check passes
2. Open front camera: `ImagePicker.pickImage(source: ImageSource.camera, preferredCameraDevice: CameraDevice.front)`
3. Show selfie preview in circular frame
4. Confirm button → upload with lat/lng as multipart
5. Switch from `http.post` to `http.MultipartRequest` for file upload

### Admin Panel

- AttendanceRecords Livewire: add selfie thumbnail column
- Click thumbnail → modal with full-size image

---

## 5. Complete New API Summary

### Public (no auth)
```
POST   /api/register
POST   /api/login
```

### Employee (auth:sanctum)
```
# Existing
GET    /api/profile
POST   /api/logout
GET    /api/get-presensi
POST   /api/save-presensi              -- updated: multipart, geofence, selfie

# New
PUT    /api/profile                     -- update profile fields
POST   /api/profile/avatar              -- upload avatar
PUT    /api/change-password             -- change password
GET    /api/office-locations            -- active office locations
GET    /api/leave-types                 -- active leave types
GET    /api/leave-balances              -- my balances
POST   /api/leave-requests              -- apply leave
GET    /api/leave-requests              -- my requests
GET    /api/leave-requests/{id}         -- request detail
DELETE /api/leave-requests/{id}         -- cancel pending
```

### HRD/Admin (auth:sanctum + role middleware)
```
# Office Locations
GET    /api/admin/office-locations
POST   /api/admin/office-locations
PUT    /api/admin/office-locations/{id}
DELETE /api/admin/office-locations/{id}

# Leave Management
GET    /api/admin/leave-requests
PUT    /api/admin/leave-requests/{id}/approve
PUT    /api/admin/leave-requests/{id}/reject
GET    /api/admin/leave-balances
PUT    /api/admin/leave-balances/{id}
GET    /api/admin/leave-types
POST   /api/admin/leave-types
PUT    /api/admin/leave-types/{id}
DELETE /api/admin/leave-types/{id}
```

---

## 6. New Database Tables Summary

```
office_locations       -- geofence config
leave_types            -- cuti tahunan, sakit, izin, dll
leave_balances         -- quota per user per type per year
leave_requests         -- applications with approval status
```

Modified existing tables:
- `users` — 5 new columns (phone, position, department, avatar_url, employee_id)
- `presensis` — 1 new column (selfie_url)

---

## 7. New Flutter Packages Needed

```yaml
image_picker: ^1.0.0     # selfie capture + avatar upload
```

Already installed: `location`, `syncfusion_flutter_maps`, `intl`, `shared_preferences`, `http`

---

## 8. Seed Data

### Default Leave Types
```
| Name           | Quota/Year | Requires Attachment |
|----------------|------------|---------------------|
| Cuti Tahunan   | 12         | false               |
| Sakit          | null       | true (if > 2 days)  |
| Izin           | null       | false               |
| Cuti Melahirkan| 90         | true                |
```

### Default Office Location
```
| Name          | Latitude   | Longitude   | Radius |
|---------------|------------|-------------|--------|
| Kantor Pusat  | -6.2088    | 106.8456    | 100m   |
```
