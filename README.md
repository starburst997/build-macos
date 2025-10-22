# Build macOS Application

A GitHub composite action to build and sign macOS applications using fastlane. This action provides a flexible, generic solution that handles npm dependencies, CocoaPods, Xcode builds, code signing with fastlane match, and optional ZIP packaging.

## Features

- **Automatic Detection**: Automatically detects and installs npm, Bundler, and CocoaPods dependencies
- **Flexible Configuration**: Customizable paths for workspace, project, scheme, and output
- **Code Signing**: Optional code signing with fastlane match and Developer ID
- **Notarization**: Optional notarization for signed builds
- **Caching**: Built-in caching for npm, CocoaPods, and Xcode DerivedData
- **ZIP Creation**: Optional ZIP archive creation for distribution
- **Generic Design**: Reusable across different macOS projects

## Prerequisites

For **unsigned builds** (testing only):

- A macOS Xcode project with workspace and scheme

For **signed builds** (distribution):

- Apple Developer Program enrollment (~$99/year)
- Code signing certificates set up with fastlane match (see [starburst997/apple-setup](https://github.com/starburst997/apple-setup))
- App Store Connect API key OR Apple ID session

## Quick Start

### Unsigned Build (Testing)

Build an unsigned app for local testing:

```yaml
name: Build macOS App
on: push

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build macOS App
        uses: starburst997/build-macos@v1
        with:
          workspace_path: "macos/MyApp.xcworkspace"
          project_path: "macos/MyApp.xcodeproj"
          scheme: "MyApp-macOS"
          output_name: "MyApp"
          mac_bundle_id: "com.company.myapp"
          build_signed: false
```

### Signed Build with Developer ID

Build and sign an app for distribution outside the Mac App Store:

```yaml
name: Build macOS App
on: push

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build macOS App
        uses: starburst997/build-macos@v1
        with:
          # Project configuration
          workspace_path: "macos/MyApp.xcworkspace"
          project_path: "macos/MyApp.xcodeproj"
          scheme: "MyApp-macOS"
          output_name: "MyApp"
          mac_bundle_id: "com.company.myapp"

          # Build configuration
          build_signed: true
          export_method: "developer-id"
          create_zip: true

          # Apple credentials
          apple_connect_email: ${{ secrets.APPLE_CONNECT_EMAIL }}
          apple_developer_email: ${{ secrets.APPLE_DEVELOPER_EMAIL }}
          apple_team_id: ${{ secrets.APPLE_TEAM_ID }}

          # Match configuration
          match_repository: ${{ secrets.MATCH_REPOSITORY }}
          match_password: ${{ secrets.MATCH_PASSWORD }}
          match_deploy_key: ${{ secrets.MATCH_DEPLOY_KEY }}

          # App Store Connect API
          appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
          appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
          appstore_p8: ${{ secrets.APPSTORE_P8 }}
```

### React Native App Example

Build a React Native macOS app with npm and CocoaPods:

```yaml
- name: Build React Native macOS App
  uses: starburst997/build-macos@v1
  with:
    working_directory: "sample"
    workspace_path: "macos/MyApp.xcworkspace"
    project_path: "macos/MyApp.xcodeproj"
    scheme: "MyApp-macOS"
    cocoapods_directory: "macos"
    output_name: "MyApp"
    mac_bundle_id: "com.company.myapp"
    build_signed: true
    create_zip: true
    # ... add credentials as above
```

## Inputs

### Required Inputs

| Input            | Description                                               |
| ---------------- | --------------------------------------------------------- |
| `workspace_path` | Path to .xcworkspace file (relative to working_directory) |
| `project_path`   | Path to .xcodeproj file (relative to working_directory)   |
| `scheme`         | Xcode scheme to build                                     |
| `output_name`    | Output app name (without .app extension)                  |
| `mac_bundle_id`  | macOS application bundle identifier                       |

### Project Configuration

| Input               | Default | Description                              |
| ------------------- | ------- | ---------------------------------------- |
| `working_directory` | `.`     | Working directory containing the project |

### Node.js/npm Configuration

| Input                       | Default             | Description                           |
| --------------------------- | ------------------- | ------------------------------------- |
| `node_version`              | `20`                | Node.js version to use                |
| `enable_npm`                | `auto`              | Enable npm (`auto`, `true`, `false`)  |
| `npm_cache_dependency_path` | `package-lock.json` | Path to package-lock.json for caching |

### Ruby/Bundler Configuration

| Input            | Default | Description                              |
| ---------------- | ------- | ---------------------------------------- |
| `ruby_version`   | `3.2`   | Ruby version to use                      |
| `enable_bundler` | `auto`  | Enable bundler (`auto`, `true`, `false`) |

### CocoaPods Configuration

| Input                    | Default | Description                                |
| ------------------------ | ------- | ------------------------------------------ |
| `enable_cocoapods`       | `auto`  | Enable CocoaPods (`auto`, `true`, `false`) |
| `cocoapods_directory`    | `macos` | Directory containing Podfile               |
| `enable_cocoapods_cache` | `true`  | Enable caching for CocoaPods               |

### Xcode Configuration

| Input                | Default | Description                          |
| -------------------- | ------- | ------------------------------------ |
| `enable_xcode_cache` | `true`  | Enable caching for Xcode DerivedData |

### Build Configuration

| Input              | Default        | Description                                       |
| ------------------ | -------------- | ------------------------------------------------- |
| `build_signed`     | `true`         | Build with code signing                           |
| `export_method`    | `developer-id` | Export method (`developer-id`, `mac-application`) |
| `output_directory` | `dist`         | Output directory for built app                    |

### ZIP Configuration

| Input        | Default | Description                                |
| ------------ | ------- | ------------------------------------------ |
| `create_zip` | `false` | Create a ZIP archive of the built app      |
| `zip_name`   | `""`    | Name of ZIP file (defaults to output_name) |

### Notarization Configuration

| Input                 | Default | Description                           |
| --------------------- | ------- | ------------------------------------- |
| `enable_notarization` | `true`  | Enable notarization for signed builds |

### Apple Credentials (Required for Signed Builds)

| Input                   | Description                            |
| ----------------------- | -------------------------------------- |
| `apple_connect_email`   | Apple Connect email (for notarization) |
| `apple_developer_email` | Apple Developer email                  |
| `apple_team_id`         | Apple Team ID                          |

### Match Configuration (Required for Signed Builds)

| Input              | Description                                |
| ------------------ | ------------------------------------------ |
| `match_repository` | Git repository for certificates (org/repo) |
| `match_password`   | Password to encrypt/decrypt certificates   |
| `match_deploy_key` | SSH deploy key for match repository        |

### App Store Connect API (Required for Signed Builds if use_session is false)

| Input                | Description                       |
| -------------------- | --------------------------------- |
| `appstore_issuer_id` | App Store Connect API issuer ID   |
| `appstore_key_id`    | App Store Connect API key ID      |
| `appstore_p8`        | App Store Connect API private key |

### Session Auth (Alternative to API Key)

| Input               | Description                                      |
| ------------------- | ------------------------------------------------ |
| `use_session`       | Use Apple ID session auth (default: `false`)     |
| `fastlane_user`     | Apple ID email (if use_session is true)          |
| `fastlane_password` | Apple ID password (if use_session is true)       |
| `fastlane_session`  | Fastlane session cookie (if use_session is true) |

## Outputs

| Output     | Description                              |
| ---------- | ---------------------------------------- |
| `app_path` | Path to the built .app bundle            |
| `zip_path` | Path to ZIP file (if create_zip is true) |

## Usage Examples

### Simple Native macOS App

```yaml
- name: Build macOS App
  uses: starburst997/build-macos@v1
  with:
    workspace_path: "MyApp.xcworkspace"
    project_path: "MyApp.xcodeproj"
    scheme: "MyApp"
    output_name: "MyApp"
    mac_bundle_id: "com.company.myapp"
    build_signed: false
```

### With Custom Working Directory

```yaml
- name: Build macOS App
  uses: starburst997/build-macos@v1
  with:
    working_directory: "apps/desktop"
    workspace_path: "macos/MyApp.xcworkspace"
    project_path: "macos/MyApp.xcodeproj"
    scheme: "MyApp-macOS"
    output_name: "MyApp"
    mac_bundle_id: "com.company.myapp"
    build_signed: false
```

### With ZIP and Custom Name

```yaml
- name: Build macOS App
  uses: starburst997/build-macos@v1
  with:
    workspace_path: "MyApp.xcworkspace"
    project_path: "MyApp.xcodeproj"
    scheme: "MyApp"
    output_name: "MyApp"
    mac_bundle_id: "com.company.myapp"
    build_signed: false
    create_zip: true
    zip_name: "MyApp-v1.0.0-macOS"
```

### Disable Caching

```yaml
- name: Build macOS App
  uses: starburst997/build-macos@v1
  with:
    workspace_path: "MyApp.xcworkspace"
    project_path: "MyApp.xcodeproj"
    scheme: "MyApp"
    output_name: "MyApp"
    mac_bundle_id: "com.company.myapp"
    enable_cocoapods_cache: false
    enable_xcode_cache: false
    build_signed: false
```

### Using Outputs

```yaml
- name: Build macOS App
  id: build
  uses: starburst997/build-macos@v1
  with:
    workspace_path: "MyApp.xcworkspace"
    project_path: "MyApp.xcodeproj"
    scheme: "MyApp"
    output_name: "MyApp"
    mac_bundle_id: "com.company.myapp"
    build_signed: true
    create_zip: true

- name: Upload to Release
  uses: actions/upload-artifact@v4
  with:
    name: macOS-app
    path: ${{ steps.build.outputs.zip_path }}
```

## Setting Up Code Signing

To use signed builds, you need to set up code signing certificates with fastlane match. This action works in conjunction with the [starburst997/apple-setup](https://github.com/starburst997/apple-setup) action which handles certificate generation and management.

### Prerequisites

Before using this action for signed builds, you need:

1. **Apple Developer Program** enrollment (~$99/year)
2. **Code signing certificates** set up using the [apple-setup](https://github.com/starburst997/apple-setup) action
3. **App Store Connect API key** OR **Apple ID session** credentials

### Complete Setup Guide

Follow these steps to set up all required secrets for signed builds:

#### Step 1: Create App in App Store Connect (Optional)

<table align="center"><tr><td>
<a href="docs/images/create_app_01.png" target="_blank"><img src="docs/images/create_app_01.png" alt="New App" title="New App" width="250" /></a><p align="center">1</p>
</td><td>
<a href="docs/images/create_app_02.png" target="_blank"><img src="docs/images/create_app_02.png" alt="New App Form" title="New App Form" width="250" /></a><p align="center">2</p>
</td><td>
<a href="docs/images/create_app_03.png" target="_blank"><img src="docs/images/create_app_03.png" alt="Create Bundle ID" title="Create Bundle ID" width="250" /></a><p align="center">3</p>
</td></tr></table>

If you haven't already created your app:

1. Register your app in [App Store Connect](https://appstoreconnect.apple.com/apps), click on the **(+)** button and select **New App**

2. You'll be asked to pick a **Bundle ID** which can be created in the **Apple Developer** website under the [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/identifiers/bundleId/add/bundle) section

3. Remember its value and pick the **Explicit** option (e.g., `com.company.appname`)

**Note:** The bundle ID is provided inline in your workflow YAML (e.g., `mac_bundle_id: "com.company.appname"`), not as a secret.

#### Step 2: Create a Match Repository

1. Create a new **private** GitHub repository (e.g., `myorg/ios-certificates`)
2. This repository will store your encrypted certificates and provisioning profiles
3. Keep this repository **private** to protect your signing certificates
4. The repository can be empty - fastlane match will populate it automatically

**Secret to save:** `MATCH_REPOSITORY` - Format: `org/repo` (e.g., `myorg/ios-certificates`)

#### Step 3: Create Match Password

1. Generate a strong random password (use your password manager)
2. This password encrypts/decrypts your certificates in the match repository
3. Save it securely - you'll need it for future projects or manual certificate access

**Secret to save:** `MATCH_PASSWORD` - Your strong encryption password

#### Step 4: Generate App Store Connect API Key

<table align="center"><tr><td>
<a href="docs/images/api_key_01.png" target="_blank"><img src="docs/images/api_key_01.png" alt="New API Key" title="New API Key" width="400" /></a><p align="center">1</p>
</td><td>
<a href="docs/images/api_key_02.png" target="_blank"><img src="docs/images/api_key_02.png" alt="New API Key form" title="New API Key form" width="400" /></a><p align="center">2</p>
</td></tr></table>

App Store Connect API Key is the recommended way to authenticate with Apple services.

1. Go to your [Users and Access](https://appstoreconnect.apple.com/access/integrations/api) page on App Store Connect

   - Click the **Integrations** tab
   - Select **App Store Connect API** on the left panel
   - Select the **Team Keys** tab
   - **Note the Issuer ID** at the top of the page

2. Create a new Key:
   - Click the **(+)** button
   - Give it a name (e.g., "GitHub Actions")
   - Select **Admin** access (required for certificate management)
   - Click **Generate**
   - **Download the .p8 file immediately** (you can only download it once!)
   - **Note the Key ID** shown in the key's row

**Secrets to save:**

- `APPSTORE_ISSUER_ID` - The Issuer ID from the API page
- `APPSTORE_KEY_ID` - The Key ID of your generated key
- `APPSTORE_P8` - The **entire contents** of the `.p8` file (including BEGIN/END lines)

#### Step 5: Create GitHub Personal Access Token

<table align="center"><tr><td>
<a href="docs/images/pat_01.png" target="_blank"><img src="docs/images/pat_01.png" alt="Generate New Token (Classic)" title="Generate New Token (Classic)" width="400" /></a><p align="center">1</p>
</td><td>
<a href="docs/images/pat_02.png" target="_blank"><img src="docs/images/pat_02.png" alt="Use repo scope" title="Use repo scope" width="400" /></a><p align="center">2</p>
</td></tr></table>

**Note:** This is only needed if you're using the [apple-setup](https://github.com/starburst997/apple-setup) action to generate certificates. Skip this if certificates are already set up.

1. Visit your GitHub [Personal Access Tokens](https://github.com/settings/tokens) settings
2. Click **Generate new token** → Select **Generate new token (classic)**
3. Give it a descriptive name (e.g., "Fastlane Match - Project Name")
4. Select scope: **repo** (Full control of private repositories)
5. Set expiration to **No expiration** (or manage renewals yourself)
6. Click **Generate token**
7. **Copy the token immediately** (you won't see it again!)

**Secret to save:** `GH_PAT` - Your generated GitHub Personal Access Token (only for apple-setup action)

#### Step 6: Get Additional Apple Credentials

Retrieve these values from your Apple Developer account:

1. **Apple Developer Email**: Your Apple ID email
2. **Apple Connect Email**: Apple Connect email (usually same as above)
3. **Apple Team ID**: From your [Apple Developer Account - Membership Details](https://developer.apple.com/account/#/membership/)

**Secrets to save:**

- `APPLE_DEVELOPER_EMAIL` - Your Apple ID
- `APPLE_CONNECT_EMAIL` - Apple Connect email (same as above if using a single developer account)
- `APPLE_TEAM_ID` - Team ID from your Apple Developer account

#### Step 7: Run Apple Setup Action (First Time Only)

Before using this build action, you must first generate and store your certificates using the [apple-setup](https://github.com/starburst997/apple-setup) action:

```yaml
name: Apple Setup
on: workflow_dispatch

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Apple Code Signing
        uses: starburst997/apple-setup@v1
        with:
          generate_macos: true
          generate_developer_id: true
          gh_pat: ${{ secrets.GH_PAT }}
          match_repository: ${{ secrets.MATCH_REPOSITORY }}
          match_password: ${{ secrets.MATCH_PASSWORD }}
          appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
          appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
          appstore_p8: ${{ secrets.APPSTORE_P8 }}
          mac_bundle_id: "com.company.appname"
```

Run this workflow **once** to generate and store your certificates in the match repository. It will also save the `MATCH_DEPLOY_KEY` secret to your repository automatically.

### Required Secrets Summary

After completing all steps above, you should have these secrets in your GitHub repository:

**Required for signed builds:**

- ✅ `APPLE_CONNECT_EMAIL` - Apple Connect email (for notarization)
- ✅ `APPLE_DEVELOPER_EMAIL` - Apple Developer email
- ✅ `APPLE_TEAM_ID` - Team ID from Apple Developer account
- ✅ `MATCH_REPOSITORY` - Match repository (e.g., `org/certificates`)
- ✅ `MATCH_PASSWORD` - Encryption password for certificates
- ✅ `MATCH_DEPLOY_KEY` - SSH deploy key (auto-generated by apple-setup action)
- ✅ `APPSTORE_ISSUER_ID` - App Store Connect API Issuer ID
- ✅ `APPSTORE_KEY_ID` - App Store Connect API Key ID
- ✅ `APPSTORE_P8` - App Store Connect API private key (.p8 contents)

**Optional (only needed for apple-setup action):**

- `GH_PAT` - GitHub Personal Access Token

**Important Notes:**

- For `APPSTORE_P8`: Copy the **entire contents** of the `.p8` file, including `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----`
- For `MATCH_PASSWORD`: Use a strong random password and **save it in your password manager** for reuse in other projects
- These secrets can be reused across multiple projects
- **Bundle IDs** are provided inline in your workflow YAML, not as secrets

## How It Works

1. **Detection Phase**: Automatically detects project dependencies (npm, Bundler, CocoaPods)
2. **Setup Phase**: Installs Node.js, Ruby, and sets up caching
3. **Dependencies Phase**: Installs npm packages, Bundler gems, and CocoaPods pods
4. **Build Phase**:
   - For unsigned: Builds with `mac-application` export method
   - For signed: Sets up match certificates, configures code signing, builds with `developer-id`, and optionally notarizes
5. **Package Phase**: Optionally creates a ZIP archive

## Comparison with Direct Workflow Steps

### Before (in your workflow):

```yaml
- name: Setup Node.js (root)
  uses: actions/setup-node@v4
  with:
    node-version: "20"
    cache: "npm"
# ... 50+ more lines of setup, caching, and build steps
```

### After (using this action):

```yaml
- uses: starburst997/build-macos@v1
  with:
    workspace_path: "macos/MyApp.xcworkspace"
    project_path: "macos/MyApp.xcodeproj"
    scheme: "MyApp-macOS"
    output_name: "MyApp"
    mac_bundle_id: ${{ secrets.MAC_BUNDLE_ID }}
    # ... add credentials
```

## Troubleshooting

### Build fails with "No such file or directory"

Make sure your paths are relative to `working_directory`. For example, if your workspace is at `sample/macos/MyApp.xcworkspace`:

```yaml
working_directory: "sample"
workspace_path: "macos/MyApp.xcworkspace"
```

### CocoaPods not being installed

The action auto-detects CocoaPods by looking for a Podfile. Make sure:

- The Podfile exists at `{working_directory}/{cocoapods_directory}/Podfile`
- Or manually enable it: `enable_cocoapods: true`

### npm dependencies not being installed

The action auto-detects npm by looking for package.json. Make sure:

- The package.json exists at `{working_directory}/package.json`
- Or manually enable it: `enable_npm: true`

### Code signing fails

- Verify you've run the apple-setup action first to generate certificates
- Check that all required secrets are set correctly
- Ensure the bundle ID matches the one used during certificate generation
- Verify the match deploy key has access to the match repository

### Notarization fails with "hardened runtime not enabled"

If you see the error `"The executable does not have the hardened runtime enabled"`, this means your app binary wasn't built with hardened runtime enabled, which is required for notarization.

**Solution:**

1. **Add an entitlements file** to your Xcode project (e.g., `macos/YourApp.entitlements`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
</dict>
</plist>
```

2. **Configure entitlements in Xcode:**
   - Open your project in Xcode
   - Select your target → Build Settings
   - Search for "Code Signing Entitlements"
   - Set it to the path of your entitlements file (e.g., `macos/YourApp.entitlements`)
   - Search for "Enable Hardened Runtime"
   - Set it to `YES`

3. **For React Native projects:** Ensure your Podfile has the following post-install hook:

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_HARDENED_RUNTIME'] = 'YES'
    end
  end
end
```

**Note:** The entitlements shown above (`allow-jit`, `allow-unsigned-executable-memory`, `disable-library-validation`) are commonly needed for apps that use JIT compilation or load dynamic libraries (like React Native, Electron, or Unity apps). Adjust based on your app's needs. For native macOS apps, you may need fewer entitlements.

## License

MIT

## Related Actions

- [starburst997/apple-setup](https://github.com/starburst997/apple-setup) - Set up Apple code signing certificates

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.
