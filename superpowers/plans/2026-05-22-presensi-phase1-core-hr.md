# Presensi Phase 1: Core HR — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade attendance app with profile management, geofence validation, leave management, and selfie capture on check-in.

**Architecture:** Extend existing Laravel 8 MVC backend with new migrations, models, controllers. Extend Flutter 3 MVC mobile app with new screens and updated attendance flow. Existing patterns preserved — no refactoring.

**Tech Stack:** Laravel 8 + Sanctum + MySQL + Livewire (backend), Flutter 3 + MVC + location + Syncfusion maps (mobile)

**Spec:** `docs/superpowers/specs/2026-05-22-presensi-phase1-core-hr-design.md`

**Backend path:** `/Users/irzan-aldi/Downloads/PROJECT/Portofolio/presensi-backend-master`
**Flutter path:** `/Users/irzan-aldi/Downloads/PROJECT/Portofolio/presensi-flutter-master`

---

## File Structure

### Backend — New Files

```
database/migrations/
├── 2026_05_22_000001_add_profile_fields_to_users_table.php
├── 2026_05_22_000002_add_selfie_url_to_presensis_table.php
├── 2026_05_22_000003_create_office_locations_table.php
├── 2026_05_22_000004_create_leave_types_table.php
├── 2026_05_22_000005_create_leave_balances_table.php
├── 2026_05_22_000006_create_leave_requests_table.php

database/seeders/
├── OfficeLocationSeeder.php
├── LeaveTypeSeeder.php

app/Models/
├── OfficeLocation.php
├── LeaveType.php
├── LeaveBalance.php
├── LeaveRequest.php

app/Http/Controllers/API/
├── ProfileController.php
├── OfficeLocationController.php
├── LeaveTypeController.php
├── LeaveBalanceController.php
├── LeaveRequestController.php

app/Services/
├── GeofenceService.php

app/Http/Livewire/
├── OfficeLocationManagement.php
├── LeaveTypeManagement.php
├── LeaveRequestManagement.php
├── LeaveBalanceManagement.php

resources/views/livewire/
├── office-location-management.blade.php
├── leave-type-management.blade.php
├── leave-request-management.blade.php
├── leave-balance-management.blade.php

resources/views/
├── office-locations.blade.php
├── leave-types.blade.php
├── leave-requests.blade.php
├── leave-balances.blade.php
```

### Backend — Modified Files

```
app/Models/User.php                         — add new fillable fields + relations
app/Models/Presensi.php                     — add selfie_url to fillable
app/Http/Controllers/API/PresensiController.php — geofence + selfie
routes/api.php                              — new routes
routes/web.php                              — new admin routes
app/Http/Livewire/AttendanceRecords.php     — add selfie column
app/Http/Livewire/UserManagement.php        — add profile fields
database/seeders/DatabaseSeeder.php         — call new seeders
```

### Flutter — New Files

```
lib/models/
├── office-location.dart
├── leave-type.dart
├── leave-balance.dart
├── leave-request.dart

lib/controllers/
├── profile_controller.dart
├── office_location_controller.dart
├── leave_controller.dart

lib/views/
├── profile_view.dart
├── change_password_view.dart
├── leave_list_view.dart
├── apply_leave_view.dart
├── leave_approval_view.dart

lib/views/widgets/
├── leave_balance_card.dart
```

### Flutter — Modified Files

```
lib/services/api_service.dart               — new API methods
lib/services/storage_service.dart           — store role
lib/views/home_view.dart                    — replace placeholder tabs
lib/views/dashboard_view.dart               — leave balance card
lib/views/attendance_form_view.dart         — geofence + selfie
lib/main.dart                               — routes
pubspec.yaml                                — add image_picker
```

---

## Task 1: Database Migrations

**Files:**
- Create: `presensi-backend-master/database/migrations/2026_05_22_000001_add_profile_fields_to_users_table.php`
- Create: `presensi-backend-master/database/migrations/2026_05_22_000002_add_selfie_url_to_presensis_table.php`
- Create: `presensi-backend-master/database/migrations/2026_05_22_000003_create_office_locations_table.php`
- Create: `presensi-backend-master/database/migrations/2026_05_22_000004_create_leave_types_table.php`
- Create: `presensi-backend-master/database/migrations/2026_05_22_000005_create_leave_balances_table.php`
- Create: `presensi-backend-master/database/migrations/2026_05_22_000006_create_leave_requests_table.php`

- [ ] **Step 1: Create users profile fields migration**

```php
<?php
// 2026_05_22_000001_add_profile_fields_to_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddProfileFieldsToUsersTable extends Migration
{
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('phone', 15)->nullable()->after('email');
            $table->string('position', 100)->nullable()->after('phone');
            $table->string('department', 100)->nullable()->after('position');
            $table->string('avatar_url')->nullable()->after('department');
            $table->string('employee_id', 50)->nullable()->after('avatar_url');
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn(['phone', 'position', 'department', 'avatar_url', 'employee_id']);
        });
    }
}
```

- [ ] **Step 2: Create selfie_url migration for presensis**

```php
<?php
// 2026_05_22_000002_add_selfie_url_to_presensis_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddSelfieUrlToPresensisTable extends Migration
{
    public function up()
    {
        Schema::table('presensis', function (Blueprint $table) {
            $table->string('selfie_url')->nullable()->after('pulang');
        });
    }

    public function down()
    {
        Schema::table('presensis', function (Blueprint $table) {
            $table->dropColumn('selfie_url');
        });
    }
}
```

- [ ] **Step 3: Create office_locations migration**

```php
<?php
// 2026_05_22_000003_create_office_locations_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateOfficeLocationsTable extends Migration
{
    public function up()
    {
        Schema::create('office_locations', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->decimal('latitude', 10, 7);
            $table->decimal('longitude', 10, 7);
            $table->integer('radius_meters')->default(100);
            $table->boolean('is_active')->default(true);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('office_locations');
    }
}
```

- [ ] **Step 4: Create leave_types migration**

```php
<?php
// 2026_05_22_000004_create_leave_types_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLeaveTypesTable extends Migration
{
    public function up()
    {
        Schema::create('leave_types', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->integer('quota_per_year')->nullable();
            $table->boolean('requires_attachment')->default(false);
            $table->boolean('is_active')->default(true);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('leave_types');
    }
}
```

