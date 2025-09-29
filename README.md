# WorkOS Passport Starter Kit

[![Tests](https://github.com/inmanturbo/homework-starter-kit/workflows/Tests/badge.svg)](https://github.com/inmanturbo/homework-starter-kit/actions)

A ready-to-use Laravel application that provides a self-hosted, WorkOS-compatible OAuth server using Laravel Passport. This starter kit gives you a complete authentication server that can replace WorkOS while maintaining full API compatibility.

## Features

- **Drop-in WorkOS Replacement**: Complete compatibility with WorkOS UserManagement API
- **Laravel Passport Integration**: Robust OAuth2 server implementation
- **ULID User IDs**: Uses Laravel's built-in ULID support for portable, sortable user identifiers
- **String Client IDs**: Full compatibility with WorkOS client applications
- **Auto-Approval for First-Party Clients**: Seamless user experience for your own apps
- **JWT Token Support**: WorkOS-compatible token format with JWKS endpoint
- **Modern Laravel Architecture**: Request-based routing with clean separation of concerns
- **Ready to Deploy**: Pre-configured Laravel application

## Quick Start

### 1. Install Dependencies

```bash
composer install
npm install && npm run build
```

### 2. Environment Setup

```bash
cp .env.example .env
php artisan key:generate
```

Configure your database in `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=workos_passport
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

### 3. Database Setup

```bash
php artisan migrate
```

### 4. Create OAuth Client

Create a first-party client (auto-approves authorization):

```bash
php artisan passport:client --public
```

Save the Client ID - you'll need this for your client applications.

### 5. Start the Server

```bash
php artisan serve --port=8000
```

Your WorkOS-compatible OAuth server is now running at `http://localhost:8000`!

## API Endpoints

The starter kit provides these WorkOS-compatible endpoints:

- `GET /user_management/authorize` - OAuth authorization endpoint
- `POST /user_management/authenticate` - Token exchange endpoint
- `POST /user_management/authenticate_with_refresh_token` - Refresh token endpoint
- `GET /user_management/users/{userId}` - Get user information
- `GET /sso/jwks/{clientId}` - JWKS endpoint for token verification

## Client Application Setup

### Laravel Applications with WorkOS SDK

Configure your client application to use your OAuth server:

**Option 1: Configuration-Based (Recommended)**

Add to your `config/services.php`:

```php
'workos' => [
    'client_id' => env('WORKOS_CLIENT_ID'),
    'secret' => env('WORKOS_API_KEY'),
    'redirect_url' => env('WORKOS_REDIRECT_URL'),
    'base_url' => env('WORKOS_BASE_URL', 'https://api.workos.com/'),
],
```

In your `.env`:

```env
WORKOS_CLIENT_ID=your_oauth_client_id
WORKOS_API_KEY=your_oauth_client_secret
WORKOS_REDIRECT_URL=http://your-app.test/authenticate
WORKOS_BASE_URL=http://localhost:8000/
```

In your `AppServiceProvider`:

```php
use WorkOS\WorkOS;

public function boot()
{
    $baseUrl = config('services.workos.base_url');
    if ($baseUrl && $baseUrl !== 'https://api.workos.com/') {
        WorkOS::setApiBaseUrl($baseUrl);
    }
}
```

**Option 2: Environment-Based**

```php
// In AppServiceProvider
public function boot()
{
    if (app()->environment('local')) {
        WorkOS::setApiBaseUrl('http://localhost:8000/');
    }
}
```

## Configuration

### Passport Configuration

The starter kit comes pre-configured with sensible defaults in `app/Providers/PassportServiceProvider.php`:

```php
// Token lifetimes
Passport::tokensExpireIn(CarbonInterval::days(15));
Passport::refreshTokensExpireIn(CarbonInterval::days(30));
Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));

// Default scopes
Passport::tokensCan([
    'read' => 'Read user information',
    'write' => 'Modify user information',
]);

Passport::setDefaultScope(['read']);
```

### User Model & ULID Support

The starter kit uses Laravel's built-in ULID support for user IDs, providing:

- **Globally Unique**: ULIDs are guaranteed to be unique across distributed systems
- **Sortable**: Lexicographically sortable by creation time
- **URL-Safe**: No special characters, perfect for APIs
- **Portable**: Work across different databases and systems

The `User` model includes the `HasUlids` trait:

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, HasUlids, Notifiable, TwoFactorAuthenticatable;
    // ...
}
```

User IDs will be ULIDs like: `01ARZ3NDEKTSV4RRFFQ69G5FAV`

If you use a custom user model, ensure it includes the `HasUlids` trait and update `config/auth.php`:

```php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\CustomUser::class, // Your custom model
    ],
],
```

## Testing the Setup

Test your OAuth flow:

```bash
# Test authorization endpoint
curl -I "http://localhost:8000/oauth/authorize?client_id=your_client_id&redirect_uri=http://your-app.test/authenticate&state=test_state"

# Test token exchange (after getting authorization code)
curl -X POST "http://localhost:8000/user_management/authenticate" \
  -H "Content-Type: application/json" \
  -d '{
    "grant_type": "authorization_code",
    "client_id": "your_client_id",
    "code": "authorization_code_here"
  }'
```

## Deployment

### Production Considerations

1. **Environment Variables**: Set proper production values in `.env`
2. **Database**: Use a production database (PostgreSQL, MySQL)
3. **HTTPS**: Always use HTTPS in production
4. **Client Secrets**: Keep OAuth client secrets secure
5. **Token Security**: Consider shorter token lifetimes for production

### Deployment Commands

```bash
# Install dependencies
composer install --optimize-autoloader --no-dev

# Clear and cache config
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Run migrations
php artisan migrate --force
```

## Package Information

This starter kit uses the [inmanturbo/homework](https://github.com/inmanturbo/homework) package to provide WorkOS compatibility. The package:

- **Modern Architecture**: Request-based routing with type-hinted FormRequest classes
- **WorkOS API Compatibility**: Extends Laravel Passport with WorkOS-compatible endpoints
- **Auto-Approval Middleware**: Seamless first-party client authorization
- **WorkOS Response Format**: Returns snake_case formatted API responses
- **JWT & JWKS Support**: Token verification with public key endpoints
- **ULID Compatible**: Works seamlessly with ULID user identifiers

## Migration from WorkOS

To migrate from WorkOS to this self-hosted solution:

1. **Deploy this starter kit** to your server
2. **Create OAuth clients** matching your WorkOS application IDs
3. **Update client applications** to point to your OAuth server
4. **Test the authentication flow** thoroughly
5. **Switch DNS/URLs** when ready

## Requirements

- PHP ^8.2
- Laravel ^12.0
- Laravel Passport ^13.0
- inmanturbo/homework ^0.0.2
- MySQL/PostgreSQL/SQLite
- Composer 2.x
- Node.js & NPM (for asset compilation)

## Support

- **Package Issues**: [inmanturbo/homework GitHub Issues](https://github.com/inmanturbo/homework/issues)
- **Starter Kit Issues**: [homework-starter-kit GitHub Issues](https://github.com/inmanturbo/homework-starter-kit/issues)

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Self-hosted • Secure • WorkOS Compatible**