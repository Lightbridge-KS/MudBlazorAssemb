# Deploying .NET Blazor WebAssembly with MudBlazor to Netlify

This guide covers how to deploy a Blazor WebAssembly application using MudBlazor to Netlify.

## Prerequisites

- .NET 9.0 SDK installed locally
- A Blazor WebAssembly project with MudBlazor
- Git repository (GitHub recommended)
- Netlify account

## Project Structure

This project uses the hosted Blazor WebAssembly template, configured for standalone static deployment:
- `MudBlazorAssemb/` - Server project (for local development only)
- `MudBlazorAssemb.Client/` - Client project (Blazor WebAssembly with MudBlazor) - **This is what gets deployed**

**Important:** Only the Client project is published to Netlify as static files. The Server project is not used in production.

## Configuration Files

### 1. `global.json` (Project Root)

Create a `global.json` file in your project root to specify the .NET SDK version:

```json
{
  "sdk": {
    "version": "9.0.100",
    "rollForward": "latestMinor"
  }
}
```

**Why `rollForward`?**
- Allows Netlify to use the latest minor version of .NET 9.0.x
- Makes builds more flexible in CI/CD environments
- Prevents build failures due to exact version unavailability

### 2. `netlify.toml` (Project Root)

Create a `netlify.toml` file to configure the build and deployment:

```toml
[build]
publish = "MudBlazorAssemb.Client/bin/Release/net9.0/publish/wwwroot"
command = "curl -sSL https://dot.net/v1/dotnet-install.sh -o /tmp/dotnet-install.sh && bash /tmp/dotnet-install.sh --channel 9.0 --install-dir $HOME/.dotnet && export PATH=$HOME/.dotnet:$PATH && dotnet --info && cd MudBlazorAssemb.Client && dotnet publish -c Release"

[[redirects]]
from = "/*"
to = "/index.html"
status = 200
```

**Explanation:**
- `publish`: Points to the **Client** project's output directory containing static files
- `command`: Installs .NET 9 SDK and builds the **Client** project as standalone WebAssembly
- `redirects`: Ensures client-side routing works (required for Blazor SPA)

### 3. `index.html` (Client Project wwwroot)

Create `MudBlazorAssemb.Client/wwwroot/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>MudBlazorAssemb</title>
    <base href="/" />
    <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" rel="stylesheet" />
    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
    <link rel="icon" type="image/x-icon" href="favicon.ico" />
</head>
<body>
    <div id="app">Loading...</div>
    <script src="_framework/blazor.webassembly.js"></script>
    <script src="_content/MudBlazor/MudBlazor.min.js"></script>
</body>
</html>
```

**Critical:** This file is **required** for static deployment. The .NET 9 hosted template uses `App.razor` which only works with a server, not for static CDN hosting.

### 4. Client Project Configuration

Update `MudBlazorAssemb.Client/MudBlazorAssemb.Client.csproj`:

```xml
<PropertyGroup>
  <StaticWebAssetProjectMode>Root</StaticWebAssetProjectMode>
</PropertyGroup>
```

This setting enables standalone publishing mode, ensuring all necessary files (including `index.html`) are included in the publish output.

### 5. `_redirects` (Server Project wwwroot - Optional)

Create `MudBlazorAssemb/wwwroot/_redirects`:

```
/*    /index.html   200
```

This file ensures all routes are handled by Blazor's client-side router.

## Build Process Explained

The build command in `netlify.toml` performs these steps:

1. **Download .NET installer script**
   ```bash
   curl -sSL https://dot.net/v1/dotnet-install.sh -o /tmp/dotnet-install.sh
   ```

2. **Install .NET 9 SDK**
   ```bash
   bash /tmp/dotnet-install.sh --channel 9.0 --install-dir $HOME/.dotnet
   ```

3. **Add SDK to PATH**
   ```bash
   export PATH=$HOME/.dotnet:$PATH
   ```

4. **Verify installation**
   ```bash
   dotnet --info
   ```

5. **Build and publish Client project**
   ```bash
   cd MudBlazorAssemb.Client && dotnet publish -c Release
   ```

## Deployment Methods

### Option 1: Git-based Deployment (Recommended)

1. **Commit all configuration files:**
   ```bash
   git add global.json netlify.toml MudBlazorAssemb.Client/wwwroot/index.html MudBlazorAssemb.Client/MudBlazorAssemb.Client.csproj
   git commit -m "Add Netlify deployment configuration for standalone WebAssembly"
   ```