- [ ] **Step 5: Create leave_balances migration**

```php
<?php
// 2026_05_22_000005_create_leave_balances_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLeaveBalancesTable extends Migration
{
    public function up()
    {
        Schema::create('leave_balances', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id');
            $table->unsignedBigInteger('leave_type_id');
            $table->integer('year');
            $table->integer('total_quota');
            $table->integer('used')->default(0);
            $table->timestamps();

            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('leave_type_id')->references('id')->on('leave_types')->onDelete('cascade');
            $table->unique(['user_id', 'leave_type_id', 'year']);
        });
    }

    public function down()
    {
        Schema::dropIfExists('leave_balances');
    }
}
```

- [ ] **Step 6: Create leave_requests migration**

```php
<?php
// 2026_05_22_000006_create_leave_requests_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLeaveRequestsTable extends Migration
{
    public function up()
    {
        Schema::create('leave_requests', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id');
            $table->unsignedBigInteger('leave_type_id');
            $table->date('start_date');
            $table->date('end_date');
            $table->integer('total_days');
            $table->text('reason');
            $table->string('attachment_url')->nullable();
            $table->enum('status', ['pending', 'approved', 'rejected'])->default('pending');
            $table->unsignedBigInteger('approved_by')->nullable();
            $table->timestamp('approved_at')->nullable();
            $table->text('reject_reason')->nullable();
            $table->timestamps();

            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('leave_type_id')->references('id')->on('leave_types')->onDelete('cascade');
            $table->foreign('approved_by')->references('id')->on('users')->onDelete('set null');
        });
    }

    public function down()
    {
        Schema::dropIfExists('leave_requests');
    }
}
```

- [ ] **Step 7: Run migrations**

Run: `cd presensi-backend-master && php artisan migrate`

- [ ] **Step 8: Commit**

```bash
cd presensi-backend-master
git add database/migrations/
git commit -m "feat: add migrations for profile fields, office locations, leave management, selfie"
```

---

## Task 2: Models

**Files:**
- Create: `app/Models/OfficeLocation.php`
- Create: `app/Models/LeaveType.php`
- Create: `app/Models/LeaveBalance.php`
- Create: `app/Models/LeaveRequest.php`
- Modify: `app/Models/User.php`
- Modify: `app/Models/Presensi.php`

- [ ] **Step 1: Create OfficeLocation model**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class OfficeLocation extends Model
{
    protected $fillable = ['name', 'latitude', 'longitude', 'radius_meters', 'is_active'];

    protected $casts = [
        'latitude' => 'decimal:7',
        'longitude' => 'decimal:7',
        'is_active' => 'boolean',
    ];

    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }
}
```

- [ ] **Step 2: Create LeaveType model**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class LeaveType extends Model
{
    protected $fillable = ['name', 'quota_per_year', 'requires_attachment', 'is_active'];

    protected $casts = [
        'requires_attachment' => 'boolean',
        'is_active' => 'boolean',
    ];

    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }

    public function balances()
    {
        return $this->hasMany(LeaveBalance::class);
    }

    public function requests()
    {
        return $this->hasMany(LeaveRequest::class);
    }
}
```

