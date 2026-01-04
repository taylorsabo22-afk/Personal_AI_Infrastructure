# PAI Platform Compatibility Status

This document tracks all platform-specific code and dependencies across PAI, providing a roadmap for cross-platform support.

**Last Updated:** 2026-01-01
**Maintainer:** Community contributions welcome

---

## Platform Support Matrix

| Platform | Status | Notes |
|----------|--------|-------|
| **macOS** | ✅ Fully Supported | Primary development platform |
| **Linux** | ✅ Fully Supported | Ubuntu/Debian tested, other distros via community |
| **Android** | ⚡ Supported via Termux | See Android Installation Guide below |
| **Windows** | ❌ Not Supported | Community contributions welcome |

---

## Android Installation Guide

### Prerequisites

PAI can run on Android devices using **Termux**, a powerful terminal emulator that provides a Linux environment on Android.

**Requirements:**
- Android device (phone or tablet) running Android 7.0 or higher
- At least 2GB of free storage
- Stable internet connection for installation

### Step 1: Install Termux

1. **Download Termux** from F-Droid (recommended) or GitHub:
   - **F-Droid:** https://f-droid.org/packages/com.termux/
   - **GitHub:** https://github.com/termux/termux-app/releases

   ⚠️ **Do NOT use the Google Play Store version** - it's outdated and incompatible.

2. **Install Termux:API** (optional, for extended functionality):
   - Download from F-Droid: https://f-droid.org/packages/com.termux.api/

### Step 2: Set Up Termux Environment

Open Termux and run these commands:

```bash
# Update package repositories
pkg update && pkg upgrade -y

# Install essential packages
pkg install -y git curl wget termux-api

# Grant storage access (important for file operations)
termux-setup-storage
```

When prompted, grant Termux storage permissions. This creates a `~/storage` directory linking to your Android device storage.

### Step 3: Install Bun Runtime

PAI uses Bun as its primary runtime. Install it in Termux:

```bash
# Install Bun
curl -fsSL https://bun.sh/install | bash

# Add Bun to PATH (add to ~/.bashrc for persistence)
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify installation
bun --version
```

### Step 4: Install PAI

```bash
# Clone the PAI repository
cd ~
git clone https://github.com/danielmiessler/PAI.git
cd PAI

# Run the Kai Bundle installer
cd Bundles/Kai
bun run install.ts
```

Follow the installation wizard prompts. The installer will:
- Detect Termux/Android environment automatically
- Create the `~/.claude` directory structure
- Configure environment variables
- Set up PAI with Android-compatible defaults

### Step 5: Install Packs

Install individual packs by giving the pack files to your AI or following the manual installation instructions in each pack.

**Recommended for Android:**
- `kai-core-install` - Core PAI functionality
- `kai-history-system` - Context tracking (works well on Android)
- `kai-hook-system` - Event capture system

**Not recommended for Android:**
- `kai-voice-system` - Audio playback has limitations on Android
- `kai-observability-server` - Web server; consider external access issues

### Android-Specific Configuration

#### Environment Variables

Termux uses a standard Linux environment. Add PAI environment variables to `~/.bashrc`:

```bash
# Edit your bashrc
nano ~/.bashrc

# Add these lines:
export DA="YourAIName"
export TIME_ZONE="America/New_York"  # Your timezone
export PAI_DIR="$HOME/.claude"
export PAI_SOURCE_APP="$DA"
```

#### Storage Paths

