# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Accountability Roulette is a single-page web application that randomly selects team roles using an interactive spinning wheel. Built with vanilla HTML/CSS/JavaScript (no frameworks), packaged in a lightweight nginx Alpine Docker container (~10MB), and deployed to Kubernetes/EKS. Features Google Analytics tracking, keyboard shortcuts, and a hidden easter egg.

## Repository Structure

```
.
├── index.html              # Complete SPA - all HTML/CSS/JS in one file
├── Dockerfile              # Alpine nginx container configuration
├── nginx.conf              # Custom nginx serving configuration
├── site.webmanifest        # PWA manifest for mobile installation
├── favicon*.png/ico        # Favicon assets for all platforms
└── infra/
    └── kubernetes-deployment.yaml  # K8s Deployment + LoadBalancer Service
```

**Critical files:**
- `index.html`: Entire application (lines 1-1400+) - all styling, logic, and markup in one self-contained file
- `infra/kubernetes-deployment.yaml`: K8s config with resource limits and health checks

## Core Architecture

### Single-File Application Pattern

The entire application exists in `index.html` with three main sections:

```html
<head>
  <!-- Google Analytics -->
  <!-- SEO meta tags -->
  <style>
    /* All CSS (~700 lines) - CSS variables for theming */
  </style>
</head>
<body>
  <!-- HTML structure (~150 lines) -->
  <script>
    /* All JavaScript (~650 lines) - vanilla JS, no frameworks */
  </script>
</body>
```

**Key principle:** Self-contained deployment. No build process, no dependencies, no bundler.

### Wheel Configuration

Teams are defined in the `roles` array (around line 998):

```javascript
const roles = [
    {
        name: 'INFRA',      // Display name
        color: '#0063e5',   // Segment color (hex)
        key: 'I',           // Keyboard shortcut
        excuses: [...]      // Array of 3 excuse strings
    },
    // ... 5 more roles (6 total)
];
```

**Pattern:** Exactly 6 segments. Canvas drawing logic assumes 360°/6 = 60° per segment.

### Keyboard Controls System

Event listener at ~line 1248 handles multiple control schemes:

```javascript
document.addEventListener('keydown', (e) => {
    // Easter egg (line 1250): '8' triggers explosion
    if (e.key === '8') { triggerExplosion(); }

    // Random spin (line 1256): Space/Enter
    if (e.code === 'Space' || e.key === 'Enter') { spinWheel(); }

    // Team-specific (line 1268): Letter keys (I/D/Q/V/P/C)
    const roleIndex = roles.findIndex(role => role.key === key);
    if (roleIndex !== -1) { spinWheel(roleIndex); }
});
```

**Pattern:** All controls route through `spinWheel(targetRoleIndex?)` function.

## Common Development Tasks

### Modify Team Roles

1. Open `index.html` in editor
2. Find `const roles = [` (around line 998)
3. Edit name/color/key/excuses for each role
4. Keep exactly 6 roles (wheel math requires this)
5. Test locally: `open index.html` in browser

### Update Branding/Colors

1. CSS variables at top of `<style>` (around line 47):
   ```css
   :root {
       --brand-dark: #040714;
       --brand-primary: #0063e5;
       /* ... edit these */
   }
   ```
2. Search and replace if changing variable names
3. Update `site.webmanifest` theme colors to match

### Deploy to Kubernetes

1. Build and push Docker image:
   ```bash
   docker build -t wheel-of-blame:latest .
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account>.dkr.ecr.us-east-1.amazonaws.com
   docker tag wheel-of-blame:latest <account>.dkr.ecr.us-east-1.amazonaws.com/wheel-of-blame:latest
   docker push <account>.dkr.ecr.us-east-1.amazonaws.com/wheel-of-blame:latest
   ```

2. Update image reference in `infra/kubernetes-deployment.yaml` (line 19)

3. Apply to cluster:
   ```bash
   kubectl apply -f infra/kubernetes-deployment.yaml
   kubectl get service wheel-of-blame-service  # Get LoadBalancer URL
   ```

### Test Locally with Docker

```bash
docker build -t wheel-of-blame:latest .
docker run -p 8080:80 wheel-of-blame:latest
# Visit http://localhost:8080
```

## Key Files to Understand

| File | Purpose | When to Reference |
|------|---------|-------------------|
| index.html | Entire application | All feature work |
| index.html:998-1053 | `roles` array definition | Changing teams/excuses |
| index.html:47-64 | CSS variables (theming) | Color/branding changes |
| index.html:1248-1276 | Keyboard event handler | Adding shortcuts |
| index.html:1166-1192 | `spinWheel()` function | Wheel mechanics |
| Dockerfile | Container configuration | Build/deployment issues |
| infra/kubernetes-deployment.yaml | K8s resources | Scaling/health checks |

## Anti-Patterns

| Don't | Do |
|-------|-----|
| Split HTML/CSS/JS into separate files | Keep as single self-contained file |
| Add npm/webpack/build process | Pure vanilla JS - no tooling |
| Change number of roles without updating math | Keep 6 segments (or recalculate 360°/N) |
| Use `--disney-*` CSS variable names | Use `--brand-*` (brand-neutral) |
| Mention brand names in code/comments | Keep all references generic |
| Add frameworks (React, Vue, etc.) | Vanilla JS only |
| Modify favicon files directly | Regenerate from favicon.io |
| Remove Google Analytics without consent | GA ID: G-DFTVJNTDCN (line 5) |

## Quick Reference

| Aspect | Standard |
|--------|----------|
| Tech stack | Vanilla HTML/CSS/JS (ES6+) |
| Container | nginx:alpine (~10MB) |
| K8s resources | 64Mi RAM / 100m CPU (request), 128Mi / 200m CPU (limit) |
| Font | Montserrat (Google Fonts) |
| Color scheme | Blue theme (#040714, #0063e5, #4BA3FF, #8A4FFF) |
| Canvas size | `min(70vw, 55vh, 500px)` - responsive |
| Wheel segments | Exactly 6 (60° each) |
| Analytics | Google Analytics GA4 (G-DFTVJNTDCN) |
| Deployment target | AWS EKS + ECR |
| Health checks | HTTP GET / on port 80 |

## When Working in This Repository

1. **Never split the single-file structure** - entire app stays in `index.html`
2. **Test in browser immediately** - just `open index.html`, no build needed
3. **Maintain exactly 6 roles** - wheel canvas math depends on this
4. **Use CSS variables** - all colors via `--brand-*` variables for easy theming
5. **Preserve keyboard shortcuts** - document any new shortcuts in help menu (lines 820-881)
6. **Test all controls** - verify Space, Enter, letter keys (I/D/Q/V/P/C), and easter egg (8)
7. **Keep brand-neutral** - no company/product names in code
8. **Test Docker build** - `docker build` should complete in <30 seconds