- [ ] **Step 3: Create LeaveBalance model**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class LeaveBalance extends Model
{
    protected $fillable = ['user_id', 'leave_type_id', 'year', 'total_quota', 'used'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function leaveType()
    {
        return $this->belongsTo(LeaveType::class);
    }

    public function getRemainingAttribute()
    {
        return $this->total_quota - $this->used;
    }

    protected $appends = ['remaining'];
}
```

- [ ] **Step 4: Create LeaveRequest model**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class LeaveRequest extends Model
{
    protected $fillable = [
        'user_id', 'leave_type_id', 'start_date', 'end_date',
        'total_days', 'reason', 'attachment_url', 'status',
        'approved_by', 'approved_at', 'reject_reason',
    ];

    protected $casts = [
        'start_date' => 'date',
        'end_date' => 'date',
        'approved_at' => 'datetime',
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function leaveType()
    {
        return $this->belongsTo(LeaveType::class);
    }

    public function approver()
    {
        return $this->belongsTo(User::class, 'approved_by');
    }

    public function scopePending($query)
    {
        return $query->where('status', 'pending');
    }

    public function scopeApproved($query)
    {
        return $query->where('status', 'approved');
    }

    /**
     * Calculate business days between two dates (exclude Sat/Sun).
     */
    public static function calculateBusinessDays($startDate, $endDate)
    {
        $start = \Carbon\Carbon::parse($startDate);
        $end = \Carbon\Carbon::parse($endDate);
        $days = 0;

        while ($start->lte($end)) {
            if (!$start->isWeekend()) {
                $days++;
            }
            $start->addDay();
        }

        return $days;
    }
}
```

- [ ] **Step 5: Update User model**

Add to `$fillable` array: `'phone', 'position', 'department', 'avatar_url', 'employee_id'`

Add relations:

```php
public function leaveBalances()
{
    return $this->hasMany(LeaveBalance::class);
}

public function leaveRequests()
{
    return $this->hasMany(LeaveRequest::class);
}
```

- [ ] **Step 6: Update Presensi model**

Add `'selfie_url'` to `$fillable` array.

- [ ] **Step 7: Commit**

```bash
cd presensi-backend-master
git add app/Models/
git commit -m "feat: add models for office location, leave type, balance, request"
```

---

## Task 3: Geofence Service

**Files:**
- Create: `app/Services/GeofenceService.php`

- [ ] **Step 1: Create GeofenceService with Haversine**

```php
<?php

namespace App\Services;

use App\Models\OfficeLocation;

class GeofenceService
{
    /**
     * Calculate distance between two GPS coordinates in meters using Haversine formula.
     */
    public static function haversineDistance($lat1, $lon1, $lat2, $lon2)
    {
        $earthRadius = 6371000; // meters
        $dLat = deg2rad($lat2 - $lat1);
        $dLon = deg2rad($lon2 - $lon1);
        $a = sin($dLat / 2) * sin($dLat / 2) +
             cos(deg2rad($lat1)) * cos(deg2rad($lat2)) *
             sin($dLon / 2) * sin($dLon / 2);
        $c = 2 * atan2(sqrt($a), sqrt(1 - $a));
        return $earthRadius * $c;
    }

    /**
     * Check if coordinates are within any active office geofence.
     * Returns ['allowed' => bool, 'nearest_distance' => float, 'nearest_office' => string]
     */
    public static function validateLocation($latitude, $longitude)
    {
        $offices = OfficeLocation::active()->get();

        if ($offices->isEmpty()) {
            // No offices configured — allow all
            return ['allowed' => true, 'nearest_distance' => 0, 'nearest_office' => null];
        }

        $nearestDistance = PHP_FLOAT_MAX;
        $nearestOffice = null;
        $allowed = false;

        foreach ($offices as $office) {
            $distance = self::haversineDistance(
                $latitude, $longitude,
                $office->latitude, $office->longitude
            );

            if ($distance < $nearestDistance) {
                $nearestDistance = $distance;
                $nearestOffice = $office->name;
            }

            if ($distance <= $office->radius_meters) {
                $allowed = true;
            }
        }

        return [
            'allowed' => $allowed,
            'nearest_distance' => round($nearestDistance),
            'nearest_office' => $nearestOffice,
        ];
    }
}
```

- [ ] **Step 2: Commit**

```bash
git add app/Services/
git commit -m "feat: add GeofenceService with Haversine distance calculation"
```

---

## Task 4: Profile API Controller

**Files:**
- Create: `app/Http/Controllers/API/ProfileController.php`
- Modify: `routes/api.php`

- [ ] **Step 1: Create ProfileController**

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Storage;

class ProfileController extends Controller
{
    public function show(Request $request)
    {
        $user = $request->user();
        return response()->json([
            'success' => true,
            'data' => [
                'id' => $user->id,
                'name' => $user->name,
                'email' => $user->email,
                'phone' => $user->phone,
                'position' => $user->position,
                'department' => $user->department,
                'employee_id' => $user->employee_id,
                'avatar_url' => $user->avatar_url ? Storage::url($user->avatar_url) : null,
                'role' => $user->role ? $user->role->name : null,
            ],
        ]);
    }

    public function update(Request $request)
    {
        $request->validate([
            'name' => 'sometimes|string|max:255',
            'phone' => 'nullable|string|max:15',
            'position' => 'nullable|string|max:100',
            'department' => 'nullable|string|max:100',
            'employee_id' => 'nullable|string|max:50',
        ]);

        $user = $request->user();
        $user->update($request->only(['name', 'phone', 'position', 'department', 'employee_id']));

        return response()->json([
            'success' => true,
            'message' => 'Profil berhasil diperbarui',
            'data' => $user->fresh(),
        ]);
    }

    public function uploadAvatar(Request $request)
    {
        $request->validate([
            'avatar' => 'required|image|mimes:jpg,jpeg,png|max:2048',
        ]);

        $user = $request->user();

        // Delete old avatar if exists
        if ($user->avatar_url) {
            Storage::delete($user->avatar_url);
        }

        $path = $request->file('avatar')->storeAs(
            'public/avatars',
            $user->id . '.' . $request->file('avatar')->extension()
        );

        $user->update(['avatar_url' => $path]);

        return response()->json([
            'success' => true,
            'message' => 'Avatar berhasil diupload',
            'data' => ['avatar_url' => Storage::url($path)],
        ]);
    }

    public function changePassword(Request $request)
    {
        $request->validate([
            'old_password' => 'required|string',
            'new_password' => 'required|string|min:8|confirmed',
        ]);

        $user = $request->user();

        if (!Hash::check($request->old_password, $user->password)) {
            return response()->json([
                'success' => false,
                'message' => 'Password lama salah',
            ], 422);
        }

        $user->update(['password' => Hash::make($request->new_password)]);

        // Revoke all tokens (force re-login)
        $user->tokens()->delete();

        return response()->json([
            'success' => true,
            'message' => 'Password berhasil diubah. Silakan login kembali.',
        ]);
    }
}
```

- [ ] **Step 2: Add routes to api.php**

Add inside the existing `auth:sanctum` middleware group:

```php
// Profile
Route::get('/profile', [ProfileController::class, 'show']);
Route::put('/profile', [ProfileController::class, 'update']);
Route::post('/profile/avatar', [ProfileController::class, 'uploadAvatar']);
Route::put('/change-password', [ProfileController::class, 'changePassword']);
```

Remove the old `/profile` route that returns `$request->user()` directly from AuthController.

Add import: `use App\Http\Controllers\API\ProfileController;`

- [ ] **Step 3: Create storage symlink**

Run: `cd presensi-backend-master && php artisan storage:link`

- [ ] **Step 4: Commit**

```bash
git add app/Http/Controllers/API/ProfileController.php routes/api.php
git commit -m "feat: add profile management API — update, avatar upload, change password"
```

---

## Task 5: Office Location API + Update PresensiController

**Files:**
- Create: `app/Http/Controllers/API/OfficeLocationController.php`
- Modify: `app/Http/Controllers/API/PresensiController.php`
- Modify: `routes/api.php`

- [ ] **Step 1: Create OfficeLocationController**

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\OfficeLocation;
use Illuminate\Http\Request;

class OfficeLocationController extends Controller
{
    // Employee: list active locations
    public function index()
    {
        $locations = OfficeLocation::active()->get();
        return response()->json(['success' => true, 'data' => $locations]);
    }

    // Admin: list all
    public function adminIndex()
    {
        $locations = OfficeLocation::all();
        return response()->json(['success' => true, 'data' => $locations]);
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'latitude' => 'required|numeric',
            'longitude' => 'required|numeric',
            'radius_meters' => 'required|integer|min:10|max:5000',
            'is_active' => 'boolean',
        ]);

        $location = OfficeLocation::create($request->all());
        return response()->json(['success' => true, 'data' => $location], 201);
    }

    public function update(Request $request, $id)
    {
        $request->validate([
            'name' => 'sometimes|string|max:255',
            'latitude' => 'sometimes|numeric',
            'longitude' => 'sometimes|numeric',
            'radius_meters' => 'sometimes|integer|min:10|max:5000',
            'is_active' => 'boolean',
        ]);

        $location = OfficeLocation::findOrFail($id);
        $location->update($request->all());
        return response()->json(['success' => true, 'data' => $location]);
    }

    public function destroy($id)
    {
        OfficeLocation::findOrFail($id)->delete();
        return response()->json(['success' => true, 'message' => 'Lokasi dihapus']);
    }
}
```

- [ ] **Step 2: Update PresensiController — add geofence + selfie**

Replace the `savePresensi` method in `PresensiController.php`:

```php
public function savePresensi(Request $request)
{
    $request->validate([
        'latitude' => 'required|numeric',
        'longitude' => 'required|numeric',
        'selfie' => 'nullable|image|mimes:jpg,jpeg,png|max:2048',
    ]);

    // Geofence validation
    $geofence = \App\Services\GeofenceService::validateLocation(
        $request->latitude, $request->longitude
    );

    if (!$geofence['allowed']) {
        return response()->json([
            'success' => false,
            'message' => 'Anda di luar area kantor. Jarak ke ' . $geofence['nearest_office'] . ': ' . $geofence['nearest_distance'] . 'm',
        ], 403);
    }

    $user = $request->user();
    $today = now()->timezone('Asia/Jakarta')->toDateString();
    $currentTime = now()->timezone('Asia/Jakarta')->toTimeString();

    // Check existing record for today
    $existingPresensi = Presensi::where('user_id', $user->id)
        ->where('tanggal', $today)
        ->first();

    // Handle selfie upload
    $selfieUrl = null;
    if ($request->hasFile('selfie')) {
        $selfieUrl = $request->file('selfie')->storeAs(
            'public/selfies/' . $user->id,
            $today . '.' . $request->file('selfie')->extension()
        );
    }

    if (!$existingPresensi) {
        // Check-in
        $presensi = Presensi::create([
            'user_id' => $user->id,
            'latitude' => $request->latitude,
            'longitude' => $request->longitude,
            'tanggal' => $today,
            'masuk' => $currentTime,
            'selfie_url' => $selfieUrl,
        ]);

        return response()->json([
            'success' => true,
            'message' => 'Berhasil absen masuk',
            'data' => $presensi,
        ], 201);
    }

    if ($existingPresensi->pulang) {
        return response()->json([
            'success' => false,
            'message' => 'Anda sudah absen masuk dan pulang hari ini',
        ], 400);
    }

    // Check-out
    $existingPresensi->update([
        'pulang' => $currentTime,
    ]);

    return response()->json([
        'success' => true,
        'message' => 'Berhasil absen pulang',
        'data' => $existingPresensi,
    ]);
}
```

- [ ] **Step 3: Add routes**

Add to `routes/api.php`:

```php
// Office locations (employee)
Route::get('/office-locations', [OfficeLocationController::class, 'index']);

// Admin routes (add inside a group with role middleware)
Route::middleware(['auth:sanctum'])->prefix('admin')->group(function () {
    Route::get('/office-locations', [OfficeLocationController::class, 'adminIndex']);
    Route::post('/office-locations', [OfficeLocationController::class, 'store']);
    Route::put('/office-locations/{id}', [OfficeLocationController::class, 'update']);
    Route::delete('/office-locations/{id}', [OfficeLocationController::class, 'destroy']);
});
```

Add import: `use App\Http\Controllers\API\OfficeLocationController;`

- [ ] **Step 4: Commit**

```bash
git add app/Http/Controllers/API/ app/Services/ routes/api.php
git commit -m "feat: add office location CRUD + geofence validation + selfie capture on check-in"
```

---

## Task 6: Leave Management API

**Files:**
- Create: `app/Http/Controllers/API/LeaveTypeController.php`
- Create: `app/Http/Controllers/API/LeaveBalanceController.php`
- Create: `app/Http/Controllers/API/LeaveRequestController.php`
- Modify: `routes/api.php`

- [ ] **Step 1: Create LeaveTypeController**

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\LeaveType;
use Illuminate\Http\Request;

class LeaveTypeController extends Controller
{
    public function index()
    {
        return response()->json(['success' => true, 'data' => LeaveType::active()->get()]);
    }

    public function adminIndex()
    {
        return response()->json(['success' => true, 'data' => LeaveType::all()]);
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'quota_per_year' => 'nullable|integer|min:1',
            'requires_attachment' => 'boolean',
            'is_active' => 'boolean',
        ]);

        $type = LeaveType::create($request->all());
        return response()->json(['success' => true, 'data' => $type], 201);
    }

    public function update(Request $request, $id)
    {
        $type = LeaveType::findOrFail($id);
        $type->update($request->all());
        return response()->json(['success' => true, 'data' => $type]);
    }

    public function destroy($id)
    {
        $type = LeaveType::findOrFail($id);
        $type->update(['is_active' => false]);
        return response()->json(['success' => true, 'message' => 'Tipe cuti dinonaktifkan']);
    }
}
```

- [ ] **Step 2: Create LeaveBalanceController**

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\LeaveBalance;
use App\Models\LeaveType;
use Illuminate\Http\Request;

class LeaveBalanceController extends Controller
{
    // Employee: my balances
    public function index(Request $request)
    {
        $year = $request->get('year', now()->year);
        $balances = LeaveBalance::with('leaveType')
            ->where('user_id', $request->user()->id)
            ->where('year', $year)
            ->get();

        return response()->json(['success' => true, 'data' => $balances]);
    }

    // Admin: all balances
    public function adminIndex(Request $request)
    {
        $query = LeaveBalance::with(['user', 'leaveType']);

        if ($request->user_id) $query->where('user_id', $request->user_id);
        if ($request->year) $query->where('year', $request->year);
        else $query->where('year', now()->year);

        return response()->json(['success' => true, 'data' => $query->get()]);
    }

    // Admin: adjust quota
    public function update(Request $request, $id)
    {
        $request->validate([
            'total_quota' => 'required|integer|min:0',
        ]);

        $balance = LeaveBalance::findOrFail($id);
        $balance->update(['total_quota' => $request->total_quota]);

        return response()->json(['success' => true, 'data' => $balance]);
    }

    /**
     * Get or create balance for user + leave type + year.
     */
    public static function getOrCreateBalance($userId, $leaveTypeId, $year = null)
    {
        $year = $year ?? now()->year;
        $leaveType = LeaveType::findOrFail($leaveTypeId);

        return LeaveBalance::firstOrCreate(
            ['user_id' => $userId, 'leave_type_id' => $leaveTypeId, 'year' => $year],
            ['total_quota' => $leaveType->quota_per_year ?? 0, 'used' => 0]
        );
    }
}
```

