# MudBlazorAssemb Project Documentation

## Project Overview

MudBlazorAssemb is a Blazor WebAssembly application built with MudBlazor UI components. It uses the hosted Blazor WebAssembly template with ASP.NET Core server and is deployed on Netlify.

## Technology Stack

- **.NET 9.0** - Target framework
- **Blazor WebAssembly** - Client-side UI framework
- **MudBlazor 8.x** - Material Design component library
- **ASP.NET Core 9.0** - Server host (for development)
- **Netlify** - Hosting and deployment platform

## Project Structure

```
MudBlazorAssemb/
├── MudBlazorAssemb/                    # Server project (for local development only)
│   ├── Components/                     # Server-side components (includes App.razor)
│   ├── wwwroot/                        # Server static files
│   │   ├── _redirects                  # Netlify redirect rules (optional)
│   │   └── favicon.ico                 # Favicon (copied to Client)
│   ├── Program.cs                      # Server startup (not used in production)
│   ├── appsettings.json                # Application settings
│   └── MudBlazorAssemb.csproj         # Server project file
│
├── MudBlazorAssemb.Client/            # Client project (DEPLOYED TO NETLIFY)
│   ├── Layout/                         # Layout components
│   │   ├── MainLayout.razor           # Main application layout
│   │   └── NavMenu.razor              # Navigation menu
│   ├── Pages/                          # Page components
│   │   ├── Home.razor                 # Home page
│   │   ├── Counter.razor              # Counter demo page
│   │   └── Weather.razor              # Weather demo page
│   ├── wwwroot/                        # Client static assets (CRITICAL FOR DEPLOYMENT)
│   │   ├── index.html                 # ⭐ Entry point (REQUIRED for static deployment)
│   │   ├── favicon.ico                # Favicon
│   │   ├── appsettings.json           # Runtime configuration
│   │   └── appsettings.Development.json
│   ├── Program.cs                      # Client startup and services
│   ├── Routes.razor                    # Routing configuration
│   ├── _Imports.razor                  # Global using directives
│   └── MudBlazorAssemb.Client.csproj  # Client project file (StaticWebAssetProjectMode=Root)
│
├── global.json                         # .NET SDK version specification
├── netlify.toml                        # Netlify build configuration (publishes Client project)
├── Deploy_Netlify_Guilde.md           # Deployment documentation
├── CLAUDE.md                           # This file
└── MudBlazorAssemb.sln                # Solution file
```

**Important Deployment Notes:**
- Only `MudBlazorAssemb.Client/` is published to Netlify as static files
- The Server project (`MudBlazorAssemb/`) is used for local development only
- `index.html` in Client/wwwroot is **critical** - without it, deployment will fail with "Page not found"

## Key Components

### Server Project (MudBlazorAssemb/)

**Program.cs (lines 1-39):**
- Configures ASP.NET Core host for Blazor WebAssembly
- Registers MudBlazor services (line 8)
- Adds Razor components with WebAssembly interactivity (lines 11-12)
- Maps static assets and Razor components (lines 33-36)
- Enables WebAssembly debugging in development (line 19)

### Client Project (MudBlazorAssemb.Client/)

**Program.cs (lines 1-8):**
- Minimal WebAssembly host configuration
- Registers MudBlazor services (line 6)
- Runs the WebAssembly application

**Layout Components:**
- `MainLayout.razor` - Defines the overall page structure
- `NavMenu.razor` - Navigation sidebar/menu

**Pages:**
- `Home.razor` - Landing page
- `Counter.razor` - Interactive counter demo
- `Weather.razor` - Sample data display page

## Configuration Files

### global.json

Specifies .NET SDK version for consistent builds:
```json
{
  "sdk": {
    "version": "9.0.100",
    "rollForward": "latestMinor"
  }
}
```

### netlify.toml

Netlify deployment configuration:
- **Build command:** Installs .NET 9 SDK and publishes the **Client** project
- **Publish directory:** `MudBlazorAssemb.Client/bin/Release/net9.0/publish/wwwroot`
- **Redirects:** Configures SPA routing for client-side navigation

```toml
[build]
publish = "MudBlazorAssemb.Client/bin/Release/net9.0/publish/wwwroot"
command = "... && cd MudBlazorAssemb.Client && dotnet publish -c Release"

[[redirects]]
from = "/*"
to = "/index.html"
status = 200
```

### index.html (Client/wwwroot)

**Critical file for static deployment.** The .NET 9 hosted template uses `App.razor` (server-rendered), which doesn't work for static CDN hosting. We create this file manually:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <base href="/" />
    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
    <!-- ... -->
</head>
<body>
    <div id="app">Loading...</div>
    <script src="_framework/blazor.webassembly.js"></script>
    <script src="_content/MudBlazor/MudBlazor.min.js"></script>
</body>
</html>
```

### Client.csproj Configuration

The Client project must be configured for standalone publishing:

```xml
<StaticWebAssetProjectMode>Root</StaticWebAssetProjectMode>
```

This ensures all necessary files (including `index.html`) are included in the publish output.

## Development Workflow

### Local Development

1. **Run the application:**
   ```bash
   cd MudBlazorAssemb
   dotnet watch run
   ```
   Opens at `https://localhost:5001`

2. **Hot reload:**
   - `.razor` file changes trigger automatic reload
   - CSS changes reflect immediately

### Building

1. **Debug build:**
   ```bash
   dotnet build
   ```

2. **Release build for Netlify (Client project only):**
   ```bash
   cd MudBlazorAssemb.Client
   dotnet publish -c Release
   ```
   Output: `bin/Release/net9.0/publish/wwwroot/`

   **Verify index.html exists:**
   ```bash
   ls -la bin/Release/net9.0/publish/wwwroot/index.html
   ```

### Testing