On Android/Termux:
- **PAI installation:** `~/.claude` (in Termux's home directory)
- **Shared storage access:** `~/storage/` (links to Android filesystem)
- **External storage:** `~/storage/shared/` (accessible by other apps)

#### Notifications

Android notifications require the Termux:API add-on:

```bash
# Install termux-api package
pkg install termux-api

# Test notification
termux-notification --title "PAI" --content "Setup complete!"
```

To enable PAI notifications, the voice and observability packs will automatically detect Termux and use `termux-notification` instead of `notify-send`.

### Android Limitations and Workarounds

#### Audio Playback

**Issue:** The `kai-voice-system` pack uses `afplay` (macOS) or `mpg123` (Linux), which may not work perfectly on Android.

**Workaround:**
```bash
# Install mpg123 for audio playback
pkg install mpg123

# Or use termux-media-player
pkg install termux-api
# Use termux-media-player command for playback
```

#### Background Execution

**Issue:** Android may kill background processes to save battery.

**Workaround:**
1. Disable battery optimization for Termux in Android settings
2. Use `termux-wake-lock acquire` to prevent system from sleeping:
   ```bash
   termux-wake-lock acquire
   # Run your PAI commands
   termux-wake-lock release
   ```

#### Process Management

**Issue:** Some desktop tools assume systemd or launchd.

**Workaround:**
- Use `nohup` and `&` for background processes
- Consider using `tmux` or `screen` for persistent sessions:
  ```bash
  pkg install tmux
  tmux new -s pai
  # Your PAI commands here
  # Detach with Ctrl+B, then D
  ```

#### File Editors

Install a text editor for configuration files:

```bash
# Choose your preferred editor
pkg install vim
# or
pkg install nano
# or  
pkg install micro
```

### Testing Your Installation

Verify PAI is working on Android:

```bash
# Check environment
echo $DA
echo $PAI_DIR
ls -la ~/.claude

# Test Bun runtime
cd ~/.claude
bun --version

# If you installed kai-history-system
ls -la ~/.claude/history
```

### Accessing Your AI on Android

Since PAI is designed to work with AI coding assistants, you have several options on Android:

1. **Use a terminal-based AI interface:**
   - Install and configure Claude CLI, OpenAI CLI, or similar tools in Termux
   
2. **Access via SSH:**
   - Install `openssh` in Termux: `pkg install openssh`
   - Connect from your desktop to your Android device
   - Use PAI on Android, control from desktop

3. **Use web-based AI interfaces:**
   - Run the observability server on Android
   - Access via mobile browser
   
4. **Sync with desktop:**
   - Use git to sync PAI configuration between devices
   - Keep `~/.claude` directory synced with cloud storage

### Troubleshooting Android Installation

**Package installation fails:**
```bash
# Update repositories
pkg update
pkg upgrade
# Try installing again
```

**Bun installation fails:**
```bash
# Check your architecture first
uname -m
# Output: aarch64 (ARM64) or x86_64 (Intel/AMD)

# For ARM64 (most Android devices)
wget https://github.com/oven-sh/bun/releases/latest/download/bun-linux-aarch64.zip
unzip bun-linux-aarch64.zip
mkdir -p ~/.local/bin
mv bun-linux-aarch64/bun ~/.local/bin/
chmod +x ~/.local/bin/bun
export PATH="$HOME/.local/bin:$PATH"

# For x86_64 devices, use:
# wget https://github.com/oven-sh/bun/releases/latest/download/bun-linux-x64.zip
```

**Permission denied errors:**
```bash
# Ensure storage is set up
termux-setup-storage
# Check permissions
ls -la ~/storage
```

**Command not found errors:**
```bash
# Reload shell configuration
source ~/.bashrc
# Or restart Termux
exit
# Open Termux again
```

### Android Best Practices

1. **Keep Termux updated:** Run `pkg upgrade` regularly
2. **Use external storage carefully:** Android has strict storage permissions
3. **Save battery:** Use wake-lock only when needed
4. **Backup your config:** Sync `~/.claude/.env` to cloud storage
5. **Use tmux:** Keep PAI sessions running when switching apps

---

## Known Platform-Specific Issues (22 Total)

### ✅ FIXED (PR #XXX - Linux Compatibility Fixes)

**Critical Blockers:**
1. ✅ `sed -i ''` syntax (macOS BSD vs GNU sed)
   - **File:** `Packs/kai-voice-system/INSTALL.md` line 337
   - **Fix:** Platform-aware sed with USERNAME fallback
   - **Status:** Fixed with conditional `uname -s` detection

2. ✅ `/opt/homebrew/bin` hardcoded in PATH
   - **Files:** `kai-observability-server/src/observability/manage.sh:8`, `kai-observability-server.md:1316`
   - **Fix:** Conditional PATH based on directory existence
   - **Status:** Fixed with `[ -d "/opt/homebrew/bin" ]` check

**Auto-Start Feature Parity:**
3. ✅ LaunchAgent plist only (no Linux alternative)
   - **File:** `Packs/kai-voice-system/INSTALL.md` Step 9
   - **Fix:** Added systemd user service for Linux
   - **Status:** Linux now has full auto-start support

4. ✅ launchctl commands (macOS-only daemon management)
   - **Context:** Part of LaunchAgent system
   - **Fix:** systemd equivalent provided for Linux
   - **Status:** Platform-specific but both supported

5. ✅ ~/Library/LaunchAgents path (macOS directory structure)
   - **Context:** Part of LaunchAgent system
   - **Fix:** Linux uses `~/.config/systemd/user`
   - **Status:** Platform-specific but both supported

**Documentation:**
6. ✅ VERIFY.md misleading "requires modifications" warning
   - **File:** `Packs/kai-voice-system/VERIFY.md` lines 11-13
   - **Fix:** Updated to reflect Linux is fully supported
   - **Status:** Documentation now accurate

---

### 📋 ALREADY HANDLED (No Action Needed)

**Audio Playback (Fixed in PR #285 - Google TTS):**
17. ✅ afplay calls conditionally executed
    - **File:** `Packs/kai-voice-system/src/voice/server.ts:286-336`
    - **Status:** Runtime platform detection via `process.platform`
    - **Implementation:** macOS uses afplay, Linux auto-detects mpg123/mpv/snap

18. ✅ Linux audio player auto-detection
    - **Status:** Fully implemented with graceful fallbacks
    - **Priority:** mpg123 → mpv → snap/mpv → warn user

19. ✅ Cross-platform notifications
    - **macOS:** osascript (native notification center)
    - **Linux:** notify-send (libnotify)
    - **Status:** Both fully implemented

20. ✅ process.platform checks
    - **Status:** Correct pattern throughout codebase
    - **Note:** Needs Windows support added (future work)

21. ✅ Bun runtime
    - **Status:** Cross-platform, no issues
    - **Installation:** Works on macOS, Linux, Windows

---

### 🔮 MINOR ISSUES (Low Priority)

**Documentation Inconsistencies:**
7. 🔮 Platform check mentions paplay but code doesn't use it
   - **File:** `Packs/kai-voice-system/INSTALL.md` platform check
   - **Impact:** Minor - doesn't block functionality
   - **Fix:** Either add paplay support or remove from docs
   - **Priority:** Low - mpg123/mpv work fine

8. 🔮 /Users/ hardcoded paths in examples
   - **Files:** Various documentation showing macOS examples
   - **Impact:** Documentation only, not actual code
   - **Fix:** Use generic paths like `$HOME` in examples
   - **Priority:** Low - users can adapt examples

**macOS-Specific Features (Can't Test Without macOS):**
9-14. 🔮 LaunchAgent plist internals (6 specific property keys)
    - **Context:** macOS-only format
    - **Status:** Not applicable to Linux
    - **Priority:** Low - macOS functionality works

15. 🔮 osascript for notifications
    - **Status:** Already has notify-send fallback
    - **Priority:** Low - both platforms supported

16. 🔮 ~/Library/Logs for logging
    - **Status:** Already uses `~/.config/pai` on Linux
    - **Priority:** Low - platform-appropriate paths used

---

### ❌ UNSUPPORTED (Windows - Community Contributions Welcome)

22. ❌ Windows support entirely absent
    - **Audio:** No Windows Media Player integration
    - **Notifications:** No Windows Toast notifications
    - **Auto-start:** No Task Scheduler implementation
    - **Shell scripts:** Assume bash (not cmd/PowerShell)
    - **Priority:** Medium - depends on community interest

**How to Contribute Windows Support:**
1. Add Windows audio playback (Windows Media Player, ffplay, or native APIs)
2. Implement Windows Toast notifications
3. Create Task Scheduler auto-start alternative
4. Convert bash scripts to cross-platform Bun/TypeScript
5. Test on Windows 10/11
6. Submit PR following PAI contribution guidelines

---

## Platform Detection Patterns

**Recommended pattern (used throughout PAI):**

```bash
# Shell scripts
OS_TYPE="$(uname -s)"
if [ "$OS_TYPE" = "Darwin" ]; then
  # macOS-specific code
elif [ "$OS_TYPE" = "Linux" ]; then
  # Linux-specific code
else
  echo "Unsupported platform: $OS_TYPE"
fi
```

```typescript
// TypeScript/Bun code
if (process.platform === 'darwin') {
  // macOS-specific code
} else if (process.platform === 'linux') {
  // Linux-specific code
} else if (process.platform === 'win32') {
  // Windows-specific code (future)
}
```

**Anti-patterns to avoid:**
- Hardcoding paths that only exist on one platform
- Assuming package manager locations (Homebrew, apt, etc.)
- Using platform-specific syntax without detection (sed -i '', etc.)
- Skipping platform checks in documentation examples

---

## Testing Requirements

Contributors fixing platform issues should:

1. **Test on target platform** - Don't submit untested code
2. **Document limitations** - Be honest about what you couldn't test
3. **Follow PAI principles** - Simple, transparent, UNIX philosophy
4. **Maintain backward compatibility** - Don't break existing platforms
5. **Add to this document** - Update the inventory with your fixes

**Current test coverage:**
- macOS: Tested by Daniel Miessler
- Linux (Ubuntu/WSL2): Tested by contributors
- Linux (other distros): Community testing
- Windows: Untested

---

## Future Work

**High Priority:**
- Windows audio playback support
- Windows notification support
- Windows auto-start mechanism

**Medium Priority:**
- Test on non-Ubuntu Linux distros (Fedora, Arch, etc.)
- Improve error messages for missing dependencies
- Add platform compatibility checks to installation

**Low Priority:**
- Support for alternative package managers
- Docker/container deployment guide
- Automated multi-platform testing (CI/CD)

---

## How to Report Platform Issues

1. Check this document to see if the issue is already known
2. Test on a clean installation (not your dev environment)
3. Open a GitHub issue with:
   - Platform details (OS, version, package manager)
   - Error message or unexpected behavior
   - Steps to reproduce
   - Proposed solution (if you have one)

**Before submitting:** Try to fix it yourself! PAI is community-driven.

---

## Contribution Guidelines

When contributing platform fixes:

1. **Fix what you can test** - Don't guess, verify
2. **Document what you can't** - Be honest about limitations
3. **Keep it simple** - Follow PAI's UNIX philosophy
4. **Stay transparent** - No magic abstractions
5. **Add tests** - At minimum, manual verification steps

**Good PR example:** "feat: Add systemd auto-start for Linux (tested on Ubuntu 24.04)"

**Bad PR example:** "feat: Universal auto-start abstraction framework for all platforms"

---

## Credits

**Platform compatibility work by:**
- Daniel Miessler - Original PAI implementation (macOS focus)
- PR #285 - Google Cloud TTS provider, Linux audio support
- PR #XXX - Linux compatibility fixes (sed, PATH, systemd)
- Community contributors - Testing and bug reports

Want your name here? Contribute a platform fix!