- [ ] **Step 3: Create LeaveRequestController**

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\LeaveRequest;
use App\Models\LeaveType;
use App\Models\LeaveBalance;
use Illuminate\Http\Request;

class LeaveRequestController extends Controller
{
    // Employee: my requests
    public function index(Request $request)
    {
        $query = LeaveRequest::with('leaveType')
            ->where('user_id', $request->user()->id)
            ->orderBy('created_at', 'desc');

        if ($request->status) $query->where('status', $request->status);

        return response()->json(['success' => true, 'data' => $query->paginate(10)]);
    }

    public function show($id, Request $request)
    {
        $leave = LeaveRequest::with(['leaveType', 'approver'])
            ->where('user_id', $request->user()->id)
            ->findOrFail($id);

        return response()->json(['success' => true, 'data' => $leave]);
    }

    public function store(Request $request)
    {
        $request->validate([
            'leave_type_id' => 'required|exists:leave_types,id',
            'start_date' => 'required|date',
            'end_date' => 'required|date|after_or_equal:start_date',
            'reason' => 'required|string|max:1000',
            'attachment' => 'nullable|file|max:5120',
        ]);

        $user = $request->user();
        $leaveType = LeaveType::findOrFail($request->leave_type_id);
        $totalDays = LeaveRequest::calculateBusinessDays($request->start_date, $request->end_date);

        if ($totalDays === 0) {
            return response()->json(['success' => false, 'message' => 'Tidak ada hari kerja dalam rentang tanggal tersebut'], 422);
        }

        // Check attachment requirement
        if ($leaveType->requires_attachment && $totalDays > 2 && !$request->hasFile('attachment')) {
            return response()->json(['success' => false, 'message' => 'Lampiran wajib untuk ' . $leaveType->name . ' lebih dari 2 hari'], 422);
        }

        // Check quota if applicable
        if ($leaveType->quota_per_year) {
            $balance = LeaveBalanceController::getOrCreateBalance($user->id, $leaveType->id);
            if ($balance->remaining < $totalDays) {
                return response()->json(['success' => false, 'message' => 'Sisa cuti tidak mencukupi. Tersisa: ' . $balance->remaining . ' hari'], 422);
            }
        }

        // Check overlap
        $overlap = LeaveRequest::where('user_id', $user->id)
            ->whereIn('status', ['pending', 'approved'])
            ->where(function ($q) use ($request) {
                $q->whereBetween('start_date', [$request->start_date, $request->end_date])
                  ->orWhereBetween('end_date', [$request->start_date, $request->end_date])
                  ->orWhere(function ($q2) use ($request) {
                      $q2->where('start_date', '<=', $request->start_date)
                          ->where('end_date', '>=', $request->end_date);
                  });
            })->exists();

        if ($overlap) {
            return response()->json(['success' => false, 'message' => 'Tanggal cuti bertabrakan dengan pengajuan lain'], 422);
        }

        // Handle attachment
        $attachmentUrl = null;
        if ($request->hasFile('attachment')) {
            $attachmentUrl = $request->file('attachment')->store('public/leave-attachments/' . $user->id);
        }

        $leaveRequest = LeaveRequest::create([
            'user_id' => $user->id,
            'leave_type_id' => $request->leave_type_id,
            'start_date' => $request->start_date,
            'end_date' => $request->end_date,
            'total_days' => $totalDays,
            'reason' => $request->reason,
            'attachment_url' => $attachmentUrl,
            'status' => 'pending',
        ]);

        return response()->json(['success' => true, 'message' => 'Pengajuan cuti berhasil', 'data' => $leaveRequest->load('leaveType')], 201);
    }

