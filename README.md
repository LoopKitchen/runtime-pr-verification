# Runtime PR Verification Action

🚀 **Firebase-aware visual testing for React SPAs using Gemini analysis**

This GitHub Action adds runtime verification to your PR workflow by automatically testing React applications deployed to Firebase Hosting. It integrates with Gemini code review analysis to generate targeted visual tests, captures screenshots and videos, and posts comprehensive results back to your PR.

## 🌟 Key Features

- **🔥 Firebase Native**: Works seamlessly with Firebase Hosting preview deployments
- **⚛️ React Optimized**: Specialized for React SPAs with Vite and Create React App support  
- **🤖 Gemini Integration**: Parses AI code review analysis to generate relevant tests
- **📸 Visual Evidence**: Captures screenshots and videos across multiple viewports
- **☁️ Integrated Storage**: Uploads results to your Firebase Storage bucket
- **📝 Rich PR Comments**: Beautiful, collapsible PR comments with visual evidence
- **🎯 Smart Testing**: Generates component, route, and interaction tests automatically

## 🚀 Quick Start

### 1. Basic Setup

Add this action to your workflow after Firebase deployment:

```yaml
name: PR Verification
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest
    outputs:
      preview_url: ${{ steps.firebase.outputs.preview_url }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Firebase
        id: firebase
        run: |
          # Your Firebase deployment
          echo "preview_url=$PREVIEW_URL" >> $GITHUB_OUTPUT

  gemini-review:
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/gemini-code-review@v1

  visual-verification:
    needs: [deploy, gemini-review]
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/runtime-pr-verification@v1
        with:
          preview-url: ${{ needs.deploy.outputs.preview_url }}
          firebase-credentials: ${{ secrets.FIREBASE_SA_BASE64 }}
          storage-bucket: ${{ vars.FIREBASE_STORAGE_BUCKET }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Required Secrets & Variables

Set up these in your repository settings:

```bash
# Secrets
FIREBASE_SA_BASE64          # Base64 encoded Firebase service account JSON
GITHUB_TOKEN               # Automatically provided by GitHub

# Variables  
FIREBASE_STORAGE_BUCKET    # Your Firebase Storage bucket name
```

## 📖 Examples for Your Repositories

### For loop-frontend (Vite + Multi-target)

```yaml
name: Frontend PR Checks
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    outputs:
      preview_url: ${{ steps.firebase.outputs.preview_url }}
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'
      
      - name: Install dependencies
        run: yarn install --frozen-lockfile
      
      - name: Build application
        run: yarn build
      
      - name: Deploy to Firebase Preview
        id: firebase
        run: |
          # Deploy to Firebase with target 'app'
          firebase deploy --only hosting:app --project ${{ vars.FIREBASE_PROJECT_ID }}
          PREVIEW_URL="https://${{ vars.FIREBASE_PROJECT_ID }}--pr-${{ github.event.number }}-app.web.app"
          echo "preview_url=$PREVIEW_URL" >> $GITHUB_OUTPUT

  gemini-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/gemini-review@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

  runtime-verification:
    needs: [deploy-preview, gemini-review]
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/runtime-pr-verification@v1
        with:
          preview-url: ${{ needs.deploy-preview.outputs.preview_url }}
          firebase-credentials: ${{ secrets.FIREBASE_SA_BASE64 }}
          storage-bucket: 'loop-frontend-screenshots'
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # Auto-detects: firebase-target=app, build-system=vite
          viewports: '1920x1080:Desktop,768x1024:Tablet,375x667:Mobile'
          test-timeout: '8m'
```

### For loop-admin (React + Single target)

```yaml
name: Admin Dashboard PR Checks  
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    outputs:
      preview_url: ${{ steps.firebase.outputs.preview_url }}
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'
      
      - name: Install and build
        run: |
          yarn install --frozen-lockfile
          yarn build
      
      - name: Deploy to Firebase Preview
        id: firebase
        run: |
          firebase deploy --only hosting:loop-ad --project ${{ vars.FIREBASE_PROJECT_ID }}
          PREVIEW_URL="https://${{ vars.FIREBASE_PROJECT_ID }}--pr-${{ github.event.number }}-loop-ad.web.app"
          echo "preview_url=$PREVIEW_URL" >> $GITHUB_OUTPUT

  gemini-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/gemini-review@v1

  runtime-verification:
    needs: [deploy-preview, gemini-review]
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/runtime-pr-verification@v1
        with:
          preview-url: ${{ needs.deploy-preview.outputs.preview_url }}
          firebase-credentials: ${{ secrets.FIREBASE_SA_BASE64 }}
          storage-bucket: 'loop-admin-screenshots'
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # Auto-detects: firebase-target=loop-ad, build-system=react
          max-routes: '15'
          cleanup-days: '14'