Run tests (when added):
```bash
dotnet test
```

## Deployment

### Automatic Deployment (Git-based)

Every push to `main` branch triggers:
1. Netlify pulls latest code
2. Installs .NET 9 SDK (via dotnet-install.sh)
3. Runs `dotnet publish -c Release` on the **Client** project
4. Deploys static files from `MudBlazorAssemb.Client/bin/Release/net9.0/publish/wwwroot/` to CDN

**Critical:** The deployment publishes the Client project, not the Server project. The output must include `index.html`.

### Manual Deployment

Using Netlify CLI:
```bash
cd MudBlazorAssemb.Client
dotnet publish -c Release
netlify deploy --prod --dir=bin/Release/net9.0/publish/wwwroot
```

See `Deploy_Netlify_Guilde.md` for detailed deployment instructions.

### Deployment Architecture

```
Local Development:
  MudBlazorAssemb (Server) → Hosts → MudBlazorAssemb.Client
  Run with: dotnet watch run (in Server project)

Production (Netlify):
  Static CDN → MudBlazorAssemb.Client wwwroot (standalone)
  Server project is NOT deployed
```

## MudBlazor Integration

### Service Registration

MudBlazor services are registered in both projects:
- **Server:** `MudBlazorAssemb/Program.cs:8`
- **Client:** `MudBlazorAssemb.Client/Program.cs:6`

### Usage

MudBlazor components are used throughout the application:
```razor
<MudButton Variant="Variant.Filled" Color="Color.Primary">
    Click Me
</MudButton>
```

### Theming

Default MudBlazor theme is applied in `MainLayout.razor`.

## Important Notes

### Render Modes

This project uses **InteractiveWebAssembly** render mode:
- Client-side execution in the browser
- Full .NET runtime via WebAssembly
- No server connection after initial load

### Static Assets

- Server static files: `MudBlazorAssemb/wwwroot/`
- Client static files: `MudBlazorAssemb.Client/wwwroot/`
- Both are merged during publish

### Base Path

Default base path is `/`. If deploying to a subdirectory, update `<base>` tag in `index.html`.

## Common Tasks

### Add a New Page

1. Create `Pages/NewPage.razor` in Client project
2. Add `@page "/newpage"` directive
3. Page is automatically available at `/newpage`

### Add MudBlazor Component

1. MudBlazor is globally imported via `_Imports.razor`
2. Use components directly without additional imports
3. Example: `<MudCard>...</MudCard>`

### Add a Service

1. Define interface and implementation
2. Register in `Program.cs`:
   ```csharp
   builder.Services.AddScoped<IMyService, MyService>();
   ```

### Update Dependencies

```bash
dotnet add package PackageName
# or specify version
dotnet add package PackageName --version 1.2.3
```

## Troubleshooting

### Build Errors

- Check .NET SDK version: `dotnet --version`
- Clean and rebuild: `dotnet clean && dotnet build`
- Clear obj/bin folders if needed

### Runtime Errors

- Check browser console (F12)
- Verify all static assets are present
- Ensure base path is correct

### Deployment Issues

**"Page not found" error on Netlify:**
- **Cause:** Missing `index.html` in the deployed files
- **Solution:**
  1. Verify `MudBlazorAssemb.Client/wwwroot/index.html` exists
  2. Check `Client.csproj` has `<StaticWebAssetProjectMode>Root</StaticWebAssetProjectMode>`
  3. Ensure `netlify.toml` publishes the **Client** project, not Server
  4. Rebuild and verify `index.html` is in publish output

**Build errors:**
- Verify `global.json` and `netlify.toml` are committed
- Check Netlify build logs for .NET SDK installation errors
- Ensure rollForward is set to "latestMinor" in `global.json`

**Routing issues:**
- Check `[[redirects]]` section in `netlify.toml`
- Verify base path is "/" in `index.html`

## Resources

- [Blazor Documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/)
- [MudBlazor Documentation](https://mudblazor.com/)
- [MudBlazor GitHub](https://github.com/MudBlazor/MudBlazor)
- [Deployment Guide](./Deploy_Netlify_Guilde.md)

## Development Environment

- **IDE:** Visual Studio Code or Visual Studio 2022
- **Required:** .NET 9.0 SDK
- **Recommended Extensions (VS Code):**
  - C# Dev Kit
  - C#
  - Blazor WASM Debugging

## Project History

- Created with .NET 9.0 Blazor WebAssembly **hosted** template
- Integrated MudBlazor 8.x for UI components
- Configured for Netlify deployment with automatic .NET SDK installation
- **Fixed:** Added `index.html` and configured Client project for standalone static deployment
  - .NET 9 hosted template uses `App.razor` (server-rendered), incompatible with static CDN
  - Solution: Created `index.html` in Client/wwwroot and set `StaticWebAssetProjectMode=Root`
  - Updated `netlify.toml` to publish Client project instead of Server
- Added comprehensive deployment documentation

## Maintenance

### Updating .NET Version

1. Update `global.json`
2. Update `TargetFramework` in `.csproj` files
3. Update package versions if needed
4. Test locally before deploying

### Updating MudBlazor

```bash
cd MudBlazorAssemb.Client
dotnet add package MudBlazor --version <new-version>
```

Check [MudBlazor releases](https://github.com/MudBlazor/MudBlazor/releases) for breaking changes.

## Future Enhancements

Potential improvements:
- Add authentication/authorization
- Implement state management (Fluxor, Redux)
- Add unit and integration tests
- Implement PWA features
- Add internationalization (i18n)
- Integrate with backend API
- Add more MudBlazor components and themes

## Contact & Support

For issues or questions:
- Check existing documentation in this repository
- Review [MudBlazor documentation](https://mudblazor.com/)
- Consult [Blazor documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/)