    // Employee: cancel pending
    public function destroy($id, Request $request)
    {
        $leave = LeaveRequest::where('user_id', $request->user()->id)
            ->where('status', 'pending')
            ->findOrFail($id);

        $leave->delete();
        return response()->json(['success' => true, 'message' => 'Pengajuan dibatalkan']);
    }

    // Admin/HRD: all requests
    public function adminIndex(Request $request)
    {
        $query = LeaveRequest::with(['user', 'leaveType', 'approver'])
            ->orderBy('created_at', 'desc');

        if ($request->status) $query->where('status', $request->status);
        if ($request->user_id) $query->where('user_id', $request->user_id);
        if ($request->date_from) $query->where('start_date', '>=', $request->date_from);
        if ($request->date_to) $query->where('end_date', '<=', $request->date_to);

        return response()->json(['success' => true, 'data' => $query->paginate(10)]);
    }

    // HRD: approve
    public function approve($id, Request $request)
    {
        $leave = LeaveRequest::pending()->findOrFail($id);

        $leave->update([
            'status' => 'approved',
            'approved_by' => $request->user()->id,
            'approved_at' => now(),
        ]);

        // Update balance
        if ($leave->leaveType->quota_per_year) {
            $balance = LeaveBalanceController::getOrCreateBalance(
                $leave->user_id, $leave->leave_type_id
            );
            $balance->increment('used', $leave->total_days);
        }

        return response()->json(['success' => true, 'message' => 'Cuti disetujui', 'data' => $leave->fresh()]);
    }

    // HRD: reject
    public function reject($id, Request $request)
    {
        $request->validate(['reject_reason' => 'required|string|max:500']);

        $leave = LeaveRequest::pending()->findOrFail($id);

        $leave->update([
            'status' => 'rejected',
            'reject_reason' => $request->reject_reason,
            'approved_by' => $request->user()->id,
            'approved_at' => now(),
        ]);

        return response()->json(['success' => true, 'message' => 'Cuti ditolak', 'data' => $leave->fresh()]);
    }
}
```

- [ ] **Step 4: Add all leave routes to api.php**

```php
use App\Http\Controllers\API\LeaveTypeController;
use App\Http\Controllers\API\LeaveBalanceController;
use App\Http\Controllers\API\LeaveRequestController;