```

## ⚙️ Configuration

### Input Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `preview-url` | ✅ | - | Firebase preview URL |
| `firebase-credentials` | ✅ | - | Base64 encoded service account JSON |
| `storage-bucket` | ✅ | - | Firebase Storage bucket name |
| `github-token` | ✅ | - | GitHub token for PR comments |
| `gemini-bot-name` | ❌ | `gemini-bot` | Name of your Gemini review bot |
| `firebase-project-id` | ❌ | *auto-detected* | Firebase project ID |
| `firebase-target` | ❌ | *auto-detected* | Firebase hosting target |
| `build-system` | ❌ | *auto-detected* | `vite` or `react` |
| `test-timeout` | ❌ | `5m` | Maximum test execution time |
| `cleanup-days` | ❌ | `30` | Days to keep screenshots in storage |
| `viewports` | ❌ | `1920x1080,768x1024,375x667` | Comma-separated viewport sizes |
| `max-routes` | ❌ | `10` | Maximum routes to test automatically |

### Output Variables

| Output | Description |
|--------|-------------|
| `status` | `success`, `failure`, or `partial` |
| `screenshots-url` | Firebase Storage URL with all results |
| `test-results` | JSON summary of test execution |
| `firebase-project` | Detected Firebase project ID |
| `firebase-target` | Detected Firebase hosting target |
| `build-system` | Detected build system |

## 🧪 Generated Tests

The action automatically generates tests based on Gemini's analysis:

### Component Tests
- **Visibility**: Verifies React components render correctly
- **Interaction**: Tests buttons, forms, and interactive elements
- **Responsive**: Checks component behavior across viewports

### Route Tests  
- **Navigation**: Tests React Router navigation between pages
- **Loading**: Verifies routes load without errors
- **Content**: Checks page content appears correctly

### Form Tests
- **Input Validation**: Tests form field interactions
- **Submission**: Verifies form submission workflows
- **Error Handling**: Checks error states and validation

### React SPA Optimizations
- **Hydration Waiting**: Waits for React to fully hydrate
- **Bundle Loading**: Handles Vite vs CRA loading patterns  
- **Client-side Navigation**: Tests SPA routing properly
- **Error Boundaries**: Verifies no crash states

## 📸 Visual Evidence

### Screenshots
- Captured across all specified viewports
- Full page screenshots with proper scrolling
- Component-focused shots for specific tests
- Before/after shots for interactive tests

### Videos  
- Recorded for interaction tests
- WebM format for browser compatibility
- Automatically compressed for storage efficiency
- Linked directly in PR comments

### Storage Organization
```
Firebase Storage Bucket:
├── runtime-pr-verification/
│   ├── PR-123/
│   │   ├── 2024-01-15/
│   │   │   ├── screenshots/
│   │   │   │   ├── spa-loading-Desktop-final.png
│   │   │   │   ├── component-header-Tablet-final.png
│   │   │   │   └── route-dashboard-Mobile-final.png
│   │   │   ├── videos/
│   │   │   │   └── form-login-Desktop.webm
│   │   │   └── test-summary.json
```

## 🤖 Gemini Integration

### Supported Analysis Patterns

The action parses Gemini comments for:

```markdown
## Code Review Analysis

**Components Modified**: `Header`, `LoginForm`, `Dashboard`
**Routes Changed**: `/login`, `/dashboard`, `/settings`  
**Risk Level**: Medium
**Testing Suggestions**:
- Verify login form validation
- Test responsive header behavior
- Check dashboard data loading
```

### Fallback Behavior

If no Gemini comment is found, the action will:
- Test the root route (`/`)
- Perform basic React SPA verification
- Capture responsive screenshots
- Check for console errors

## 🔧 Setup Guide

### 1. Firebase Service Account

Create a service account with these permissions:
- Firebase Hosting Admin
- Storage Admin  
- Viewer (for project access)

```bash
# Create service account
gcloud iam service-accounts create runtime-pr-verification \
  --display-name="Runtime PR Verification"

# Grant permissions
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="serviceAccount:runtime-pr-verification@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/firebase.admin"

# Generate key
gcloud iam service-accounts keys create key.json \
  --iam-account=runtime-pr-verification@YOUR_PROJECT_ID.iam.gserviceaccount.com

