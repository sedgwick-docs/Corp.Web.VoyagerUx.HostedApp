# Corp.Web.VoyagerUx.HostedApp

A NuGet package for ASP.NET Core MVC and Razor Pages applications hosted as iframes within the VoyagerUx portal. Provides shared session management, activity tracking, navigation control, navigation history tracking, and toast notifications.

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration Reference](#configuration-reference)
- [API Reference](#api-reference)
  - [Shared Session Management](#shared-session-management)
  - [Navigation Control](#navigation-control)
  - [Navigation History Tracking](#navigation-history-tracking)
  - [Toast Notifications](#toast-notifications)
  - [Activity Tracking](#activity-tracking)
- [Tag Helpers](#tag-helpers)
- [Standalone Mode](#standalone-mode)
- [Styling](#styling)
- [Troubleshooting](#troubleshooting)
- [Package Information](#package-information)

---

## Features

| Feature | Description |
|---------|-------------|
| **Shared Session Management** | Cookie-based cross-application data sharing with server & client APIs |
| **Navigation History Tracking** | Automatic user navigation tracking with `[TrackNavigationHistory]` attribute via client-side postMessage |
| **Navigation Control** | Control parent portal navigation from iframe apps |
| **Activity Tracking** | Automatic iframe activity detection and session timeout prevention |
| **Session Timeout Warning** | Configurable countdown dialog before session expiration with renew/end options |
| **Toast Notifications** | Cross-iframe notification system via `window.postMessage` |
| **Iframe Security** | Antiforgery configured for iframe hosting (`X-Frame-Options` suppressed, `SameSite=None`, `Secure`) |
| **Structured Logging** | Serilog logging and ASP.NET Core session configured via `Corp.Lib.Logging` |
| **Standalone Mode** | Run applications independently during development |
| **Tag Helpers** | Easy integration with `<voyagerux-scripts />`, `<voyagerux-tracking-meta />`, and `<voyagerux-feature-tests />` |
| **CSP Compliant** | No inline JavaScript - configuration via data attributes |
| **Themed CSS** | Pre-built stylesheet for hosted application consistency |

---

## Installation

### NuGet Package (Recommended)

```bash
dotnet add package Corp.Web.VoyagerUx.HostedApp
```

### Package Reference

```xml
<PackageReference Include="Corp.Web.VoyagerUx.HostedApp" Version="10.1.3" />
```

---

## Quick Start

### 1. Add Configuration

Add `VoyagerUxSettings` to `appsettings.json`:

```json
{
  "VoyagerUxSettings": {
    "AppName": "My Application",
    "PortalBaseUrl": "https://localhost:7250"
  }
}
```

**Required Settings:**

| Setting | Description |
|---------|-------------|
| `AppName` | Your application name (used for tracking and logging) |
| `PortalBaseUrl` | VoyagerUx portal URL |

**Portal URLs:**
- Development: `https://localhost:7250`
- Production: `https://mcsvoyager.specialty-web.com`

### 2. Register Services

Add to `Program.cs`:

```csharp
using Corp.Web.VoyagerUx.HostedApp.Extensions;

var builder = WebApplication.CreateBuilder(args);

// Add VoyagerUx hosted app services (includes Serilog logging, session, and antiforgery)
builder.AddVoyagerUxHostedApp(
    configure: options =>
    {
        builder.Configuration.GetSection("VoyagerUxSettings").Bind(options);
    },
    sessionExpirationInMinutes: 60  // Must match the portal's SessionExpirationInMinutes value
);

var app = builder.Build();

// Configure VoyagerUx middleware (includes Serilog request logging and session middleware)
app.UseVoyagerUxHostedApp();

app.UseStaticFiles();  // Required for scripts and CSS
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Or for Razor Pages:
// app.MapRazorPages();

app.Run();
```

> **Important:** Do not call `builder.AddLogging()` or `app.UseLogging()` separately — `AddVoyagerUxHostedApp` and `UseVoyagerUxHostedApp` handle these internally.

### 3. Add Scripts and CSS to Layout

Add to your `_Layout.cshtml`:

**In `<head>`:**
```html
<link rel="stylesheet" href="~/css/site.css" />
<voyagerux-tracking-meta />
<voyagerux-scripts />
```

### 4. Add Tag Helper Import

Add to `_ViewImports.cshtml`:

```cshtml
@addTagHelper *, Corp.Web.VoyagerUx.HostedApp
```

**Done!** Your application now has:
- ✅ Serilog structured logging (via `Corp.Lib.Logging`)
- ✅ ASP.NET Core session with configurable timeout
- ✅ Antiforgery configured for iframe hosting (`X-Frame-Options` suppressed, `SameSite=None`)
- ✅ MVC Controllers + Views (or Razor Pages)
- ✅ JSON serialization (`PropertyNamingPolicy = null`)
- ✅ Shared session management
- ✅ Navigation history tracking (via client-side postMessage)
- ✅ Activity tracking with automatic session-expired overlay
- ✅ Navigation control
- ✅ Toast notifications
- ✅ Consistent portal styling

---

## Configuration Reference

### Complete Configuration Example

```json
{
  "VoyagerUxSettings": {
    "AppName": "My Application",
    "AppId": "my-app-id",
    "PortalBaseUrl": "https://localhost:7250",
    "StandaloneMode": false,
    "EnableClientLogging": false
  }
}
```

### Configuration Properties

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `AppName` | `string` | ✅ Yes | `"VoyagerUx Hosted App"` | Application name for tracking and logging |
| `PortalBaseUrl` | `string` | ✅ Yes* | `null` | Portal URL (*required unless `StandaloneMode = true`) |
| `AppId` | `string` | No | `null` | Custom application identifier for activity tracking |
| `StandaloneMode` | `bool` | No | `false` | Run without VoyagerUx features (development mode) |
| `EnableClientLogging` | `bool` | No | `false` | Enable JavaScript debug console logging |

### Environment-Specific Configuration

**appsettings.json** (base):
```json
{
  "VoyagerUxSettings": {
    "AppName": "My Application"
  }
}
```

**appsettings.Development.json**:
```json
{
  "VoyagerUxSettings": {
    "PortalBaseUrl": "https://localhost:7250",
    "EnableClientLogging": true
  }
}
```

**appsettings.Production.json**:
```json
{
  "VoyagerUxSettings": {
    "PortalBaseUrl": "https://mcsvoyager.specialty-web.com"
  }
}
```

### Configuration Validation

The package validates configuration on startup and throws `InvalidOperationException` if invalid:

| Error | Solution |
|-------|----------|
| `VoyagerUxSettings.AppName is required` | Add `AppName` to your configuration |
| `VoyagerUxSettings.PortalBaseUrl is required` | Add `PortalBaseUrl` or set `StandaloneMode: true` |

---

## Internally Configured Services

`AddVoyagerUxHostedApp` configures the following services automatically. Do not configure these separately.

### Serilog Logging & Session

Calls `Corp.Lib.Logging.AddLogging(true, sessionExpirationInMinutes)` internally, which:
- Configures Serilog as the logging provider
- Enables ASP.NET Core session with the specified idle timeout
- Registers the `Corp.Lib.Logging.Logger.Log` static Serilog instance

`UseVoyagerUxHostedApp` calls `Corp.Lib.Logging.UseLogging(true)` internally, which:
- Adds the Serilog request logging middleware
- Adds the ASP.NET Core session middleware

The `sessionExpirationInMinutes` parameter on `AddVoyagerUxHostedApp` controls the server-side session idle timeout and must match the portal's `SessionExpirationInMinutes` configuration value.

### Antiforgery for Iframe Hosting

Configures ASP.NET Core antiforgery for cross-origin iframe hosting:

| Setting | Value | Reason |
|---------|-------|--------|
| `SuppressXFrameOptionsHeader` | `true` | Allows the application to be loaded inside the portal's iframe |
| `Cookie.SameSite` | `None` | Required for cross-origin cookie access between portal and iframe |
| `Cookie.SecurePolicy` | `Always` | Required when `SameSite=None` (browsers reject insecure `None` cookies) |
| `Cookie.Expiration` | `120 minutes` | Set to the Citrix session timeout (2 hours) |

> **Note:** These antiforgery settings are critical for iframe operation. The `X-Frame-Options` header is suppressed so the browser allows the hosted app to render inside the portal iframe. The `SameSite=None` + `Secure` combination is required by modern browsers for cross-origin cookies.

---

## API Reference

### Shared Session Management

Share data across iframe applications using cookies. Both server-side (C#) and client-side (JavaScript) APIs are available with matching method names.

#### Server-Side API (C#)

Inject `SharedSessionManager` into your controller or page:

```csharp
using Corp.Web.VoyagerUx.HostedApp.Session;

public class HomeController : Controller
{
    private readonly SharedSessionManager _session;
    
    public HomeController(SharedSessionManager session)
    {
        _session = session;
    }
    
    public IActionResult Index()
    {
        // Set/Get string values
        _session.SetValue("userId", "12345");
        var userId = _session.GetValue("userId");
        
        // Set/Get objects (auto-serialized to JSON)
        _session.SetObject("user", new { Id = 123, Name = "John" });
        var user = _session.GetObject<UserModel>("user");
        
        // Check existence
        if (_session.HasValue("userId")) { }
        
        // Remove value
        _session.RemoveValue("userId");
        
        // Clear all VoyagerUx cookies
        _session.ClearAll();
        
        return View();
    }
}
```

#### Server-Side Method Reference

| Method | Description |
|--------|-------------|
| `SetValue(key, value, persistent)` | Store string value. `persistent=true` for 30-day expiry |
| `GetValue(key)` | Get string value or `null` |
| `SetObject<T>(key, value, persistent)` | Store object (JSON serialized) |
| `GetObject<T>(key)` | Get object (JSON deserialized) or `default` |
| `HasValue(key)` | Check if key exists |
| `RemoveValue(key)` | Remove a value |
| `ClearAll()` | Remove all VoyagerUx cookies |

#### Client-Side API (JavaScript)

```javascript
// Set/Get string values
VoyagerUxSession.setValue('username', 'john.doe');
var username = VoyagerUxSession.getValue('username');

// Set/Get objects (auto-serialized/deserialized)
VoyagerUxSession.setObject('user', { id: 123, name: 'John' });
var user = VoyagerUxSession.getObject('user');

// Check existence
if (VoyagerUxSession.hasValue('username')) { }

// Remove value
VoyagerUxSession.removeValue('username');

// Clear all
VoyagerUxSession.clearAll();

// Persistent storage (30 days instead of 60 minutes)
VoyagerUxSession.setValue('remember', 'value', true);
```

#### Client-Side Method Reference

| Method | Description |
|--------|-------------|
| `setValue(key, value, persistent)` | Store string value |
| `getValue(key)` | Get string value or `null` |
| `setObject(key, value, persistent)` | Store object (JSON serialized) |
| `getObject(key)` | Get object or `null` |
| `hasValue(key)` | Check if key exists |
| `removeValue(key)` | Remove a value |
| `clearAll()` | Remove all VoyagerUx cookies |

#### Cookie Details

| Property | Value |
|----------|-------|
| **Prefix** | `VoyagerUx_` |
| **Max Size** | 4KB per cookie |
| **Default Expiry** | 60 minutes |
| **Persistent Expiry** | 30 days |
| **SameSite** | `Lax` |
| **HttpOnly** | `false` (allows JavaScript access) |

---

### Navigation Control

Control parent portal navigation from iframe applications via `window.postMessage`.

#### Server-Side API (C#)

Inject `NavigationManager` to queue navigation commands:

```csharp
using Corp.Web.VoyagerUx.HostedApp.Navigation;

public class SettingsController : Controller
{
    private readonly NavigationManager _navigation;
    
    public SettingsController(NavigationManager navigation)
    {
        _navigation = navigation;
    }
    
    public IActionResult GoToUsers()
    {
        // Navigate by TreeView item ID
        _navigation.NavigateById(5);
        return View();
    }
    
    public IActionResult GoToPage()
    {
        // Navigate by URL with optional display label
        _navigation.NavigateToUrl("/admin/users", "User Management");
        return View();
    }
    
    public IActionResult GoHome()
    {
        // Navigate to home (clears TreeView selection)
        _navigation.NavigateHome();
        return View();
    }
    
    public IActionResult ExpandMenu()
    {
        // Expand/collapse TreeView nodes
        _navigation.ExpandNode(3);
        _navigation.CollapseNode(3);
        return View();
    }
}
```

#### Server-Side Method Reference

| Method | Description |
|--------|-------------|
| `NavigateById(navigationId)` | Navigate to TreeView item by ID |
| `NavigateToUrl(url, displayLabel?, selectTreeItem?)` | Navigate to URL |
| `NavigateHome()` | Navigate to home page |
| `ExpandNode(navigationId)` | Expand TreeView node |
| `CollapseNode(navigationId)` | Collapse TreeView node |
| `Refresh()` | Refresh the current iframe |

#### Client-Side API (JavaScript)

```javascript
// Navigate by TreeView item ID
VoyagerUxNavigation.navigateById(5);

// Navigate by URL
VoyagerUxNavigation.navigateToUrl('/admin/users');
VoyagerUxNavigation.navigateToUrl('/admin/users', { selectTreeItem: true });
VoyagerUxNavigation.navigateToUrl('/external', { openInNewTab: true });

// Navigate to home
VoyagerUxNavigation.navigateHome();

// Control TreeView
VoyagerUxNavigation.expandNode(3);
VoyagerUxNavigation.collapseNode(3);

// Refresh current iframe
VoyagerUxNavigation.refresh();

// Get navigation state (async)
VoyagerUxNavigation.getNavigationState(function(error, state) {
    if (!error) {
        console.log(state.selectedNavigationId);
    }
});

// Check if running in iframe
if (VoyagerUxNavigation.isInIframe()) {
    // Running inside VoyagerUx portal
}
```

#### Client-Side Method Reference

| Method | Description |
|--------|-------------|
| `navigateById(id)` | Navigate to TreeView item |
| `navigateToUrl(url, options?)` | Navigate to URL |
| `navigateHome()` | Navigate to home page |
| `expandNode(id)` | Expand TreeView node |
| `collapseNode(id)` | Collapse TreeView node |
| `refresh()` | Refresh current iframe |
| `getNavigationState(callback)` | Get current navigation state |
| `isInIframe()` | Check if running in iframe |

---

---

### Navigation History Tracking

Automatically track user navigation by adding `[TrackNavigationHistory]` to controller actions or Razor Page handlers. Tracking is performed via client-side `postMessage` to the parent portal.

#### How It Works

1. Add `[TrackNavigationHistory]` attribute to controller actions
2. The `AutoTrackNavigationHistoryFilter` populates metadata in `HttpContext.Items`
3. `<voyagerux-tracking-meta />` tag helper renders metadata to HTML
4. `voyagerux-navigation-history.js` reads metadata and sends to portal via `postMessage`
5. Parent portal handles the tracking API call

#### Basic Usage

```csharp
using Corp.Web.VoyagerUx.Lib.Attributes;

public class ClaimsController : Controller
{
    [TrackNavigationHistory("Claims List")]
    public IActionResult Index()
    {
        return View();
    }
    
    [TrackNavigationHistory("Claim Details")]
    public IActionResult Details(int id)
    {
        return View();
    }
}
```

#### Dynamic Labels with Interface

Implement `INavigationHistoryContextProvider` for runtime-generated labels:

```csharp
using Corp.Web.VoyagerUx.Lib.Attributes;
using Corp.Web.VoyagerUx.Lib.Interfaces;

public class ClaimsController : Controller, INavigationHistoryContextProvider
{
    public int ClaimId { get; set; }
    public string ClaimantName { get; set; }
    
    // Called automatically after action executes
    public string GetNavigationLabel() => $"Claim {ClaimId} - {ClaimantName}";
    
    [TrackNavigationHistory]  // No label needed - uses GetNavigationLabel()
    public IActionResult Details(int id)
    {
        ClaimId = id;
        ClaimantName = GetClaimantName(id);
        return View();
    }
}
```

#### Razor Pages Usage

Navigation history tracking works with Razor Page handlers:

```csharp
using Corp.Web.VoyagerUx.Lib.Attributes;
using Corp.Web.VoyagerUx.Lib.Interfaces;
using Microsoft.AspNetCore.Mvc.RazorPages;

public class DetailsModel : PageModel, INavigationHistoryContextProvider
{
    public int ClaimId { get; set; }
    public string ClaimantName { get; set; }
    
    public string GetNavigationLabel() => $"Claim {ClaimId} - {ClaimantName}";
    
    [TrackNavigationHistory]
    public void OnGet(int id)
    {
        ClaimId = id;
        ClaimantName = GetClaimantName(id);
    }
}
```

#### Attribute Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `DisplayLabel` | `string?` | `null` | Static label for history entry |
| `Enabled` | `bool` | `true` | Enable/disable tracking |

#### Requirements

- ✅ `PortalBaseUrl` must be configured
- ✅ `[TrackNavigationHistory]` attribute on action method
- ✅ `<voyagerux-tracking-meta />` in layout `<head>`
- ✅ Not in standalone mode
- ✅ User must be authenticated
- ✅ Action must return 2xx status code

**API Endpoint:** `POST {PortalBaseUrl}/api/navigationhistory/insert`

---

### Toast Notifications

Display notifications in the parent portal via `window.postMessage`.

#### Client-Side API (JavaScript)

```javascript
// Shorthand global functions
showToastSuccess('Data saved successfully!');
showToastError('An error occurred');
showToastWarning('Please review your input');
showToastInfo('Processing...');

// Custom display time (milliseconds)
showToastSuccess('Done!', 2000);
showToastError('Failed!', 6000);

// Object-oriented syntax
VoyagerUxToast.success('Saved!');
VoyagerUxToast.error('Failed!');
VoyagerUxToast.warning('Check your input');
VoyagerUxToast.info('Loading...');

// Custom type and duration
VoyagerUxToast.show('Custom message', 'info', 3000);
```

#### Method Reference

| Method | Default Duration | Description |
|--------|------------------|-------------|
| `success(message, displayTime?)` | 4000ms | Green success toast |
| `info(message, displayTime?)` | 4000ms | Blue info toast |
| `warning(message, displayTime?)` | 5000ms | Orange warning toast |
| `error(message, displayTime?)` | 6000ms | Red error toast |
| `show(message, type, displayTime?)` | 4000ms | Custom toast |

#### Global Shorthand Functions

| Function | Equivalent |
|----------|------------|
| `showToastSuccess(msg, time?)` | `VoyagerUxToast.success(msg, time)` |
| `showToastInfo(msg, time?)` | `VoyagerUxToast.info(msg, time)` |
| `showToastWarning(msg, time?)` | `VoyagerUxToast.warning(msg, time)` |
| `showToastError(msg, time?)` | `VoyagerUxToast.error(msg, time)` |

---

### Activity Tracking

Automatically tracks user activity to prevent session timeouts. Auto-initialized by `<voyagerux-scripts />` - no additional configuration needed.

#### How It Works

1. Script registers with parent portal on load
2. User interactions (clicks, mouse presses, keypresses, scroll, touch, focus) trigger activity pings
3. Parent portal receives pings and resets session timeout
4. When the portal signals session end or expiration, a blocking overlay is automatically shown inside the iframe
5. If the session is renewed, the overlay is removed and activity tracking resumes

#### Client-Side API (JavaScript)

```javascript
// Manual activity ping
IframeActivityTracker.ping();

// Get tracker status
var status = IframeActivityTracker.getStatus();
console.log(status.appName);        // Application name
console.log(status.appId);          // Application identifier
console.log(status.activityCount);  // Number of pings sent
console.log(status.isRegistered);   // Registration status
console.log(status.lastActivitySent);   // Last activity timestamp
```

---

### Session Timeout Warning

The VoyagerUx portal displays a countdown dialog before session expiration. This feature is configured via the portal's `IConfiguration` and works automatically with the activity tracking system.

#### How It Works

1. The portal reads `SessionExpirationInMinutes` and `SessionExpirationWarningInMinutes` from configuration
2. Each iframe-hosted application sends activity pings to the portal via `postMessage`
3. The portal's activity tracker sends heartbeats to the server, renewing the session
4. When there is no activity for `(SessionExpirationInMinutes - SessionExpirationWarningInMinutes)` minutes, a countdown dialog appears
5. The user can choose to **Continue Session** (renews the timeout) or **End Session** (clears all session data)
6. If the countdown reaches zero, the session expires and a full-page overlay is shown
7. Hosted iframes are automatically notified and display their own blocking overlay — **no hosted app code required**

#### Portal Configuration

The following configuration keys must be set on the **portal** application (not the hosted apps):

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `SessionExpirationInMinutes` | `int` | `60` | Total session timeout in minutes |
| `SessionExpirationWarningInMinutes` | `int` | `5` | Minutes before expiration to show the warning dialog |

These values come from `IConfiguration` (typically via `Corp.Api.Configuration.Lib`).

#### Portal Program.cs Session Wiring

The portal reads `SessionExpirationInMinutes` from configuration and passes it to `Corp.Lib.Logging.AddLogging` so the server-side ASP.NET session timeout matches the client-side countdown. Note that `AddConfigurationApiAndLoadConfigurations` must be called first to ensure the configuration value is available:

```csharp
// In the portal's Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.AddSecurityPermissionsLibrary();

builder.AddConfigurationApiAndLoadConfigurations();

var sessionExpirationInMinutes = builder.Configuration.GetValue("SessionExpirationInMinutes", 60);

builder.AddLogging(true, sessionExpirationInMinutes);
```

#### Hosted App Session Configuration

Each hosted application sets its session timeout via the `sessionExpirationInMinutes` parameter on `AddVoyagerUxHostedApp`, which passes it to `Corp.Lib.Logging.AddLogging` internally:

```csharp
// In the hosted application's Program.cs
builder.AddVoyagerUxHostedApp(
    configure: options =>
    {
        builder.Configuration.GetSection("VoyagerUxSettings").Bind(options);
    },
    sessionExpirationInMinutes: 60  // Must match the portal's SessionExpirationInMinutes value
);
```

> **Important:** All applications (portal and hosted apps) should use the same session timeout value to ensure consistent behavior. The portal handles the countdown dialog; individual hosted apps only need to keep their server-side session timeout in sync. Do not call `builder.AddLogging()` separately — it is called internally by `AddVoyagerUxHostedApp`.

#### JavaScript Events

The portal broadcasts session events to all registered iframes via `postMessage`. The `voyagerux-activity.js` script automatically converts these into local `window` `CustomEvent`s that your hosted application code can listen for:

| Event | Fired When | Description |
|-------|-----------|-------------|
| `voyagerux:session-warning` | Warning dialog appears | Session is about to expire; `detail` includes `expirationMinutes` and `warningMinutes` |
| `voyagerux:session-renewed` | User clicks "Continue Session" | Session has been renewed with a fresh timeout |
| `voyagerux:session-ended` | User clicks "End Session" | User chose to end the session |
| `voyagerux:session-expired` | Countdown reaches zero | No user action; session has expired |

```javascript
// Listen for session events in a hosted iframe application
window.addEventListener('voyagerux:session-warning', function (e) {
    console.log('Session expiring soon', e.detail);
});

window.addEventListener('voyagerux:session-expired', function () {
    console.log('Session has expired');
});

window.addEventListener('voyagerux:session-renewed', function (e) {
    console.log('Session renewed', e.detail);
});
```

> **Note:** All session lifecycle handling is automatic. Activity tracking pauses on session end/expire and resumes on renewal. A blocking overlay is shown inside the iframe when the session ends — **hosted applications do not need any code to handle session timeout**. The events above are available for optional custom behavior (e.g., canceling background operations).

#### Portal API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/session/config` | `GET` | Returns `{ expirationMinutes, warningMinutes }` |
| `/api/session/heartbeat` | `POST` | Renews session; body: `{ source }` |
| `/api/session/status` | `GET` | Returns session status and remaining time |
| `/api/session/end` | `POST` | Ends the session and clears all shared data |

---

## Tag Helpers

### VoyagerUx Scripts

Renders all required JavaScript files and initializes tracking.

```html
<voyagerux-scripts />
```

With optional attribute overrides:

```html
<voyagerux-scripts 
    app-name="My Custom App"
    app-id="my-app"
    enable-logging="true" />
```

#### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `app-name` | `string?` | Config value | Override application name |
| `app-id` | `string?` | Config value | Custom application ID |
| `enable-logging` | `bool?` | Config value | Enable JS debug logging |

#### Rendered Scripts

```html
<script src="/js/voyagerux-session.js"></script>
<script src="/js/voyagerux-navigation.js"></script>
<script src="/js/voyagerux-navigation-history.js"></script>
<script src="/js/voyagerux-activity.js"></script>
<script src="/js/voyagerux-toast.js"></script>
<script src="/js/voyagerux-init.js" data-voyagerux-config="..."></script>
```

### VoyagerUx Tracking Meta

Renders navigation tracking metadata for client-side postMessage tracking.

```html
<voyagerux-tracking-meta />
```

Place this in your `_Layout.cshtml` `<head>` section, before or after `<voyagerux-scripts />`.

When a controller action has the `[TrackNavigationHistory]` attribute, this tag helper outputs a meta tag containing tracking data that the JavaScript component reads and sends to the parent portal.

### Feature Tests

Interactive testing interface for all VoyagerUx features.

```html
<voyagerux-feature-tests />
```

With customization:

```html
<voyagerux-feature-tests 
    app-name="My Application" 
    accent-color="#00f584" 
    show-header="true" />
```

#### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `app-name` | `string?` | `"VoyagerUx Hosted App"` | Application name in header |
| `accent-color` | `string` | `"#00f584"` | Icon accent color |
| `show-header` | `bool` | `true` | Show welcome header section |

#### Test Cards Included

- 🔔 Toast Notifications - Test all toast types
- 💾 Shared Session - Test cookie read/write
- 📊 Activity Tracking - Monitor activity pings
- 🧭 Navigation Control - Test navigation commands

> **Note:** Requires Bootstrap 5 and Font Awesome for styling.

### App Configuration

Displays application configuration values for developer diagnostics.

```html
<voyagerux-appconfiguration />
```

With customization:

```html
<voyagerux-appconfiguration 
    title="App Settings"
    heading="Application Configuration"
    max-height="500"
    show-footer="true"
    accent-color="#00f584" />
```

#### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `title` | `string` | `"Configuration Values"` | Card title |
| `heading` | `string?` | `"Application Configuration"` | Section heading above card |
| `description` | `string` | (default text) | Description text above table |
| `max-height` | `int` | `400` | Max height of scrollable table in pixels |
| `show-footer` | `bool` | `true` | Show card footer |
| `accent-color` | `string` | `"#00f584"` | Icon accent color |

> **Warning:** This displays all configuration values including sensitive data. Do not expose to end users.

---

## Standalone Mode

For development outside the VoyagerUx portal:

```json
{
  "VoyagerUxSettings": {
    "AppName": "My Application",
    "StandaloneMode": true
  }
}
```

### What's Enabled/Disabled

| Feature | Standalone Mode |
|---------|----------------|
| Serilog logging | ✅ Enabled |
| ASP.NET Core session | ✅ Enabled |
| Antiforgery for iframe hosting | ✅ Enabled |
| MVC Controllers + Views | ✅ Enabled |
| Razor Pages | ✅ Enabled |
| JSON serialization options | ✅ Enabled |
| Shared session | ❌ Disabled |
| Navigation history | ❌ Disabled |
| Activity tracking | ❌ Disabled |
| Navigation control | ❌ Disabled |

### Handling Null Services

Use nullable injection for services when supporting standalone mode:

```csharp
public class HomeController : Controller
{
    private readonly NavigationManager? _navigation;
    private readonly SharedSessionManager? _session;
    
    public HomeController(
        NavigationManager? navigation = null,
        SharedSessionManager? session = null)
    {
        _navigation = navigation;
        _session = session;
    }
    
    public IActionResult Index()
    {
        // Safe null checks
        _navigation?.NavigateToUrl("/page");
        
        var userId = _session?.GetValue("userId");
        
        return View();
    }
}
```

---

## Styling

The package includes a pre-built CSS file that matches the VoyagerUx portal theme.

### Include the Stylesheet

Add to your `_Layout.cshtml`:

```html
<link rel="stylesheet" href="~/css/site.css" />
```

### Available CSS Classes

See the included `SITE-CSS-README.md` for comprehensive documentation of available styles including:

- **Layout** - Page headers, panels, toolbars, grid system
- **Components** - Cards, buttons, forms, tables, tabs, badges, alerts
- **DevExtreme** - Overrides for DataGrid, Tabs, TextEditors
- **Utilities** - Spacing, text, display, backgrounds, borders

### Color Variables

```css
:root {
    --fresh-mint: #00f584;      /* Primary accent */
    --mint-hover: #00d675;      /* Hover state */
    --mint-pale: #e6fff5;       /* Light backgrounds */
    --text-primary: #212121;    /* Main text */
    --text-secondary: #666666;  /* Secondary text */
    --btn-dark: #2d3748;        /* Button background */
}
```

---

## Namespaces

```csharp
using Corp.Web.VoyagerUx.HostedApp.Extensions;   // AddVoyagerUxHostedApp, VoyagerUxHostedAppOptions
using Corp.Web.VoyagerUx.HostedApp.Session;      // SharedSessionManager
using Corp.Web.VoyagerUx.HostedApp.Navigation;   // NavigationManager
using Corp.Web.VoyagerUx.Lib.Attributes;         // TrackNavigationHistoryAttribute
using Corp.Web.VoyagerUx.Lib.Interfaces;         // INavigationHistoryContextProvider, INavigationHistoryTracker
using Corp.Web.VoyagerUx.Lib.Filters;            // AutoTrackNavigationHistoryFilter
using Corp.Web.VoyagerUx.HostedApp.TagHelpers;   // Tag helpers (auto-registered)
```

---

## Requirements

| Requirement | Version/Notes |
|-------------|---------------|
| **.NET** | 10.0+ |
| **ASP.NET Core** | MVC or Razor Pages |
| **VoyagerUx Portal** | Required for iframe hosting (not in standalone mode) |
| **Bootstrap 5** | For feature test page styling (optional) |
| **Font Awesome** | For feature test page icons (optional) |

---

## Troubleshooting

### Application won't start

**Error:** `InvalidOperationException: VoyagerUxSettings.AppName is required.`

**Solution:** Add `VoyagerUxSettings` section to `appsettings.json`:

```json
{
  "VoyagerUxSettings": {
    "AppName": "My Application",
    "PortalBaseUrl": "https://localhost:7250"
  }
}
```

### Tag helpers not recognized

**Error:** `<voyagerux-scripts>` renders as literal text

**Solution:** Add tag helper reference to `_ViewImports.cshtml`:

```cshtml
@addTagHelper *, Corp.Web.VoyagerUx.HostedApp
```

### Navigation history not tracking

**Checklist:**
- ✅ `PortalBaseUrl` is configured
- ✅ `[TrackNavigationHistory]` attribute on action method
- ✅ `<voyagerux-tracking-meta />` in layout `<head>`
- ✅ Not in standalone mode
- ✅ User is authenticated
- ✅ Action returns 2xx status
- ✅ Running inside VoyagerUx portal iframe

### Types not found

**Error:** `CS0246: The type or namespace name 'SharedSessionManager' could not be found`

**Solution:** Add namespace imports:

```csharp
using Corp.Web.VoyagerUx.HostedApp.Session;
using Corp.Web.VoyagerUx.HostedApp.Navigation;
using Corp.Web.VoyagerUx.Lib.Attributes;
using Corp.Web.VoyagerUx.Lib.Interfaces;
using Corp.Web.VoyagerUx.HostedApp.Extensions;
```

### Scripts not loading

**Cause:** Missing `app.UseStaticFiles()` in `Program.cs`

**Solution:** Add before middleware registration:

```csharp
app.UseStaticFiles();  // Required for static web assets
```

### Session not shared between apps

**Causes:**
- Running in standalone mode
- Apps on different domains
- Cookies disabled in browser

**Solutions:**
- Ensure `StandaloneMode = false` in production
- Verify both apps are on same domain
- Check browser allows cookies

### Cookie size limit exceeded

**Error:** `InvalidOperationException: Cookie size (XXXX bytes) exceeds maximum allowed size (4000 bytes)`

**Solutions:**
- Store less data (IDs instead of full objects)
- Split across multiple cookies
- Use compact format (pipe-delimited instead of JSON)
- Store reference ID and fetch data from API when needed

### NullReferenceException on VoyagerUx services

**Cause:** Using services in standalone mode without null checks

**Solution:** Use nullable injection:

```csharp
public MyController(NavigationManager? navigation = null)
{
    _navigation = navigation;
}

// Use null-conditional operators
_navigation?.NavigateToUrl("/page");
```

### Toast not appearing

**Cause:** Not running inside iframe or parent doesn't handle messages

**Solution:** 
- Verify app is loaded in VoyagerUx portal iframe
- Check browser console for postMessage errors
- Toast falls back to console.log when not in iframe

---

## Package Information

| Property | Value |
|----------|-------|
| **Package ID** | `Corp.Web.VoyagerUx.HostedApp` |
| **Version** | `10.1.3` |
| **Target Framework** | `net10.0` |
| **License** | MIT |
| **Authors** | Mathew Hamilton |
| **Company** | Sedgwick.Consumer.Claims |
| **Repository** | Azure DevOps - IT-Specialty/Common |

### Static Web Assets

Scripts and CSS are copied to the consuming project's `wwwroot` directory via build targets:
```
wwwroot/js/
wwwroot/css/
```

### Dependencies

| Package | Version | Notes |
|---------|---------|-------|
| `Corp.Lib.Security.Permissions` | 10.2.1 | Required for session user resolution |
| `Corp.Web.VoyagerUx.Lib` | (bundled) | Included in package - provides attributes, interfaces, and filters |
| `Corp.Lib.Logging` | (transitive) | Configured internally by `AddVoyagerUxHostedApp` |