// Inside auth:sanctum group — Employee routes:
Route::get('/leave-types', [LeaveTypeController::class, 'index']);
Route::get('/leave-balances', [LeaveBalanceController::class, 'index']);
Route::get('/leave-requests', [LeaveRequestController::class, 'index']);
Route::post('/leave-requests', [LeaveRequestController::class, 'store']);
Route::get('/leave-requests/{id}', [LeaveRequestController::class, 'show']);
Route::delete('/leave-requests/{id}', [LeaveRequestController::class, 'destroy']);

// Inside admin prefix group:
Route::get('/admin/leave-types', [LeaveTypeController::class, 'adminIndex']);
Route::post('/admin/leave-types', [LeaveTypeController::class, 'store']);
Route::put('/admin/leave-types/{id}', [LeaveTypeController::class, 'update']);
Route::delete('/admin/leave-types/{id}', [LeaveTypeController::class, 'destroy']);

Route::get('/admin/leave-requests', [LeaveRequestController::class, 'adminIndex']);
Route::put('/admin/leave-requests/{id}/approve', [LeaveRequestController::class, 'approve']);
Route::put('/admin/leave-requests/{id}/reject', [LeaveRequestController::class, 'reject']);

Route::get('/admin/leave-balances', [LeaveBalanceController::class, 'adminIndex']);
Route::put('/admin/leave-balances/{id}', [LeaveBalanceController::class, 'update']);
```

- [ ] **Step 5: Commit**

```bash
git add app/Http/Controllers/API/ routes/api.php
git commit -m "feat: add leave management API — types, balances, requests, approval flow"
```

---

## Task 7: Seed Data

**Files:**
- Create: `database/seeders/OfficeLocationSeeder.php`
- Create: `database/seeders/LeaveTypeSeeder.php`
- Modify: `database/seeders/DatabaseSeeder.php`

- [ ] **Step 1: Create seeders**

`OfficeLocationSeeder.php`:
```php
<?php

namespace Database\Seeders;

use App\Models\OfficeLocation;
use Illuminate\Database\Seeder;

class OfficeLocationSeeder extends Seeder
{
    public function run()
    {
        OfficeLocation::firstOrCreate(
            ['name' => 'Kantor Pusat'],
            ['latitude' => -6.2088, 'longitude' => 106.8456, 'radius_meters' => 100, 'is_active' => true]
        );
    }
}
```

`LeaveTypeSeeder.php`:
```php
<?php

namespace Database\Seeders;

use App\Models\LeaveType;
use Illuminate\Database\Seeder;

class LeaveTypeSeeder extends Seeder
{
    public function run()
    {
        $types = [
            ['name' => 'Cuti Tahunan', 'quota_per_year' => 12, 'requires_attachment' => false],
            ['name' => 'Sakit', 'quota_per_year' => null, 'requires_attachment' => true],
            ['name' => 'Izin', 'quota_per_year' => null, 'requires_attachment' => false],
            ['name' => 'Cuti Melahirkan', 'quota_per_year' => 90, 'requires_attachment' => true],
        ];

        foreach ($types as $type) {
            LeaveType::firstOrCreate(['name' => $type['name']], $type);
        }
    }
}
```

- [ ] **Step 2: Update DatabaseSeeder**

Add to `run()`:
```php
$this->call([
    OfficeLocationSeeder::class,
    LeaveTypeSeeder::class,
]);
```

- [ ] **Step 3: Run seeders**

Run: `cd presensi-backend-master && php artisan db:seed --class=OfficeLocationSeeder && php artisan db:seed --class=LeaveTypeSeeder`

- [ ] **Step 4: Commit**

```bash
git add database/seeders/
git commit -m "feat: add seed data for office locations and leave types"
```

---

## Task 8: Livewire Admin Components

**Files:**
- Create: `app/Http/Livewire/OfficeLocationManagement.php` + blade view
- Create: `app/Http/Livewire/LeaveTypeManagement.php` + blade view
- Create: `app/Http/Livewire/LeaveRequestManagement.php` + blade view
- Create: `app/Http/Livewire/LeaveBalanceManagement.php` + blade view
- Create: blade layout views for each
- Modify: `routes/web.php`
- Modify: `app/Http/Livewire/AttendanceRecords.php` — add selfie column

Follow existing Livewire patterns from `AttendanceRecords.php` and `UserManagement.php`:
- Extend Component
- Use `WithPagination` trait with Bootstrap theme
- Search, filter, sort properties
- CRUD modal methods
- Render with paginated query

Each component needs:
1. Livewire PHP class in `app/Http/Livewire/`
2. Blade view in `resources/views/livewire/`
3. Page blade in `resources/views/` (extends layout, includes Livewire component)
4. Route in `routes/web.php`

**OfficeLocationManagement:** Table with name/lat/lng/radius/active, create/edit modal, delete confirm.

**LeaveTypeManagement:** Table with name/quota/requires_attachment/active, create/edit modal.

**LeaveRequestManagement:** Table with employee/type/dates/days/status, filter by status/user, approve/reject buttons for pending.

**LeaveBalanceManagement:** Table with employee/type/year/quota/used/remaining, edit quota modal.

**AttendanceRecords update:** Add selfie_url column — show thumbnail if exists, link to full image.

Add web routes:
```php
// Inside admin middleware group:
Route::get('/office-locations', function () { return view('office-locations'); })->name('admin.office-locations');
Route::get('/leave-types', function () { return view('leave-types'); })->name('admin.leave-types');
Route::get('/leave-requests', function () { return view('leave-requests'); })->name('admin.leave-requests');
Route::get('/leave-balances', function () { return view('leave-balances'); })->name('admin.leave-balances');
```

- [ ] **Step 1: Create all 4 Livewire components + views**
- [ ] **Step 2: Update AttendanceRecords with selfie column**
- [ ] **Step 3: Add web routes**
- [ ] **Step 4: Commit**

```bash
git add app/Http/Livewire/ resources/views/ routes/web.php
git commit -m "feat: add Livewire admin panels for office locations, leave types, requests, balances"
```

---

## Task 9: Flutter — Update API Service + Models

**Files:**
- Modify: `presensi-flutter-master/pubspec.yaml`
- Create: `lib/models/office-location.dart`
- Create: `lib/models/leave-type.dart`
- Create: `lib/models/leave-balance.dart`
- Create: `lib/models/leave-request.dart`
- Modify: `lib/services/api_service.dart`
- Modify: `lib/services/storage_service.dart`

- [ ] **Step 1: Add image_picker to pubspec.yaml**

```yaml
dependencies:
  image_picker: ^1.0.0