# Base64 encode for GitHub secret
base64 -i key.json | pbcopy
```

### 2. Firebase Storage Setup

Ensure your Firebase Storage bucket exists and has proper rules:

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /runtime-pr-verification/{allPaths=**} {
      allow read: if true; // Public read for signed URLs
      allow write: if false; // Only service account can write
    }
  }
}
```

### 3. Repository Secrets

Add to your repository settings:

```bash
# Repository Settings > Secrets and variables > Actions

# Secrets
FIREBASE_SA_BASE64 = "ewogICJ0eXBlIjogInNlcnZpY2VfYWNjb3VudCIsC..."

# Variables  
FIREBASE_PROJECT_ID = "your-project-id"
FIREBASE_STORAGE_BUCKET = "your-project-id.appspot.com"
```

## 🎯 Advanced Usage

### Custom Test Generation

Override default test generation:

```yaml
- uses: your-org/runtime-pr-verification@v1
  with:
    preview-url: ${{ needs.deploy.outputs.preview_url }}
    firebase-credentials: ${{ secrets.FIREBASE_SA_BASE64 }}
    storage-bucket: ${{ vars.FIREBASE_STORAGE_BUCKET }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    # Custom viewport configurations
    viewports: '1440x900:Laptop,834x1194:iPad,390x844:iPhone'
    # Extended timeout for complex SPAs
    test-timeout: '10m'
    # More thorough route testing
    max-routes: '20'
    # Faster cleanup cycle  
    cleanup-days: '7'
```

### Multi-Environment Testing

Test different Firebase targets:

```yaml
strategy:
  matrix:
    environment: [staging, production]
    
steps:
  - uses: your-org/runtime-pr-verification@v1
    with:
      preview-url: ${{ matrix.environment == 'staging' && needs.deploy.outputs.staging_url || needs.deploy.outputs.prod_url }}
      firebase-credentials: ${{ secrets.FIREBASE_SA_BASE64 }}
      storage-bucket: ${{ vars.FIREBASE_STORAGE_BUCKET }}
      github-token: ${{ secrets.GITHUB_TOKEN }}
      firebase-target: ${{ matrix.environment }}
```

### Conditional Execution

Only run for UI changes:

```yaml
- name: Check for UI changes
  id: ui-changes
  run: |
    CHANGED_FILES=$(git diff --name-only ${{ github.event.before }} ${{ github.sha }})
    if echo "$CHANGED_FILES" | grep -E '\.(tsx?|jsx?|css|scss|less)$'; then
      echo "ui-changes=true" >> $GITHUB_OUTPUT
    else
      echo "ui-changes=false" >> $GITHUB_OUTPUT
    fi

- uses: your-org/runtime-pr-verification@v1
  if: steps.ui-changes.outputs.ui-changes == 'true'
  with:
    # ... configuration
```

## 🔍 Troubleshooting

### Common Issues

**❌ Firebase deployment not ready**
```
Error: Firebase deployment did not become accessible within 600 seconds
```
*Solution*: Increase `test-timeout` or check Firebase deployment logs.

**❌ React SPA not hydrating**
```
Warning: React SPA ready check failed. Continuing with test...
```
*Solution*: Ensure your app renders to `#root` or `#app` element.

**❌ Screenshots empty or broken**
```
Screenshots captured but appear blank
```
*Solution*: Check for CSS that might hide content or loading states.

**❌ Firebase Storage upload failed**
```
Failed to upload screenshot: Permission denied
```
*Solution*: Verify service account has Storage Admin role.

### Debug Mode

Enable verbose logging:

```yaml
- uses: your-org/runtime-pr-verification@v1
  with:
    # ... other inputs
  env:
    ACTIONS_STEP_DEBUG: true
    ACTIONS_RUNNER_DEBUG: true
```

### Local Testing

Test the action locally:

```bash
npm install
npm run test:local -- --url=https://your-preview.web.app
```

## 📊 Performance

- **Execution Time**: Typically 2-5 minutes for standard React SPAs
- **Screenshot Size**: ~50-200KB per image (PNG, optimized)
- **Video Size**: ~1-5MB per interaction video (WebM, compressed)
- **Storage Usage**: ~10-50MB per PR (auto-cleanup after 30 days)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Submit a pull request

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🏷️ Version History

- **v1.0.0**: Initial release with Firebase + React support
- **v1.1.0**: Added Gemini integration and multi-viewport testing
- **v1.2.0**: Enhanced error handling and performance optimization

---

**Made with ❤️ for React + Firebase developers**

*This action is optimized for Loop Kitchen's development workflow but works great for any React SPA deployed to Firebase Hosting.*# runtime-pr-verification
