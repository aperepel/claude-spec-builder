# Publishing to Claude Plugin Marketplace

## Prerequisites

- Claude Code CLI installed
- GitHub repository with plugin code

## Steps

1. **Ensure plugin.json is valid**
   ```bash
   cat plugin.json
   ```

2. **Test locally**
   ```bash
   claude plugins dev .
   /interview
   ```

3. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Prepare for release"
   git push origin main
   ```

4. **Publish**
   ```bash
   claude plugins publish
   ```

5. **Verify**
   ```bash
   claude plugins search claude-spec-builder
   ```

## Updating

After making changes:
```bash
# Bump version in plugin.json
claude plugins publish
```