```

Run: `cd presensi-flutter-master && flutter pub get`

- [ ] **Step 2: Create Flutter models**

`lib/models/office-location.dart`:
```dart
class OfficeLocation {
  final int id;
  final String name;
  final double latitude;
  final double longitude;
  final int radiusMeters;
  final bool isActive;

  OfficeLocation({required this.id, required this.name, required this.latitude, required this.longitude, required this.radiusMeters, required this.isActive});

  factory OfficeLocation.fromJson(Map<String, dynamic> json) => OfficeLocation(
    id: json['id'], name: json['name'],
    latitude: double.parse(json['latitude'].toString()),
    longitude: double.parse(json['longitude'].toString()),
    radiusMeters: json['radius_meters'], isActive: json['is_active'],
  );
}
```

`lib/models/leave-type.dart`:
```dart
class LeaveTypeModel {
  final int id;
  final String name;
  final int? quotaPerYear;
  final bool requiresAttachment;

  LeaveTypeModel({required this.id, required this.name, this.quotaPerYear, required this.requiresAttachment});

  factory LeaveTypeModel.fromJson(Map<String, dynamic> json) => LeaveTypeModel(
    id: json['id'], name: json['name'],
    quotaPerYear: json['quota_per_year'],
    requiresAttachment: json['requires_attachment'] ?? false,
  );
}
```

`lib/models/leave-balance.dart`:
```dart
class LeaveBalanceModel {
  final int id;
  final int totalQuota;
  final int used;
  final int remaining;
  final String leaveTypeName;

  LeaveBalanceModel({required this.id, required this.totalQuota, required this.used, required this.remaining, required this.leaveTypeName});

  factory LeaveBalanceModel.fromJson(Map<String, dynamic> json) => LeaveBalanceModel(
    id: json['id'], totalQuota: json['total_quota'], used: json['used'],
    remaining: json['remaining'],
    leaveTypeName: json['leave_type'] != null ? json['leave_type']['name'] : '',
  );
}
```

`lib/models/leave-request.dart`:
```dart
class LeaveRequestModel {
  final int id;
  final String startDate;
  final String endDate;
  final int totalDays;
  final String reason;
  final String status;
  final String? rejectReason;
  final String? attachmentUrl;
  final String leaveTypeName;
  final String? approverName;

  LeaveRequestModel({
    required this.id, required this.startDate, required this.endDate,
    required this.totalDays, required this.reason, required this.status,
    this.rejectReason, this.attachmentUrl, required this.leaveTypeName,
    this.approverName,
  });

  factory LeaveRequestModel.fromJson(Map<String, dynamic> json) => LeaveRequestModel(
    id: json['id'],
    startDate: json['start_date'], endDate: json['end_date'],
    totalDays: json['total_days'], reason: json['reason'],
    status: json['status'], rejectReason: json['reject_reason'],
    attachmentUrl: json['attachment_url'],
    leaveTypeName: json['leave_type'] != null ? json['leave_type']['name'] : '',
    approverName: json['approver'] != null ? json['approver']['name'] : null,
  );
}
```

- [ ] **Step 3: Add API methods to api_service.dart**

Add these static methods:

```dart
// Profile
static Future<http.Response> updateProfile(String token, Map<String, String> data) async {
  return await http.put(Uri.parse('$baseUrl/profile'), headers: _authHeaders(token), body: jsonEncode(data));
}

static Future<http.StreamedResponse> uploadAvatar(String token, File file) async {
  var request = http.MultipartRequest('POST', Uri.parse('$baseUrl/profile/avatar'));
  request.headers['Authorization'] = 'Bearer $token';
  request.files.add(await http.MultipartFile.fromPath('avatar', file.path));
  return await request.send();
}

static Future<http.Response> changePassword(String token, String oldPw, String newPw, String confirmPw) async {
  return await http.put(Uri.parse('$baseUrl/change-password'), headers: _authHeaders(token),
    body: jsonEncode({'old_password': oldPw, 'new_password': newPw, 'new_password_confirmation': confirmPw}));
}

// Office locations
static Future<http.Response> getOfficeLocations(String token) async {
  return await http.get(Uri.parse('$baseUrl/office-locations'), headers: _authHeaders(token));
}

// Leave
static Future<http.Response> getLeaveTypes(String token) async {
  return await http.get(Uri.parse('$baseUrl/leave-types'), headers: _authHeaders(token));
}

static Future<http.Response> getLeaveBalances(String token) async {
  return await http.get(Uri.parse('$baseUrl/leave-balances'), headers: _authHeaders(token));
}

static Future<http.Response> getLeaveRequests(String token, {String? status}) async {
  var url = '$baseUrl/leave-requests';
  if (status != null) url += '?status=$status';
  return await http.get(Uri.parse(url), headers: _authHeaders(token));
}

static Future<http.Response> getLeaveRequestDetail(String token, int id) async {
  return await http.get(Uri.parse('$baseUrl/leave-requests/$id'), headers: _authHeaders(token));
}

static Future<http.StreamedResponse> submitLeaveRequest(String token, Map<String, String> data, {File? attachment}) async {
  var request = http.MultipartRequest('POST', Uri.parse('$baseUrl/leave-requests'));
  request.headers['Authorization'] = 'Bearer $token';
  data.forEach((key, value) => request.fields[key] = value);
  if (attachment != null) {
    request.files.add(await http.MultipartFile.fromPath('attachment', attachment.path));
  }
  return await request.send();
}

static Future<http.Response> cancelLeaveRequest(String token, int id) async {
  return await http.delete(Uri.parse('$baseUrl/leave-requests/$id'), headers: _authHeaders(token));
}

// Multipart save-presensi (updated)
static Future<http.StreamedResponse> savePresensiWithSelfie(String token, double lat, double lng, File selfie) async {
  var request = http.MultipartRequest('POST', Uri.parse('$baseUrl/save-presensi'));
  request.headers['Authorization'] = 'Bearer $token';
  request.fields['latitude'] = lat.toString();
  request.fields['longitude'] = lng.toString();
  request.files.add(await http.MultipartFile.fromPath('selfie', selfie.path));
  return await request.send();
}