2. **Push to GitHub:**
   ```bash
   git push origin main
   ```

3. **Connect to Netlify:**
   - Go to [Netlify Dashboard](https://app.netlify.com)
   - Click "Add new site" � "Import an existing project"
   - Connect to GitHub and select your repository
   - Netlify will auto-detect settings from `netlify.toml`
   - Click "Deploy site"

4. **Continuous Deployment:**
   - Every push to `main` branch triggers automatic rebuilds
   - Changes are live within minutes

### Option 2: Netlify CLI

1. **Install Netlify CLI:**
   ```bash
   npm install -g netlify-cli
   ```

2. **Login to Netlify:**
   ```bash
   netlify login
   ```

3. **Build and deploy:**
   ```bash
   cd MudBlazorAssemb.Client
   dotnet publish -c Release
   netlify deploy --prod --dir=bin/Release/net9.0/publish/wwwroot
   ```

## Testing Locally

Before deploying, test your build locally:

```bash
# Build the Client project
cd MudBlazorAssemb.Client
dotnet publish -c Release

# Navigate to the publish output
cd bin/Release/net9.0/publish/wwwroot

# Verify index.html exists
ls -la index.html

# Serve the static files (using Python)
python3 -m http.server 8080

# Or using dotnet-serve
dotnet tool install -g dotnet-serve
dotnet serve -d . -p 8080
```

Open `http://localhost:8080` to verify the app works. You should see your MudBlazor application load correctly.

## Troubleshooting

### Build fails with NETSDK1045 error

**Problem:** Netlify's default .NET SDK is version 8.x, but project targets .NET 9.

**Solution:** Ensure `global.json` and updated `netlify.toml` are committed and pushed.

### "Page not found" or missing index.html

**Problem:** Deployment succeeds but shows "Page not found". The published files are missing `index.html`.

**Root Cause:** The .NET 9 hosted Blazor template uses `App.razor` (server-rendered) instead of static `index.html`. This only works with a server, not static CDN hosting.

**Solution:**
1. Create `MudBlazorAssemb.Client/wwwroot/index.html` (see configuration section above)
2. Update `MudBlazorAssemb.Client.csproj` to set `<StaticWebAssetProjectMode>Root</StaticWebAssetProjectMode>`
3. Update `netlify.toml` to publish the **Client** project, not the Server project
4. Rebuild and verify `index.html` is in the publish output

### 404 errors on page refresh

**Problem:** Missing redirect configuration.

**Solution:** Verify `[[redirects]]` section is in `netlify.toml`. The `_redirects` file in Server project is optional and not used in Netlify deployment.

### Build succeeds but app shows blank page

**Check:**
1. Browser console for errors
2. Ensure base path in `index.html` is set to `/`
3. Verify all static assets are in the publish output

### Long build times

The first build takes longer due to .NET SDK installation (2-3 minutes). Subsequent builds are faster as Netlify caches dependencies.

## Post-Deployment

### Custom Domain

1. Go to your site in Netlify Dashboard
2. Navigate to "Domain settings"
3. Click "Add custom domain"
4. Follow DNS configuration instructions

### Environment Variables

If your app needs environment variables:

1. Go to "Site settings" � "Environment variables"
2. Add variables as key-value pairs
3. Access in code via configuration

### Performance Optimization

- Enable Netlify's compression (enabled by default)
- Use Netlify CDN for global distribution
- Consider enabling Netlify Analytics for insights

## Resources

- [Netlify Documentation](https://docs.netlify.com/)
- [Blazor WebAssembly Documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models#blazor-webassembly)
- [MudBlazor Documentation](https://mudblazor.com/)
- [.NET Install Script](https://dot.net/v1/dotnet-install.sh)

## Summary

Key files for Netlify deployment:
- `global.json` - Pins .NET SDK version with rollForward support
- `netlify.toml` - Build configuration (publishes Client project)
- `MudBlazorAssemb.Client/wwwroot/index.html` - **Required** entry point for static deployment
- `MudBlazorAssemb.Client.csproj` - Must set `StaticWebAssetProjectMode` to `Root`

Once configured, deployment is automatic on every git push to main branch.

**Key Difference from Standard Template:**
The .NET 9 hosted Blazor template uses `App.razor` which requires a server. For static Netlify deployment, we create `index.html` and configure the Client project for standalone publishing.
