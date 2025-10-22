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

To use signed builds, you need to set up code signing certificates with fastlane match. Follow the guide at [starburst997/apple-setup](https://github.com/starburst997/apple-setup) to:

1. Create an App Store Connect API key
2. Create a match repository for storing certificates
3. Generate certificates and provisioning profiles
4. Set up the required GitHub secrets

### Required Secrets for Signed Builds

Add these secrets to your GitHub repository:

- `APPLE_CONNECT_EMAIL` - Apple Connect email
- `APPLE_DEVELOPER_EMAIL` - Apple Developer email
- `APPLE_TEAM_ID` - Team ID from Apple Developer account
- `MATCH_REPOSITORY` - Match repository (e.g., `org/certificates`)
- `MATCH_PASSWORD` - Encryption password for certificates
- `MATCH_DEPLOY_KEY` - SSH deploy key for match repository
- `APPSTORE_ISSUER_ID` - App Store Connect API Issuer ID
- `APPSTORE_KEY_ID` - App Store Connect API Key ID
- `APPSTORE_P8` - App Store Connect API private key (.p8 contents)

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

## License

MIT

## Related Actions

- [starburst997/apple-setup](https://github.com/starburst997/apple-setup) - Set up Apple code signing certificates
- [starburst997/setup-ruby](https://github.com/starburst997/setup-ruby) - Set up Ruby with platform support

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.