static Map<String, String> _authHeaders(String token) => {
  'Authorization': 'Bearer $token',
  'Content-Type': 'application/json',
  'Accept': 'application/json',
};
```

- [ ] **Step 4: Update StorageService — store role**

Add to `storage_service.dart`:
```dart
static Future<void> saveRole(String role) async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setString('role', role);
}

static Future<String?> getRole() async {
  final prefs = await SharedPreferences.getInstance();
  return prefs.getString('role');
}
```

- [ ] **Step 5: Commit**

```bash
cd presensi-flutter-master
git add pubspec.yaml lib/models/ lib/services/
git commit -m "feat: add Flutter models and API service methods for profile, geofence, leave"
```

---

## Task 10: Flutter — Profile + Change Password Screens

**Files:**
- Create: `lib/controllers/profile_controller.dart`
- Create: `lib/views/profile_view.dart`
- Create: `lib/views/change_password_view.dart`
- Modify: `lib/views/home_view.dart` — replace profile placeholder

Build ProfileView as a StatefulWidget:
- Load profile on init via API
- Display avatar (CircleAvatar, tap to pick image via ImagePicker)
- Form fields: name, phone, position, department, employee_id
- Save button → PUT /api/profile
- Change password button → navigate to ChangePasswordView
- Logout button

Build ChangePasswordView:
- 3 TextFields: old password, new password, confirm
- Submit → PUT /api/change-password
- On success → clear storage, navigate to login

Update HomeView: replace the placeholder Profile tab (index 3) with ProfileView.

- [ ] **Step 1: Create controllers + screens**
- [ ] **Step 2: Wire into HomeView**
- [ ] **Step 3: Commit**

```bash
cd presensi-flutter-master
git add lib/
git commit -m "feat: add profile management and change password screens"
```

---

## Task 11: Flutter — Update Attendance Form (Geofence + Selfie)

**Files:**
- Create: `lib/controllers/office_location_controller.dart`
- Modify: `lib/views/attendance_form_view.dart`

Update AttendanceFormView:
1. On init: fetch office locations via API
2. Show office markers on Syncfusion map (MapMarkerLayer)
3. Calculate distance to nearest office using Haversine (Dart version)
4. Show status indicator: green "Dalam area (50m)" or red "Di luar area (250m)"
5. If outside radius: disable record button
6. On record button tap:
   a. Open front camera via ImagePicker
   b. Show selfie preview
   c. Upload via MultipartRequest (savePresensiWithSelfie)
7. Show success/failure SnackBar

Dart Haversine:
```dart
import 'dart:math';

double haversineDistance(double lat1, double lon1, double lat2, double lon2) {
  const earthRadius = 6371000; // meters
  final dLat = _toRadians(lat2 - lat1);
  final dLon = _toRadians(lon2 - lon1);
  final a = sin(dLat / 2) * sin(dLat / 2) +
      cos(_toRadians(lat1)) * cos(_toRadians(lat2)) * sin(dLon / 2) * sin(dLon / 2);
  final c = 2 * atan2(sqrt(a), sqrt(1 - a));
  return earthRadius * c;
}

double _toRadians(double degrees) => degrees * pi / 180;
```

- [ ] **Step 1: Create controller + update attendance form**
- [ ] **Step 2: Test geofence + selfie flow**
- [ ] **Step 3: Commit**

```bash
cd presensi-flutter-master
git add lib/
git commit -m "feat: add geofence validation and selfie capture to attendance form"
```

---

## Task 12: Flutter — Leave Management Screens

**Files:**
- Create: `lib/controllers/leave_controller.dart`
- Create: `lib/views/leave_list_view.dart`
- Create: `lib/views/apply_leave_view.dart`
- Create: `lib/views/widgets/leave_balance_card.dart`
- Modify: `lib/views/home_view.dart` — replace leave placeholder tab
- Modify: `lib/views/dashboard_view.dart` — add balance card

**LeaveListView:**
- Tab bar: All / Pending / Approved / Rejected
- ListView.builder with leave request cards
- Each card: type badge, date range, days count, status badge (colored), reason
- Pull-to-refresh
- FAB → navigate to ApplyLeaveView
- Tap card → detail with cancel button (if pending)

**ApplyLeaveView:**
- Dropdown: leave type (from API)
- Date pickers: start + end date
- Auto-calculate business days (exclude weekends) — show "X hari kerja"
- Show remaining quota for selected type
- Reason TextArea
- Attachment picker (optional)
- Submit button → multipart POST

**LeaveBalanceCard widget:**
- Compact card for dashboard
- Shows per leave type: name, used/total, progress bar
- Colored: green if >50% remaining, yellow <50%, red <20%

**DashboardView update:**
- Add LeaveBalanceCard below today's attendance card
- Load balances on init

**HomeView update:**
- Replace "Requests (coming soon)" tab with LeaveListView

- [ ] **Step 1: Create leave controller + all screens**
- [ ] **Step 2: Add balance card to dashboard**
- [ ] **Step 3: Wire into HomeView**
- [ ] **Step 4: Commit**

```bash
cd presensi-flutter-master
git add lib/
git commit -m "feat: add leave management screens — list, apply, balance card"
```

---

## Task 13: Final Verification

- [ ] **Step 1: Run Laravel**

```bash
cd presensi-backend-master && php artisan serve
```

Test key endpoints:
```bash
# Login
curl -X POST http://localhost:8000/api/login -d '{"email":"admin@example.com","password":"password"}' -H "Content-Type: application/json"

# Profile
curl http://localhost:8000/api/profile -H "Authorization: Bearer {token}"

# Office locations
curl http://localhost:8000/api/office-locations -H "Authorization: Bearer {token}"

# Leave types
curl http://localhost:8000/api/leave-types -H "Authorization: Bearer {token}"
```

- [ ] **Step 2: Run Flutter analyze**

Run: `cd presensi-flutter-master && flutter analyze`

Expected: 0 errors.

- [ ] **Step 3: Final commits**

```bash
cd presensi-backend-master && git add . && git commit -m "feat: complete Phase 1 Core HR"
cd presensi-flutter-master && git add . && git commit -m "feat: complete Phase 1 Core HR"
```
