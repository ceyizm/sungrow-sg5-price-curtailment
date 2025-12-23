# GitHub Repository Setup Instructions

Follow these steps to publish your Sungrow integration to GitHub.

## Prerequisites

- GitHub account (create at https://github.com)
- Git installed on your system
- Terminal access

## Step 1: Create GitHub Repository

### Via GitHub Website

1. Go to https://github.com
2. Click the **+** icon → **New repository**
3. Fill in details:
   - **Repository name:** `sungrow-sg5-price-curtailment`
   - **Description:** "Price-aware solar curtailment for Sungrow SG5.0RS inverters with Home Assistant"
   - **Public** or **Private** (recommend Public for community sharing)
   - **Do NOT** check "Initialize with README" (we already have one)
   - **Do NOT** add .gitignore or license (we have them)
4. Click **Create repository**
5. Copy the repository URL (e.g., `https://github.com/Artic0din/sungrow-sg5-price-curtailment.git`)

## Step 2: Initialize Local Repository

Open terminal and navigate to the repository directory:

```bash
cd /Users/ryanfoyle/sungrow-sg5-price-curtailment
```

Initialize git:

```bash
git init
```

**Expected output:** `Initialized empty Git repository in /Users/ryanfoyle/sungrow-sg5-price-curtailment/.git/`

## Step 3: Add Files to Git

Add all files:

```bash
git add .
```

Check what will be committed:

```bash
git status
```

**Expected output:** List of new files in green (ready to commit)

## Step 4: Make Initial Commit

```bash
git commit -m "Initial commit: Sungrow SG5.0RS price-aware curtailment system

- Complete modbus integration for SG5.0RS
- Price-based solar curtailment automation
- Teslemetry integration for Powerwall
- Amber Electric integration for dynamic pricing
- Full documentation and examples
- Dashboard configuration
- MIT License"
```

**Expected output:** Summary of files committed

## Step 5: Connect to GitHub

Add your GitHub repository as remote:

```bash
git remote add origin https://github.com/Artic0din/sungrow-sg5-price-curtailment.git
```

**Replace** `Artic0din` with your actual GitHub username!

Verify remote was added:

```bash
git remote -v
```

**Expected output:**
```
origin  https://github.com/Artic0din/sungrow-sg5-price-curtailment.git (fetch)
origin  https://github.com/Artic0din/sungrow-sg5-price-curtailment.git (push)
```

## Step 6: Push to GitHub

### If using HTTPS (username/password or token)

```bash
git push -u origin main
```

**Note:** GitHub now requires Personal Access Token instead of password:
1. Go to GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Give it `repo` permissions
4. Use token as password when prompted

### If using SSH (recommended)

First, set up SSH keys if you haven't:

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Add to SSH agent
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub
```

Add the public key to GitHub:
1. GitHub → Settings → SSH and GPG keys → New SSH key
2. Paste the key, save

Then update remote to use SSH:

```bash
git remote set-url origin git@github.com:Artic0din/sungrow-sg5-price-curtailment.git
git push -u origin main
```

## Step 7: Verify Upload

1. Go to your GitHub repository in browser
2. Refresh the page
3. You should see:
   - All files uploaded
   - README.md displayed on main page
   - Directory structure visible

## Step 8: Configure Repository Settings

### Add Topics (Tags)

1. Click **⚙️ Settings** (repository settings, not account)
2. Under "About" → Click **⚙️** gear icon
3. Add topics:
   - `home-assistant`
   - `sungrow`
   - `solar`
   - `battery`
   - `tesla-powerwall`
   - `amber-electric`
   - `automation`
   - `modbus`
   - `price-aware`
   - `curtailment`
4. Click **Save changes**

### Update Description

1. Same "About" section
2. Add description:
   ```
   Automatically curtail solar production when battery is full and electricity prices are zero/negative. Built for Sungrow SG5.0RS with Tesla Powerwall and dynamic pricing.
   ```
3. Add website (optional): Your Home Assistant URL or blog
4. Click **Save changes**

### Enable Issues and Discussions

1. **Settings** → **General**
2. Under "Features":
   - ✅ **Issues** - For bug reports
   - ✅ **Discussions** - For questions and community
3. Scroll down, click **Save changes**

## Step 9: Create Release (Optional)

Create your first release:

1. Click **Releases** (right sidebar)
2. Click **Create a new release**
3. Fill in:
   - **Tag version:** `v1.0.0`
   - **Release title:** `v1.0.0 - Initial Release`
   - **Description:**
     ```markdown
     ## 🎉 Initial Release

     Complete price-aware solar curtailment system for Sungrow SG5.0RS inverters.

     ### Features
     - ✅ Automatic curtailment during zero/negative pricing
     - ✅ Tesla Powerwall integration via Teslemetry
     - ✅ Amber Electric dynamic pricing
     - ✅ Manual inverter control
     - ✅ Configurable thresholds
     - ✅ Complete documentation

     ### Installation
     See [INSTALLATION.md](docs/INSTALLATION.md) for setup instructions.

     ### Tested On
     - Home Assistant 2024.12
     - Sungrow SG5.0RS with WiNet-S
     - Tesla Powerwall 2
     - Amber Electric (Australia)
     ```
4. Click **Publish release**

## Step 10: Update README.md Links

Edit README.md and replace placeholder URLs:

```bash
# Find and replace
# From: https://github.com/Artic0din/sungrow-sg5-price-curtailment
# To: https://github.com/YOUR_ACTUAL_USERNAME/sungrow-sg5-price-curtailment
```

Then commit and push:

```bash
git add README.md
git commit -m "docs: update repository URLs"
git push
```

## Step 11: Add README Badge (Optional)

Add badges to top of README.md:

```markdown
# Sungrow SG5.0RS Price-Aware Solar Curtailment for Home Assistant

[![GitHub release](https://img.shields.io/github/release/Artic0din/sungrow-sg5-price-curtailment.svg)](https://github.com/Artic0din/sungrow-sg5-price-curtailment/releases)
[![License](https://img.shields.io/github/license/Artic0din/sungrow-sg5-price-curtailment.svg)](LICENSE)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.1+-blue.svg)](https://www.home-assistant.io/)
[![Maintenance](https://img.shields.io/badge/Maintained-Yes-green.svg)](https://github.com/Artic0din/sungrow-sg5-price-curtailment/graphs/commit-activity)

Automatically curtail solar production when...
```

## Step 12: Share Your Repository

Now that it's live, share it:

1. **Home Assistant Community Forum:**
   - Post in "Share Your Projects"
   - Link to your GitHub repo

2. **Reddit:**
   - r/homeassistant
   - r/solar

3. **Twitter/X:**
   ```
   Just released my Sungrow SG5.0RS price-aware curtailment integration for @home_assistant!

   Automatically stops solar export during negative pricing events 🚫⚡💰

   https://github.com/Artic0din/sungrow-sg5-price-curtailment

   #HomeAssistant #Solar #HomeAutomation
   ```

## Future Updates

### Making Changes

```bash
# Make your edits to files

# Stage changes
git add .

# Commit with descriptive message
git commit -m "feat: add support for SG6.0RS model"

# Push to GitHub
git push
```

### Creating New Releases

When you make significant updates:

```bash
# Tag the commit
git tag -a v1.1.0 -m "Version 1.1.0 - Add SG6.0RS support"

# Push tags
git push --tags
```

Then create release on GitHub as in Step 9.

## Troubleshooting

### Authentication Failed

**Problem:** `fatal: Authentication failed`

**Solution:** Use Personal Access Token instead of password:
1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic) with `repo` scope
3. Use token as password

### Permission Denied (SSH)

**Problem:** `Permission denied (publickey)`

**Solution:** Add SSH key to GitHub (see Step 6)

### Repository Already Exists

**Problem:** `remote origin already exists`

**Solution:**
```bash
git remote remove origin
git remote add origin https://github.com/Artic0din/sungrow-sg5-price-curtailment.git
```

### Wrong Branch Name

**Problem:** Using `master` instead of `main`

**Solution:**
```bash
git branch -M main
git push -u origin main
```

## Repository Maintenance

### Weekly
- Respond to issues
- Review pull requests
- Update documentation if needed

### Monthly
- Check for Home Assistant compatibility
- Update dependencies
- Review and close stale issues

### Major Releases
- Update version number
- Create release notes
- Announce in community

## Getting Stars

To get more GitHub stars (visibility):

1. **Quality documentation** - You already have this ✅
2. **Solve real problems** - You already do this ✅
3. **Share in communities** - Post to forums, Reddit
4. **Respond to issues** - Be helpful and responsive
5. **Keep updating** - Regular commits show maintenance

## Analytics

View repository insights:
- **Insights** → **Traffic** - See visitor count
- **Insights** → **Community** - See engagement
- **Insights** → **Contributors** - See who helped

---

**Your repository is now live and ready to help the community!** 🚀

Don't forget to:
- ⭐ Star your own repo (first star!)
- 👀 Watch for issues/PRs
- 📢 Share with the community
