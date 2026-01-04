# PAI on Android - Quick Start Guide

Run your Personal AI Infrastructure on any Android device using Termux.

---

## What Works on Android?

✅ **Fully Supported:**
- Core PAI functionality
- History tracking system
- Hook system for event capture
- Custom skills and workflows
- File-based operations
- Git integration

⚠️ **Limited Support:**
- Voice system (requires audio workarounds)
- Observability dashboard (web server works, external access needs configuration)
- Background process management (requires wake-lock)

❌ **Not Supported:**
- Desktop-specific notifications (use Termux notifications instead)
- System-level integrations

---

## Quick Installation (5 Minutes)

### Step 1: Install Termux (2 min)

**Download Termux from F-Droid (NOT Google Play):**
- Visit: https://f-droid.org/packages/com.termux/
- Or GitHub: https://github.com/termux/termux-app/releases
- Install the APK

**Why not Google Play?** The Play Store version is outdated and incompatible.

### Step 2: Set Up Environment (2 min)

Open Termux and run:

```bash
# Update system
pkg update && pkg upgrade -y

# Install requirements
pkg install -y git curl

# Grant storage access
termux-setup-storage
```

Grant the permission when prompted.

### Step 3: Install Bun (1 min)

```bash
# Install Bun runtime
curl -fsSL https://bun.sh/install | bash

# Configure PATH
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify
bun --version
```

### Step 4: Install PAI (< 1 min)

```bash
# Clone repository
git clone https://github.com/danielmiessler/PAI.git
cd PAI/Bundles/Kai

# Run installer
bun run install.ts
```

Follow the prompts to configure your AI assistant name, timezone, etc.

---

## Essential Android Commands

### Managing Sessions

```bash
# Keep Termux awake (prevents process killing)
pkg install termux-api
termux-wake-lock acquire

# When done
termux-wake-lock release
```

### Persistent Sessions with tmux

```bash
# Install tmux
pkg install tmux

# Start a session
tmux new -s pai

# Detach (keep running in background)
# Press: Ctrl+B, then D

# Reattach later
tmux attach -t pai
```

### Storage Access

```bash
# PAI directory (Termux internal)
cd ~/.claude

# Android shared storage
cd ~/storage/shared

# Move files between Termux and Android
cp ~/.claude/.env ~/storage/shared/pai-backup.env
```

---

## Android-Specific Configuration

### Environment Variables

Add to `~/.bashrc`:

```bash
export DA="YourAIName"
export TIME_ZONE="America/New_York"
export PAI_DIR="$HOME/.claude"
export PAI_PLATFORM="android"
```

### Notifications

Install Termux:API from F-Droid, then:

```bash
pkg install termux-api

# Test notification
termux-notification --title "PAI" --content "Ready!"
```

PAI hooks will automatically use `termux-notification` on Android.

### Audio Playback

For the voice system:

```bash
# Option 1: mpg123
pkg install mpg123

# Option 2: ffmpeg
pkg install ffmpeg

# Option 3: termux-media-player (requires Termux:API)
pkg install termux-api
```

---

## Recommended Packs for Android

### Essential (Install These First)

1. **kai-core-install** - Core functionality
2. **kai-history-system** - Context tracking
3. **kai-hook-system** - Event capture

### Optional (Works Well)

4. **kai-prompting-skill** - Prompt templates
5. **kai-agents-skill** - Agent orchestration
6. **kai-art-skill** - Visual content generation (if you have API keys)

### Advanced (Requires Configuration)

7. **kai-observability-server** - Web dashboard (configure port forwarding)
8. **kai-voice-system** - TTS notifications (configure audio)

---

## Common Android Issues & Solutions

### "Command not found" after installing Bun

```bash
source ~/.bashrc
# Or restart Termux
```

### Package installation fails

```bash
pkg update
pkg upgrade
# Try again
```

### Storage permission denied

```bash
termux-setup-storage
# Grant permission in Android settings
```

### Process killed when switching apps

```bash
# Install termux-api
pkg install termux-api

# Acquire wake lock before long operations
termux-wake-lock acquire

# Or use tmux for persistent sessions
tmux new -s pai
```

### Bun installation fails

Try manual installation:

```bash
# For ARM64 (most modern Android devices)
wget https://github.com/oven-sh/bun/releases/latest/download/bun-linux-aarch64.zip
unzip bun-linux-aarch64.zip
mkdir -p ~/.local/bin
mv bun-linux-aarch64/bun ~/.local/bin/
chmod +x ~/.local/bin/bun
export PATH="$HOME/.local/bin:$PATH"
```

---

## Android Best Practices

### 1. Use tmux for Persistence

Always run PAI in a tmux session so you can switch apps without killing processes:

```bash
tmux new -s pai
cd ~/.claude
# Your PAI work here
# Ctrl+B, D to detach
```

### 2. Backup Your Configuration

Regularly sync your `.env` and configuration:

```bash
# Backup to Android storage
cp ~/.claude/.env ~/storage/shared/pai-backup.env

# Or use git
cd ~/.claude
git init
git add .env skills/
git commit -m "Backup config"
```

