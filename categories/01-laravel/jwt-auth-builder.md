---
name: jwt-auth-builder
description: Configures JWT authentication in Laravel 12 with token cache management, password reset, profile update, and audits. ACL support is conditional.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# JWT Auth Builder

Configures complete JWT authentication with single-session enforcement via token cache.

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model + JWTAuth) → Model
Response always via Resource
```

## Before Starting - Check for ACL

**IMPORTANT:** Before creating files, check if project has ACL configured:

```bash
# Check for Modular ACL (3-level: User → Role → Module → Permission)
ls app/Models/Module.php 2>/dev/null
ls app/Traits/HasModulePermission.php 2>/dev/null

# Check for Simple ACL (spatie standard)
grep -r "spatie/laravel-permission" composer.json
grep -l "HasRoles" app/Models/User.php
```

### ACL Type Detection

| Check | Result | Action |
|-------|--------|--------|
| `Module.php` + `HasModulePermission.php` exists | **Modular ACL** | Include UserService with module permissions |
| Only `spatie/laravel-permission` + `HasRoles` | **Simple ACL** | Include UserService with basic roles |
| None of the above | **No ACL** | Skip all ACL logic |

### If MODULAR ACL EXISTS (Module-based):
- Include `UserService` dependency
- Load roles with module permissions in `me()`
- Use `UserResource` that includes roles/modules/permissions
- **See `@whyll-agents:acl-builder` for full ACL setup**

### If SIMPLE ACL EXISTS (Spatie standard):
- Include `UserService` dependency for `loadRoleAndPermissions()`
- Include roles/permissions in `me()` response
- Use `UserResource` that includes roles/permissions

### If NO ACL:
- **DO NOT** include `UserService` dependency
- **DO NOT** call `loadRoleAndPermissions()`
- Return user directly without roles/permissions
- Use simple `UserResource` without roles

## Install

```bash
composer require tymon/jwt-auth
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
php artisan jwt:secret
```

## Config (`config/auth.php`)

```php
'guards' => [
    'api' => ['driver' => 'jwt', 'provider' => 'users'],
],
```

## User Model

```php
use Tymon\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    public function getJWTIdentifier(): mixed { return $this->getKey(); }
    public function getJWTCustomClaims(): array { return []; }
}
```

## Routes (`routes/api.php`)

```php
use App\Http\Controllers\AuthController;

Route::controller(AuthController::class)->prefix('auth')->group(function () {
    Route::post('/', 'login');
    Route::get('/', 'me');
    Route::delete('/', 'logout');
    Route::put('/', 'refresh');
    Route::post('/forgot-password', 'forgotPassword')->name('password.email');
    Route::post('/reset-password', 'resetPassword')->name('password.update');
    Route::get('/reset-password/{token}', 'showResetForm')->name('password.reset');
    Route::put('/profile', 'updateProfile');
    Route::get('/audits', 'getUserAudits');
});
```

**Note:** Always add `use` import at top of routes file, never use inline namespace.

## AuthController (NO LOGIC)

```php
class AuthController extends Controller
{
    private AuthService $service;
    public function __construct() { $this->service = new AuthService(); }

    public function login(LoginRequest $r): JsonResource { return $this->service->login($r->validated()); }
    public function me(): JsonResource { return $this->service->me(); }
    public function logout(): JsonResource { return $this->service->logout(); }
    public function refresh(): JsonResource { return $this->service->refresh(); }
    public function forgotPassword(ForgotPasswordRequest $r): JsonResource { return $this->service->forgotPassword($r->validated()); }
    public function resetPassword(ResetPasswordRequest $r): JsonResource { return $this->service->resetPassword($r->validated()); }
    public function showResetForm(string $token) { return $this->service->showResetForm($token, request()->boolean('isMobile')); }
    public function updateProfile(UpdateProfileRequest $r): JsonResource { return $this->service->updateProfile($r->validated()); }
    public function getUserAudits(): JsonResource { return $this->service->getUserAudits(); }
}
```

## AuthService

### With ACL (if project has roles/permissions)

```php
class AuthService extends Service
{
    protected UserService $userService; // Only if ACL exists