### 3. Disable Battery Optimization

In Android Settings:
1. Apps → Termux → Battery
2. Set to "Unrestricted"

This prevents Android from killing Termux in the background.

### 4. Use External Keyboard

For serious work, connect a Bluetooth keyboard. Makes editing much easier.

### 5. Keep Termux Updated

```bash
pkg update && pkg upgrade
```

Run weekly to get security updates and bug fixes.

---

## Using PAI with AI on Android

### Option 1: Terminal-Based AI

Install AI CLI tools in Termux:

```bash
# OpenAI CLI (example)
pip install openai
# Configure with your API key
```

### Option 2: Web-Based Access

Run observability server, access via browser:

```bash
cd ~/.claude/observability
./manage.sh start
# Open browser to localhost:3030
```

### Option 3: SSH from Desktop

```bash
# Install SSH server in Termux
pkg install openssh

# Start SSH server
sshd

# Get your IP
ifconfig

# From desktop, connect:
# ssh -p 8022 <termux-ip>
```

Then use your desktop AI to control PAI on Android.

### Option 4: Synced Configuration

Keep PAI config synced between devices:

```bash
# On Android
cd ~/.claude
git init
git add .
git commit -m "Initial config"
git remote add origin <your-repo>
git push

# On desktop
git pull
# Now both devices share the same PAI config
```

---

## Performance Tips

### Free Up RAM

```bash
# Clear package cache
pkg clean

# Remove unused packages
pkg autoremove
```

### Optimize Bun

```bash
# Use Bun's production mode for better performance
export BUN_ENV=production
```

### Use Lightweight Editors

Instead of heavy IDEs, use:

```bash
pkg install micro  # Modern, mouse-friendly
pkg install vim    # Classic, powerful
pkg install nano   # Simple, beginner-friendly
```

---

## Accessing Your Files

### From Termux to Android

```bash
# Copy file to Downloads
cp file.txt ~/storage/downloads/

# Copy to shared storage
cp file.txt ~/storage/shared/
```

### From Android to Termux

```bash
# Access Android Downloads in Termux
ls ~/storage/downloads/

# Copy to Termux
cp ~/storage/downloads/file.txt ~/
```

### Using File Managers

Install a file manager that supports Termux:
- Material Files (F-Droid)
- Ghost Commander (F-Droid)

These can browse `~/storage/` to access Termux files.

---

## Advanced: Running Web Server

To access PAI observability dashboard from other devices:

### 1. Start the Server

```bash
cd ~/.claude/observability
./manage.sh start
```

### 2. Find Your IP

```bash
ifconfig wlan0 | grep inet
# Look for inet addr: 192.168.x.x
```

### 3. Access from Browser

On the same WiFi:
- Open browser: `http://192.168.x.x:3030`

### 4. Port Forwarding (Optional)

For external access, configure your router to forward port 3030 to your Android device's IP.

**Security Warning:** Only expose to trusted networks. Consider using SSH tunneling for secure remote access.

---

## Troubleshooting

### Check System Info

```bash
uname -a          # System info
echo $PREFIX      # Termux prefix
bun --version     # Bun version
which bun         # Bun location
env | grep PAI    # PAI environment
```

### Verify Installation

```bash
cd ~/.claude
ls -la            # Check directory structure
cat .env          # Check configuration
ls skills/        # Check installed skills
```

### Reset Installation

If something goes wrong:

```bash
# Backup first
cp -r ~/.claude ~/.claude-backup

# Remove and reinstall
rm -rf ~/.claude
cd ~/PAI/Bundles/Kai
bun run install.ts
```

---

## Getting Help

**Documentation:**
- Full guide: `/PLATFORM.md` (Android Installation Guide section)
- Platform compatibility: `/PLATFORM.md`
- Pack documentation: `/Packs/`

**Community:**
- GitHub Issues: https://github.com/danielmiessler/PAI/issues
- Tag issues with `[Android]` for platform-specific help

**Termux Resources:**
- Termux Wiki: https://wiki.termux.com
- Termux GitHub: https://github.com/termux/termux-app

---

## What's Next?

Once installed:

1. **Explore the packs**: Browse `/Packs/` directory
2. **Customize SKILL.md**: Edit `~/.claude/skills/CORE/SKILL.md`
3. **Set up history**: Install `kai-history-system` pack
4. **Configure hooks**: Install `kai-hook-system` pack
5. **Add skills**: Install domain-specific skill packs

**Android-Specific Next Steps:**

- Set up tmux sessions for persistence
- Configure wake-lock for important tasks
- Sync your `.env` to cloud backup
- Explore Termux:API capabilities
- Join the community and share your Android setup!

---

## About Android Support

PAI Android support is community-driven. The core team tests primarily on macOS and Linux desktop, but the community maintains and improves Android compatibility.

**Contribute:**
- Report Android-specific issues
- Share your Termux setup
- Submit fixes for Android bugs
- Document Android best practices

**Status:** ⚡ Supported via Termux (as of v2.1+)

---

*Built with ❤️ by the PAI community*

*Optimized for Termux on Android*