    public function __construct()
    {
        $this->model = new User();
        $this->repository = new AuthRepository();
        $this->userService = new UserService(); // Only if ACL exists
    }

    public function login(array $credentials): JsonResource
    {
        try {
            $token = $this->repository->authenticate($credentials);
            if (!$token) {
                return new ErrorResource($this->model, 'Unauthorized', 401);
            }
            $user = Auth::user();
            $this->repository->storeCurrentToken($user->uuid, $token);
            return new TokenResource(['token' => $token]);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:login - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    // WITH ACL - includes roles/permissions
    public function me(): JsonResource
    {
        try {
            $user = $this->repository->getAuthenticatedUser();
            $currentToken = JWTAuth::getToken()->get();
            $this->repository->ensureTokenInCache($user->uuid, $currentToken);
            $userWithRoles = $this->userService->loadRoleAndPermissions($user); // Only if ACL
            return new UserResource($userWithRoles);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:me - {$e->getMessage()}");
            if (str_contains($e->getMessage(), 'invalidated')) {
                return new ErrorResource($this->model, 'Token invalidated, please login again', 401);
            }
            return new ErrorResource($this->model);
        }
    }
```

### Without ACL (simple auth only)

```php
class AuthService extends Service
{
    public function __construct()
    {
        $this->model = new User();
        $this->repository = new AuthRepository();
        // NO UserService - no ACL
    }

    // WITHOUT ACL - simple user response
    public function me(): JsonResource
    {
        try {
            $user = $this->repository->getAuthenticatedUser();
            $currentToken = JWTAuth::getToken()->get();
            $this->repository->ensureTokenInCache($user->uuid, $currentToken);
            return new UserResource($user); // Direct user, no roles
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:me - {$e->getMessage()}");
            if (str_contains($e->getMessage(), 'invalidated')) {
                return new ErrorResource($this->model, 'Token invalidated, please login again', 401);
            }
            return new ErrorResource($this->model);
        }
    }
```

### Common methods (both versions)

    public function logout(): JsonResource
    {
        try {
            $user = Auth::user();
            $token = JWTAuth::getToken();
            $tokenString = $token->get();
            
            try {
                $this->repository->ensureTokenInCache($user->uuid, $tokenString);
            } catch (Exception $e) {
                Log::channel('services')->warning('Logout with invalidated token, proceeding');
            }
            
            $this->repository->invalidateToken($token);
            $this->repository->removeTokenFromCache($user->uuid);
            return new MessageResource(['message' => 'Successfully logged out']);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:logout - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function refresh(): JsonResource
    {
        try {
            $user = Auth::user();
            $oldToken = JWTAuth::getToken();
            $oldTokenString = $oldToken->get();
            
            $this->repository->ensureTokenInCache($user->uuid, $oldTokenString);
            $newToken = $this->repository->refreshToken();
            $this->repository->invalidateToken($oldToken);
            $this->repository->storeCurrentToken($user->uuid, $newToken);
            
            return new TokenResource(['token' => $newToken]);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:refresh - {$e->getMessage()}");
            if (str_contains($e->getMessage(), 'invalidated')) {
                return new ErrorResource($this->model, 'Token invalidated, please login again', 401);
            }
            return new ErrorResource($this->model);
        }
    }

    public function forgotPassword(array $data): JsonResource
    {
        try {
            $status = $this->repository->sendPasswordResetLink($data);
            $success = $status === Password::RESET_LINK_SENT;
            return new StatusResource(['status' => $success ? 'success' : 'error', 'message' => __($status)], $success ? 200 : 400);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:forgotPassword - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function showResetForm(string $token, bool $isMobile = false)
    {
        return view('emails.auth.reset-password', ['token' => $token, 'isMobile' => $isMobile]);
    }

    public function resetPassword(array $data): JsonResource
    {
        try {
            $status = $this->repository->resetPassword($data);
            $success = $status === Password::PASSWORD_RESET;
            return new StatusResource(['status' => $success ? 'success' : 'error', 'message' => __($status)], $success ? 200 : 400);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:resetPassword - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function updateProfile(array $data): JsonResource
    {
        try {
            $result = $this->repository->updateProfile($data);
            if ($result) {
                return new StatusResource(['status' => 'success', 'message' => 'Profile updated successfully']);
            }
            return new ErrorResource($this->model, 'Could not update profile', 400);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:updateProfile - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function getUserAudits(): JsonResource
    {
        try {
            $audits = $this->repository->getUserAudits();
            return new AuditCollection($audits);
        } catch (Exception $e) {
            Log::channel('services')->error("AuthService:getUserAudits - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }
}
```

## AuthRepository

```php
class AuthRepository extends Repository
{
    public function __construct() { $this->model = new User(); }

    public function authenticate(array $credentials): ?string
    {
        try {
            return Auth::attempt($credentials) ? JWTAuth::fromUser(Auth::user()) : null;
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:authenticate - {$e->getMessage()}");
            throw $e;
        }
    }

    public function getAuthenticatedUser(): ?User
    {
        return Auth::user();
    }

    public function invalidateToken($token): void
    {
        try {
            JWTAuth::invalidate($token);
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:invalidateToken - {$e->getMessage()}");
            throw $e;
        }
    }

    public function refreshToken(): string
    {
        try {
            return JWTAuth::parseToken()->refresh();
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:refreshToken - {$e->getMessage()}");
            throw $e;
        }
    }

    public function getTokenTTL(): int
    {
        return config('jwt.ttl') * 60;
    }

    public function sendPasswordResetLink(array $data): string
    {
        try {
            return Password::sendResetLink($data);
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:sendPasswordResetLink - {$e->getMessage()}");
            throw $e;
        }
    }

    public function resetPassword(array $data): string
    {
        try {
            return Password::reset($data, function (User $user, string $password) {
                $user->forceFill(['password' => Hash::make($password)])->setRememberToken(Str::random(60));
                $user->save();
                event(new PasswordReset($user));
            });
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:resetPassword - {$e->getMessage()}");
            throw $e;
        }
    }

    public function updateProfile(array $data): bool
    {
        try {
            $user = Auth::user();
            if (!$user) return false;

            $updateData = [];
            if (isset($data['password'])) $updateData['password'] = Hash::make($data['password']);
            if (isset($data['name'])) $updateData['name'] = $data['name'];
            if (empty($updateData)) return false;

            return User::where('uuid', $user->uuid)->update($updateData) > 0;
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:updateProfile - {$e->getMessage()}");
            throw $e;
        }
    }

    public function getUserAudits(): LengthAwarePaginator
    {
        try {
            $user = Auth::user();
            if (!$user) throw new Exception('User not authenticated');

            $query = Audit::query()
                ->where('user_type', User::class)
                ->where('user_id', $user->uuid)
                ->orderBy('created_at', 'desc');

            if ($event = request('type')) $query->where('event', 'like', "%{$event}%");
            if ($type = request('model_type')) $query->where('auditable_type', 'like', "%{$type}%");

            return $query->paginate(request('per_page', 10));
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:getUserAudits - {$e->getMessage()}");
            throw $e;
        }
    }

    public function storeCurrentToken(string $userUuid, string $token): void
    {
        try {
            $cacheKey = "user_active_token:{$userUuid}";
            $previousToken = Cache::get($cacheKey);

            if ($previousToken && $previousToken !== $token) {
                try { JWTAuth::setToken($previousToken)->invalidate(); }
                catch (Exception $e) { Log::channel('repositories')->info("Previous token already invalid"); }
            }

            Cache::put($cacheKey, $token, now()->addMinutes(config('jwt.ttl')));
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:storeCurrentToken - {$e->getMessage()}");
            throw $e;
        }
    }

    public function ensureTokenInCache(string $userUuid, string $currentToken): void
    {
        try {
            $cacheKey = "user_active_token:{$userUuid}";
            $cachedToken = Cache::get($cacheKey);

            if (!$cachedToken || $cachedToken !== $currentToken) {
                Cache::put($cacheKey, $currentToken, now()->addMinutes(config('jwt.ttl')));
            }
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:ensureTokenInCache - {$e->getMessage()}");
            throw $e;
        }
    }

    public function removeTokenFromCache(string $userUuid): void
    {
        try {
            Cache::forget("user_active_token:{$userUuid}");
        } catch (Exception $e) {
            Log::channel('repositories')->error("AuthRepository:removeTokenFromCache - {$e->getMessage()}");
            throw $e;
        }
    }
}
```

## FormRequests

```php
// LoginRequest
class LoginRequest extends FormRequest {
    public function authorize(): bool { return true; }
    public function rules(): array { return ['email' => ['required', 'email'], 'password' => ['required', 'string']]; }
}

// ForgotPasswordRequest
class ForgotPasswordRequest extends FormRequest {
    public function authorize(): bool { return true; }
    public function rules(): array { return ['email' => ['required', 'email', 'exists:users,email']]; }
}

// ResetPasswordRequest
class ResetPasswordRequest extends FormRequest {
    public function authorize(): bool { return true; }
    public function rules(): array {
        return [
            'token' => ['required', 'string'],
            'email' => ['required', 'email', 'exists:users,email'],
            'password' => ['required', 'string', 'min:6', 'confirmed'],
        ];
    }
}

// UpdateProfileRequest
class UpdateProfileRequest extends FormRequest {
    public function authorize(): bool { return true; }
    public function rules(): array {
        return [
            'name' => ['sometimes', 'string', 'max:255'],
            'password' => ['sometimes', 'string', 'min:6', 'confirmed'],
        ];
    }
}
```

## Resources

```php
// TokenResource - for array data
class TokenResource extends JsonResource
{
    public static $wrap = null; // Disable data wrapping for token response

    public function toArray(Request $r): array
    {
        return [
            'access_token' => $this->resource['token'],
            'token_type' => 'bearer',
            'expires_in' => config('jwt.ttl') * 60,
        ];
    }
}

// StatusResource - for array data
class StatusResource extends JsonResource
{
    public static $wrap = null;

    public function toArray(Request $r): array
    {
        return [
            'status' => $this->resource['status'],
            'message' => $this->resource['message'],
        ];
    }
}

// MessageResource - for array data
class MessageResource extends JsonResource
{
    public static $wrap = null;

    public function toArray(Request $r): array
    {
        return ['message' => $this->resource['message']];
    }
}
```

## JwtMiddleware

```php
class JwtMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        try { JWTAuth::parseToken()->authenticate(); }
        catch (Exception $e) { return response()->json(['error' => 'Unauthorized'], 401); }
        return $next($request);
    }
}
```

Register in `bootstrap/app.php`:
```php
$middleware->alias(['jwt' => \App\Http\Middleware\JwtMiddleware::class]);
```

## Workflow

1. **Check for ACL** (grep for spatie/laravel-permission, Role model, HasRoles trait)
2. Install package + jwt:secret
3. Edit config/auth.php
4. Edit User model (JWTSubject)
5. Create: AuthRepository → AuthService → FormRequests → AuthController → Resources
   - **If ACL exists:** include UserService, loadRoleAndPermissions in me()
   - **If NO ACL:** simple user response without roles
6. Create JwtMiddleware + register
7. Edit routes/api.php
8. Run `vendor/bin/pint --dirty`

## ACL Detection Summary

| Check | ACL Exists | No ACL |
|-------|------------|--------|
| `composer.json` has `spatie/laravel-permission` | ✅ Include ACL | ❌ Skip ACL |
| `app/Models/Role.php` exists | ✅ Include ACL | ❌ Skip ACL |
| `User.php` has `HasRoles` trait | ✅ Include ACL | ❌ Skip ACL |

**If ANY of these checks pass → Include ACL support**
**If NONE pass → Simple auth without ACL**
