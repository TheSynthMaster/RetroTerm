# RetroTerm v2.0 — User and Technical Manual

**Native multi-platform terminal emulator for SSH2, Telnet, Local PTY, PLATO, and SFTP**  
macOS (Cocoa / Metal) · Windows x86_64 & ARM64 (Win32 / Direct3D 11) · Linux x86_64 & ARM64 (SDL2 / OpenGL)

---

## Contents

### Part I — User Guide
1. [About RetroTerm](#1-about-retroterm)
2. [Platforms and Installation](#2-platforms-and-installation)
3. [Connections, Protocols, and Profiles](#3-connections-protocols-and-profiles)
4. [Display Engines, Controls, and Matrix Screensaver](#4-display-engines-controls-and-matrix-screensaver)
5. [SFTP File Transfer Browser](#5-sftp-file-transfer-browser)
6. [Mapping Remote SFTP as a Drive on Windows (WinFsp)](#6-mapping-remote-sftp-as-a-drive-on-windows-winfsp)
7. [Mapping Remote SFTP on macOS and Linux](#7-mapping-remote-sftp-on-macos-and-linux)
8. [A Brief History of PLATO](#8-a-brief-history-of-plato)
9. [PLATO Mode and Keyboard Specifics](#9-plato-mode-and-keyboard-specifics)
10. [VT/SSH Keyboard, Copy/Paste, and Automation Scripting](#10-vtssh-keyboard-copypaste-and-automation-scripting)
11. [Menus, Diagnostics, and Troubleshooting](#11-menus-diagnostics-and-troubleshooting)

### Part II — Technical Reference
12. [Architecture and Shared C11 Core](#12-architecture-and-shared-c11-core)
13. [Modern Dynamic Grid and Screensaver Engines](#13-modern-dynamic-grid-and-screensaver-engines)
14. [SFTP Virtual Filesystem and WebDAV Bridge](#14-sftp-virtual-filesystem-and-webdav-bridge)
15. [PlatoLives Optical Rendering Engines](#15-platolives-optical-rendering-engines)
16. [Building, Cross-Compilation, and Test Suite](#16-building-cross-compilation-and-test-suite)
17. [Credits, Upstream Components, and References](#17-credits-upstream-components-and-references)

---

# Part I — User Guide

## 1. About RetroTerm

RetroTerm is a native cross-platform terminal emulator that brings together a modern VT100/VT220/xterm workstation terminal (`libvterm`), SSH2 (`libssh2`), Telnet, Local PTY, native PLATO sessions, and SFTP file management with the physical vintage display engines of **PlatoLives**.

A central design rule in RetroTerm is the separation between **logical terminal state** and **optical presentation**. Protocol decoding, character grids, graphics primitives, and touch/mouse coordinates operate on an authoritative logical terminal. The selected display engine—whether an orange gas-plasma panel, a Tektronix 4014 storage tube, a PDP-1 radar scope, a Commodore 64 VIC-II monitor, or a crisp high-DPI modern grid—only changes how that state is presented.

---

## 2. Platforms and Installation

| Platform | Binary / Package | Graphics & OS Integration |
| :--- | :--- | :--- |
| **macOS 13+** (Apple Silicon & Intel) | `RetroTerm.app` | Apple Metal (60 FPS), CoreText, macOS Keychain, POSIX `openpty`, `/sbin/mount_webdav` |
| **Windows 10/11** (`x86_64` & `ARM64`) | `RetroTerm-windows-x86_64.exe` / `RetroTerm-windows-arm64.exe` | Direct3D 11, DirectWrite, Windows Credential Manager, ConPTY, optional WinFsp |
| **Linux** (`x86_64` & `ARM64`) | `RetroTerm-linux-desktop-install.tar.gz` | SDL2, OpenGL, OpenSSL 3 (`libcrypto.so.3`), POSIX `openpty`, GVFS (`gio`) |

### Installing on Linux

Extract the Linux desktop archive and run the installer script:

```bash
tar -xzf RetroTerm-linux-desktop-install.tar.gz
./linux_desktop_install/install.sh
```

The script detects `x86_64` or `aarch64`/`arm64`, installs the binary to `~/.local/bin/retroterm`, installs the application icon, and creates `~/.local/share/applications/RetroTerm.desktop`. Make sure SDL2, OpenGL, and OpenSSL 3 runtime libraries are installed on your distribution.

---

## 3. Connections, Protocols, and Profiles

### 3.1 Quick Connect

Press **Cmd+Shift+O** (macOS) or **Ctrl+Shift+O** (Windows and Linux)—or choose **Connection → Quick Connect...**—to open a session without saving a profile.

Quick Connect lets you choose the protocol (`SSH`, `Telnet`, `PTY`, or `PLATO`), host and port (default `22` for SSH, `23` for Telnet, `8005` for PLATO), authentication method for SSH (password or private key), or a local shell for PTY.

### 3.2 Supported Protocols

- **SSH2**: Encrypted non-blocking SSH2 sessions via `libssh2` with OpenSSH `known_hosts` host-key verification, password and private-key authentication, and automatic terminal resize notifications.
- **Telnet**: Asynchronous Telnet client supporting `NAWS` window size negotiation and `TTYPE` terminal identification.
- **Local PTY**: Spawns a local shell directly inside RetroTerm using POSIX `openpty` on macOS/Linux or Windows `ConPTY` on Windows 10/11.
  - **Shell Picker**: Both Quick Connect and Manage Profiles provide a pre-populated dropdown of shells found on the running system—entries from `/etc/shells` on macOS and Linux, or Command Prompt (`cmd.exe`), Windows PowerShell, PowerShell 7 (`pwsh.exe`), WSL, Git Bash, MSYS2, and Cygwin on Windows when installed—alongside a **Default** system shell option and the ability to enter a custom shell path.
- **PLATO**: Direct TCP connection to PLATO IV / CYBIS hosts (such as Cyber1). See [Chapter 8](#8-a-brief-history-of-plato) and [Chapter 9](#9-plato-mode-and-keyboard-specifics) for PLATO-specific behavior and keyboard operation.

### 3.3 Connection Profiles Manager

**Connection → Manage Profiles...** (**Cmd+,** on macOS) stores up to **32 named profiles** in `profiles.ini`. Instead of reconfiguring host, visual engine, and automation every time you connect, a profile saves the complete working context:

- **Identity & Network**: Profile name, protocol (`SSH`, `Telnet`, `PTY`, `PLATO`), host (or shell for PTY), TCP port, username, private key path, and `TERM` string (default `xterm-256color`).
- **Display & Optics**: Preferred Display Engine, phosphor color override, persistence, distortion, beam level, Tektronix baud rate, PDP-1 font/refresh/preset, and C64 preset/border/color scheme. Controls that do not apply to the chosen engine are hidden or disabled.
- **Window & Startup**: Default window scale (`1x`–`4x`), Full Screen toggle, and an optional **Startup Script** selected from your saved scripts. As soon as an SSH, Telnet, or PTY session connects, its assigned startup script runs automatically.

Fresh installations also include four preset profiles:
- `Localhost (PTY)` — Modern mode
- `Star Wars ASCII (Blinkenlights)` — Telnet (`towel.blinkenlights.nl:23`), Tektronix Instant
- `Telehack` — Telnet (`telehack.com:23`), Real Plasma (flat, 2000 ms persistence)
- `Cyber1.org` — PLATO (`cyberserv.org:8005`), Real Plasma (100 ms persistence)

### 3.4 Multiple Windows, Credentials, and Recovery

- **Same Window vs. New Window**: **Connection → Connect with Profile** reuses the current window. **Connection → New Window** or **New Window with Profile** opens an independent terminal window with its own transport, screen state, and optical buffers, and immediately initiates the profile's connection.
- **Saved Passwords**: SSH passwords can be saved securely in **macOS Keychain** or **Windows Credential Manager**. On Linux, saved passwords are stored obfuscated (not encrypted) in `~/.retroterm/credentials.ini`.
- **Reconnect Recovery**: If a connection fails or drops unexpectedly, a non-blocking diagnostics window identifies the cause (DNS failure, connection refused, timeout, unreachable host, authentication failure, host-key mismatch, or mid-session drop) and offers **Retry Now**, a toggleable **30-second Auto-Retry** countdown, or **Abort** (which wipes temporary in-memory retry credentials cleanly).

---

## 4. Display Engines, Controls, and Matrix Screensaver

### 4.1 The 8 Display Engines

Select any mode on the fly from the **View** menu (changes in **View** apply to the active window immediately without overwriting your saved profiles):

| Mode | Visual Character | Notes |
| :--- | :--- | :--- |
| **Real Plasma** | Orange neon gas-discharge panel with optical glow and persistence | Configurable persistence (`100–5000 ms`) and curvature (`None`, `Barrel`, `Cylindrical`) |
| **Crisp Monochrome** | Flat high-contrast monochrome pixels, zero glow or distortion | Ideal for sharp legibility and protocol inspection |
| **Crisp Color** | Flat multi-color ANSI/xterm or PLATO color pixels | Zero optical blur or curvature |
| **Real Color CRT** | RGB shadow-mask CRT with Gaussian electron beam, scanlines, bloom, and curvature | Adjustable beam level (`Standard`, `High`, `Ultra`), decay (`20–5000 ms`), and curvature |
| **Tektronix 4014** | Persistent P31 green storage-tube vectors with write flare and page-erase flash | Selectable baud pacing (`Instant` or `110`–`9600 bps`), erase-flash toggle, and accurate dark flood-gun background noise |
| **DEC PDP-1 Type 30** | Point-plotting CRT with P7 cascade phosphor (blue-white flash into yellow-green persistence) | Selectable 1961/1964 fonts, refresh rate (`5–60 Hz`), and Historic/Fast/Modern presets |
| **Commodore 64 (VIC-II)** | Classic 8-bit monitor presentation | Presets (`RF / Composite`, `1702 CRT`, `Crisp Hires`), `12.5%` border (`Off`, `Static`, `Turbo Loader FX`), and color schemes (`Classic Blue`, `Multi-Color`, `Amber`, `Green`) |
| **Modern (High-Fidelity)** | Resizable variable-column/row GPU text grid (`Cmd+Shift+M` / `Ctrl+Shift+M`) | Full UTF-8, TrueColor, `Ctrl+=` / `Ctrl+-` / `Ctrl+0` font sizing, Block/Underline/Bar cursor + blink. *Disabled in PLATO mode.* |

### 4.2 Color Overrides, Sizing, and HUD

- **Override Color**: In **Real Plasma**, **Crisp Monochrome**, **Real Color CRT**, and **PDP-1**, you can override the phosphor tint to **Default**, **Green (P31)**, **Amber (P40)**, or **White (P4)**. Themes across menus, HUD, and dialogs adapt accordingly.
- **Window Size & Full Screen**: **View → Window Size** offers `1x`–`4x` integer scales (`640×400` base for VT, `512×512` base for PLATO); resizing freely scales with aspect fit. **F11** toggles Full Screen. On Linux, **View → Hide Menu Bar** hides the top bar until the pointer touches the top edge.
- **Performance HUD (`F12`)**: Displays live frame rate (`FPS`), frame/adapter/optical timings, CPU load, RX/TX throughput (`KB/s`), active protocol, display mode, and Tektronix queue occupancy.

### 4.3 Matrix Screensaver

RetroTerm includes an idle **Matrix Digital Rain** screensaver built around a custom `10×14` bitmap font of horizontally mirrored half-width Katakana, numerals, and symbols.

- **Controls**: Open **View → Matrix Screensaver** to choose **Start Screensaver Now** or set the inactivity timeout (**Disabled**, **After 1 Minute**, **After 2 Minutes** [default], **After 5 Minutes**, **After 10 Minutes**). The timeout is saved in `settings.ini`.
- **Theme-Matched Rendering**: The screensaver follows the active display engine and any active phosphor override:
  - *Phosphor Override (`P31 Green`, `P40 Amber`, `P4 White`)* takes precedence across all engines while keeping physical effects (scanlines, micro-grain, or curvature).
  - *Tektronix 4014*: Matrix green with transient write flare on leading glyphs and subtle dielectric micro-grain.
  - *Modern*: Classic film Matrix green on pure `#000000` at native HiDPI canvas resolution.
  - *PLATO (Real Plasma / Crisp Mono)*: Orange neon plasma glow with optional plasma curvature.
  - *DEC PDP-1 Type 30*: Dual-phosphor P7 radar scope (ice-blue flash on the leading glyph fading into long-persistence yellow-green sustain and radial bloom).
  - *Commodore 64 (VIC-II)*: C64 monochrome VIC-II palette on `$D021` Dark Blue (including optional `$D020` border, or Amber/Green if selected).
  - *Real Color CRT / Crisp Color*: Random vivid color per rain column with bright white-tinted heads, CRT scanlines, and optional CRT curvature.
- **Non-Destructive & Safe Wake-up**: Starting the screensaver never clears or alters the underlying terminal buffer. Any key press, mouse click, scroll wheel event, or deliberate pointer movement (`>= 4 px`) immediately dismisses the screensaver and **consumes the wake event** so stray keystrokes or clicks are never sent to your SSH, Telnet, PTY, or PLATO session.

---

## 5. SFTP File Transfer Browser

When connected over **SSH**, the top-level **SFTP** menu becomes available (it is hidden in Telnet, PTY, and PLATO sessions).

Choose **SFTP → Open SFTP File Transfer...** (**Cmd+Shift+F** on macOS, **Ctrl+Shift+F** on Windows and Linux) to open the dual-pane SFTP browser over the active SSH connection without re-entering credentials.

Capabilities:
- Navigate local and remote directories side by side.
- **Upload** files from your computer to the remote host and **Download** remote files locally.
- Create directories (`mkdir`), **Rename**, **Delete**, refresh listings, and inspect file sizes and permissions.

---

## 6. Mapping Remote SFTP as a Drive on Windows (WinFsp)

On Windows, RetroTerm can mount a remote SFTP path directly as a native Windows drive letter (`D:`–`Z:`) using **WinFsp**.

### Prerequisites

- **WinFsp** (Windows File System Proxy) must be installed on the system.
- On first launch, RetroTerm offers a one-click prompt to install or configure WinFsp if it is not detected. You can reopen this setup dialog at any time from **SFTP → Install / Configure WinFsp (SFTP Drive)...**.

### Mounting and Unmounting

1. Establish an active **SSH** session.
2. Open **SFTP → Map Remote Path as Windows Drive...**.
3. Configure:
   - **Drive Letter**: Choose `Auto` (selects the first available letter) or a specific drive letter (`D:`–`Z:`).
   - **Remote Root**: Remote directory to expose as the root of the drive (e.g., `/home/user` or `/`).
   - **Read-Only**: Optionally mount the remote filesystem read-only to prevent accidental writes or deletions.
4. Click **Mount**. The remote directory appears in Windows File Explorer and is accessible to standard Windows applications.
5. To unmount, select **SFTP → Unmount Active SFTP Drive**. Disconnecting the SSH session or closing RetroTerm also unmounts the drive cleanly.

---

## 7. Mapping Remote SFTP on macOS and Linux

On macOS and Linux, RetroTerm exposes the active SFTP session through an internal loopback WebDAV bridge bound strictly to `127.0.0.1`, avoiding external `sshfs` or kernel-extension dependencies.

### 7.1 macOS (`mount_webdav`)

1. Establish an active **SSH** session.
2. Choose **SFTP → Map Remote Path as Mac Folder...**.
3. Configure:
   - **Mount Point**: Local folder path (defaults to `~/RetroTermMount`, created automatically if needed).
   - **Remote Root**: Remote directory to expose as the mounted root.
   - **Read-Only**: Optional read-only protection.
4. RetroTerm starts its local WebDAV server and mounts it via `/sbin/mount_webdav`, opening the mounted folder in Finder.
5. Choose **SFTP → Unmount Active SFTP Mount** (or disconnect/quit) to unmount cleanly.

### 7.2 Linux (GVFS / `gio mount`)

1. Establish an active **SSH** session.
2. Choose **SFTP → Map Remote Path as Linux Folder...**.
3. Configure the **Remote Root** and optional **Read-Only** flag.
4. RetroTerm starts the loopback WebDAV server (`dav://127.0.0.1:<port>/`) and mounts it via `gio mount`, opening it in your file manager (`gio open`). On desktop environments without GVFS `gio`, RetroTerm falls back to opening the local HTTP WebDAV address via `xdg-open`.
5. Choose **SFTP → Unmount Active SFTP Mount** (or disconnect/quit) to run `gio mount -u` and stop the local bridge.

---

## 8. A Brief History of PLATO

**PLATO** (*Programmed Logic for Automatic Teaching Operations*) began in **1960** at the University of Illinois at Urbana–Champaign, led by **Donald Bitzer** at the Computer-based Education Research Laboratory (**CERL**). While most 1960s and 1970s terminals were restricted to scrolling lines of ASCII text, PLATO was built from the outset as an interactive graphical learning and collaboration network.

A defining milestone arrived with **PLATO IV** (early 1970s) and the **Owens-Illinois Digivue 512×512 plasma display panel**, co-invented by Bitzer and H. Gene Slottow:
- Each of the **512 × 512** dots was a tiny neon-argon gas-discharge cell trapped between glass plates, glowing in PLATO's signature **orange** emission.
- Unlike a raster CRT, the plasma panel had **inherent bistable memory**: once a cell was ignited (`WRITE`) or extinguished (`ERASE`), it stayed in that state without flicker or refresh memory—allowing high-resolution line graphics, custom downloadable character sets, and selective pixel erasure (`WRITE` / `ERASE` / `INVERSE`) over modest 1200 bps phone lines.
- Terminals also featured an optical **infrared touch panel** (a `16×16` grid of LED/photodetector beams across the glass) so students could tap directly on diagrams and menus.

Powered by the **TUTOR** programming language on Control Data Corporation (CDC) mainframes, PLATO gave rise to many hallmarks of modern online culture: real-time multi-user chat (**Talkomatic**), structured message boards (**PLATO Notes**), online multiplayer dungeon crawlers and flight simulators, and interactive music and physics lessons. Later **MicroTutor** terminals (such as the CDC **IST-II** / **IST-III** based on 8080/Z80 microprocessors) could even boot standalone `.mte` lesson floppy disks offline without a mainframe link.

Today, that ecosystem is kept alive by preservation systems such as **Cyber1** (`cyberserv.org:8005`), to which RetroTerm connects natively.

---

## 9. PLATO Mode and Keyboard Specifics

Selecting the **PLATO** protocol (e.g., via the `Cyber1.org` preset or Quick Connect) switches RetroTerm into native PLATO terminal mode:
- **Display**: Logical `512 × 512` bitmapped screen (`64 × 32` text grid) rendered through the selected vintage optical engine (`Modern` mode is disabled because it is a variable-grid VT renderer).
- **Touch Panel**: Left-clicking inside the active square sends both fine-grained and legacy coarse PLATO touch coordinates. When barrel or cylindrical distortion is active, RetroTerm mathematically inverts the distortion so clicks land on the exact logical coordinate under the pointer.
- **Context-Aware Menus**: The `SFTP` menu is replaced by **Microtutor**, and **Keyboard** / **Scripts** switch to PLATO-specific profiles and scripts.

### 9.1 Understanding the PLATO Keyboard

The physical PLATO IV / IST keyboard was very different from a modern PC or Mac keyboard. Instead of relying solely on ASCII control codes, PLATO applications expect dedicated **function keys** whose meaning is part of the PLATO user interface:

| PLATO Key | Purpose in PLATO / TUTOR |
| :--- | :--- |
| **NEXT** / **SHIFT-NEXT** | Advance to the next page, submit an answer, or confirm an input (`Return` / `Enter` by default) |
| **BACK** / **SHIFT-BACK** | Return to the previous page or lesson menu (`Esc` or `F8`) |
| **STOP** / **SHIFT-STOP** | Exit the current unit, lesson, or prompt (`F10` or `F4`; `SHIFT-STOP` often logs off or returns to main menu) |
| **HELP** / **SHIFT-HELP** | Request lesson help or hint (`F1` / `F6`) |
| **LAB** / **SHIFT-LAB** | Open lab/calculator/auxiliary tools (`F2` / `F7`) |
| **DATA** / **SHIFT-DATA** | Open data or reference display (`F3` / `F9`) |
| **EDIT** / **SHIFT-EDIT** | Invoke line/character editor (`F5`) |
| **ERASE** / **SHIFT-ERASE** | Erase character or entire input field (`Backspace` / `Delete` / `Left Arrow`) |
| **COPY** / **SHIFT-COPY** | Copy expression/line in editors (`Ctrl+C` / `Alt+C`) |
| **MICRO** / **FONT** | Prefix for special/Greek/math characters (`MICRO`, `Ctrl+M`) and alternate font (`FONT`, `Ctrl+F`) |
| **SUPER** / **SUB** | Superscript (`Up Arrow` / `Page Up` / `Ctrl+P`) and Subscript (`Down Arrow` / `Page Down` / `Ctrl+Y`); shifted forms ascend/descend |
| **SQUARE** / **ACCESS** | Special PLATO square symbol (`Ctrl+Q`) and Access prefix (`Shift+Ctrl+Q`) for accented/special glyphs |
| **ANS** / **TERM** | Answer-checking (`Ctrl+A`) and Terminal-command prompt (`Ctrl+T`) |
| **ASSIGN (`<=`) / MULTIPLY (`×`) / DIVIDE (`÷`)** | TUTOR assignment and math symbols (`Alt+Left`, `Ctrl+X`, `Ctrl+G`) |

### 9.2 Default PLATO Keyboard Shortcuts (Standard Profile)

Open **Keyboard → PLATO Keyboard Reference...** (**Cmd+K** / **Ctrl+K**) at any time to view the built-in quick-reference card.

| PLATO Key | Physical Shortcut (Windows / Linux / macOS) | Shifted Form |
| :--- | :--- | :--- |
| **NEXT** | `Return` / `Enter` / `Ctrl+N` | `Shift+Return` / `Shift+Ctrl+N` |
| **BACK** | `Esc` / `F8` / `Ctrl+B` | `Shift+Esc` / `Shift+F8` |
| **STOP** | `F10` / `F4` / `Ctrl+S` / `Alt+S` | `Shift` + any STOP shortcut |
| **ERASE** | `Backspace` / `Delete` / `Left Arrow` / `Ctrl+R` | `Shift` + any ERASE shortcut |
| **HELP** | `F1` / `F6` / `Ctrl+H` / `Alt+H` | `Shift` + any HELP shortcut |
| **LAB** | `F2` / `F7` / `Ctrl+L` / `Alt+L` | `Shift` + any LAB shortcut |
| **DATA** | `F3` / `F9` / `Ctrl+D` / `Alt+D` | `Shift` + any DATA shortcut |
| **EDIT** | `F5` / `Ctrl+E` / `Alt+E` | `Shift+F5` / `Shift+Ctrl+E` |
| **COPY** | `Ctrl+C` / `Alt+C` | `Shift+Ctrl+C` / `Shift+Alt+C` |
| **MICRO / FONT** | `Ctrl+M` / `Ctrl+F` (or `Alt+M` / `Alt+F`) | — |
| **SUPER / SUB** | `Up`/`Page Up`/`Ctrl+P` and `Down`/`Page Down`/`Ctrl+Y` | `Shift` + key |
| **SQUARE / ACCESS** | `Ctrl+Q` / `Alt+Q` | `Shift+Ctrl+Q` / `Shift+Alt+Q` (`ACCESS`) |
| **ANS / TERM** | `Ctrl+A` / `Ctrl+/` and `Ctrl+T` | — |
| **TAB / ASSIGN** | `Tab` / `Right Arrow` and `Alt+Left Arrow` (`<=`) | — |
| **MULTIPLY / DIVIDE** | `Ctrl+X` / `Alt+X` and `Ctrl+G` / `Alt+G` | — |

### 9.3 PLATO Keyboard Profiles, Double-Tap, and Arrow Modes

PLATO games and lessons often use different directional conventions. Under the **Keyboard** menu in PLATO mode:
- **Built-in Keyboard Profiles**: Switch instantly between **Standard PLATO**, **WASD Gaming**, **WASX / Avatar**, **Numpad Arrows** (8-way movement), and **WADX** (traditional TUTOR directional diamond), or cycle presets with **Cmd+Shift+G** (macOS) / **Ctrl+Shift+G** (Windows & Linux).
- **Manage PLATO Keyboard Profiles...** (**Cmd+Shift+K** / **Ctrl+Shift+K**): Duplicate any built-in profile, click **Press Key to Bind...**, and remap physical keys to any PLATO action, raw byte, or sequence. Custom profiles are saved as JSON and take effect without reconnecting.
- **Double-Tap Access/Shift-Letters**: When enabled, tapping selected keys twice quickly (~350 ms) emits their shifted or Access-prefixed PLATO equivalent.
- **Cycle Arrow Mode**: Quickly toggles arrow-key behavior between `Standard`, `WASD`, `WASX`, and `WADX`.

### 9.4 Offline MicroTutor (`.mte` Floppy Images)

When in PLATO mode (on any display engine except Tektronix 4014), the **Microtutor** top-level menu provides:
- **Load and Boot from MTE...**: Select a compatible `.mte` MicroTutor disk image and execute it locally on the integrated Z80/8080 and IST-II floppy runtime—no network connection required. Exiting or connecting to a host resets terminal memory and programmable fonts cleanly.
- **Enable Z80 Diagnostic Log**: Traces Z80 MicroTutor execution for debugging or preservation work.

---

## 10. VT/SSH Keyboard, Copy/Paste, and Automation Scripting

### 10.1 VT100 / VT220 Keymaps and Signals

For SSH, Telnet, and PTY sessions, **Keyboard** provides:
- **Built-in Profiles**: `Standard VT100 / Linux` (DEL `0x7F`, ANSI arrows), `Legacy UNIX / BSD / Solaris` (BS `0x08`), and `Vim / Tmux / Application` (DECCKM Application cursor mode), plus **Manage Keyboard Profiles...** to record custom VT/ANSI bindings.
- **Signals & Control Characters**: Direct menu items for `Send Break`, `Ctrl+C` (`SIGINT`), `Ctrl+D` (`EOF`), `Ctrl+Z` (`SIGTSTP`), `Ctrl+\` (`SIGQUIT`), `Ctrl+L`, `Ctrl+A`, `Ctrl+E`, `Ctrl+W`, `Ctrl+U`, `Ctrl+K`, `Ctrl+R`, `Alt as Meta`, and VT Function/Editing keys.
- **Keyboard Reference** (**Cmd+K** / **Ctrl+K**): Summarizes ANSI vs. DECCKM arrows, VT220 navigation sequences, F1–F12 (`PF1`–`F24`), control codes, and Alt-as-Meta navigation.

### 10.2 Copy, Paste, and Live Text Buffer

- **Mouse Selection**: Drag across the terminal in *either* Modern or any vintage optical mode to highlight text; releasing the mouse copies the selection automatically with an on-screen HUD toast. (Hold **Alt** to select text even when a remote TUI like `vim` or `htop` has mouse reporting active.)
- **Right-Click Paste**: Right-clicking pastes clipboard text using paced transmission (respecting bracketed paste when active in VT modes). **Edit → Cancel Paste** aborts a long paste in progress.
- **Edit Tools**: **Copy All Text Verbatim**, **Copy All Text Compact** (**Cmd+Shift+C** / **Ctrl+Shift+C**), **Copy Selected Text...**, **Copy Screen Image**, and **Tools → Show Live Text Buffer** (**Cmd+Shift+T** / **Ctrl+Shift+T**) for inspecting the live character matrix independently of optical effects.

### 10.3 State-Aware Automation Scripting

Open **Scripts → Manage Scripts...** to create, clone, or bind automation scripts (up to 16 KB, 128 variables, optional hotkey). Unlike blind macro recorders, scripts inspect what is actually on screen:

- `wait_text [regex] "<pattern>" [in x0,y0..x1,y1] [timeout <dur>] [into x,y]`
- `find_text [regex] "<pattern>" [in x0,y0..x1,y1] [into x,y]`
- `send "<text>"` / `send-env <VAR>` (redacts secrets from diagnostic logs)
- `key <NAME>` (e.g., `NEXT`, `SHIFT-STOP`, `ENTER`, `ESC`, `CTRL-C`)
- Control flow: `@label:`, `goto`, `if ... goto`, `for ... next`, `let`, `wait <dur>`
- Connection control: `connect`, `connect default`, `connect last`, `connect "<Profile>"`, `reconnect`, `disconnect`
- Diagnostics: `screenshot`, `copy-all`, `copy-area`, `diagnostic-log on|off`

Press **Cmd+Shift+X** (macOS) or **Ctrl+Shift+X** (Windows/Linux)—or choose **Scripts → Cancel Script Execution**—to stop a running script immediately. A pulsing LED indicator in the top-right corner of the terminal shows when a script is active.

---

## 11. Menus, Diagnostics, and Troubleshooting

### 11.1 Top-Level Menu Map

| Menu | Purpose |
| :--- | :--- |
| **Connection** | Quick Connect, Connect Default, Connect with Profile, Manage Profiles, New Window, New Window with Profile, Disconnect, Close Window, Quit |
| **View** | All 8 Display Engines, per-engine options (Plasma, CRT, Tektronix, PDP-1, C64, Modern), Window Size (`1x`–`4x`), Full Screen (`F11`), Performance HUD (`F12`), Matrix Screensaver, Override Color, Hide Menu Bar (Linux) |
| **Keyboard** | Active keyboard profile selection, Manage Keyboard Profiles, Keyboard Reference, plus PLATO double-tap / arrow modes (in PLATO mode) or Signals & Controls (in VT mode) |
| **Edit** | Copy Verbatim/Compact/Selected/Screen Image, Paste, Cancel Paste |
| **Tools** | Show Live Text Buffer, Enable Network Diagnostic Log, Enable Renderer Performance Log |
| **SFTP** *(SSH only)* | SFTP File Transfer Browser, Map Remote Path as Drive/Folder, Unmount, WinFsp Setup (Windows) |
| **Microtutor** *(PLATO only)* | Load and Boot from MTE, Enable Z80 Diagnostic Log |
| **Scripts** | Run saved scripts, Manage Scripts, Cancel Script Execution, Scripting Reference |
| **Help** | About RetroTerm |

### 11.2 Troubleshooting

| Symptom | Likely Cause | Action |
| :--- | :--- | :--- |
| SSH/Telnet/PLATO fails or times out | Host unreachable or wrong port | Check endpoint in Manage Profiles; use **Retry Now** or **30s Auto-Retry** |
| Modern mode option is grayed out | Active session uses **PLATO** | Switch to SSH, Telnet, or PTY to use Modern mode |
| Windows SFTP Drive says WinFsp missing | WinFsp runtime not installed | Open **SFTP → Install / Configure WinFsp...** and install WinFsp |
| Linux SFTP folder opens in browser instead of file manager | GVFS `gio` is unavailable | Install `gvfs` / `gvfs-backends` for native file-manager mounting |
| PLATO keys do not match game or lesson | Wrong PLATO keyboard profile | Press `Cmd+Shift+G` / `Ctrl+Shift+G` to cycle presets or customize bindings |
| MicroTutor `.mte` will not boot | Tektronix 4014 mode is active | Switch to Real Plasma, Crisp Mono, CRT, PDP-1, or C64 before booting `.mte` |
| Tektronix drawing lags on large pages | Low emulated baud rate (`110–300 bps`) | Raise Tektronix speed or select **Instant**; inspect queue in HUD (`F12`) |
| Screensaver wakes on tiny cursor jitter | Normal hardware mouse vibration | Move mouse `>= 4 px`, click, or press any key to dismiss |

---

# Part II — Technical Reference

## 12. Architecture and Shared C11 Core

RetroTerm separates portable terminal logic (`src/core/` and `src/transport/`) from thin native presentation layers (`src/ui/mac/`, `src/ui/win/`, `src/ui/linux/`).

- **Logical Terminal & VT Adapter (`rt_vt`, `rt_adapter`)**: Wraps `libvterm` with dynamic row/column sizing, scrollback/alternate-screen state, SGR 256/TrueColor attributes, OSC 0/2/52 handlers, DECSCUSR cursor shapes, and mouse tracking (`X10`, `SGR`, `URXVT`). For vintage modes, `rt_adapter` maps the VT cell grid into `plato_terminal_t` (`640×400` logical raster for VT modes; `512×512` native bitboard for PLATO sessions).
- **Transports (`rt_ssh`, `rt_telnet`, `rt_pty`, `plato_transport`)**: Non-blocking socket/PTY I/O with separate RX ring buffering so optical rendering never stalls network arrival.
- **Connection Diagnostics (`rt_conn_diag`)**: Shared failure classifier mapping OS/SSH/Telnet/PLATO socket errors to structured categories and user-facing diagnostic guidance.
- **Configuration & Secrets**:
  - `profiles.ini` (`rt_profile`), `settings.ini` (`rt_settings`), `scripts.ini` (`rt_script`), and JSON keymap profiles (`rt_keymap`) are stored in the OS application config directory (`%APPDATA%\RetroTerm` on Windows, `~/Library/Application Support/RetroTerm` on macOS, `~/.config/RetroTerm` on Linux).
  - SSH secrets use macOS Keychain (`Security.framework`), Windows Credential Manager (`CredWriteW`/`CredReadW`), or `~/.retroterm/credentials.ini` on Linux, with explicit `rt_ssh_secure_zero()` scrubbing on in-memory buffers.

---

## 13. Modern Dynamic Grid and Screensaver Engines

### 13.1 Modern Dynamic Grid (`rt_modern`, `rt_modern_font`)

In `Modern (High-Fidelity)` mode, `rt_modern` renders directly into a HiDPI BGRA framebuffer matching the window's drawable pixel dimensions:
- Glyphs are rasterized on demand into a cached atlas using **CoreText** (macOS), **DirectWrite** (Windows), or embedded **Roboto Mono** via `stb_truetype` (Linux), with geometric vector fallback for Powerline and box-drawing glyphs (`U+2500`–`U+259F`).
- Dirty-line tracking updates only modified cells; the resulting BGRA surface is uploaded to a `MTLTexture` (Metal), `ID3D11Texture2D` (Direct3D 11), or `GL_TEXTURE_2D` (`GL_BGRA`, OpenGL).

### 13.2 Matrix Screensaver (`rt_screensaver`)

- **Glyph Set**: 64 embedded `10×14` bitmaps (44 horizontally mirrored half-width Katakana glyphs, 10 mirrored numerals `0–9`, 10 symbols `:+*=-<>|".`), scaled proportionally to maintain ~64–80 columns at any window or optical resolution.
- **Simulation**: Each column tracks a sub-pixel descending head, randomized speed, depth gain, drop color index, and tail length. Cells retain continuous `energy` and transient `flare` state, mutating glyphs stochastically while active.
- **Theme Resolution**: `phosphor_color != 0` (`1=P31 Green`, `2=P40 Amber`, `3=P4 White`) overrides the color ramp across all modes while preserving engine physics (scanlines, micro-grain, or distortion). Otherwise, native ramps match each engine (`Tektronix` P31 write-flare + grain, `PLATO` orange plasma + curvature, `PDP-1` dual-phosphor P7 flash + sustain + bloom, `C64` `$D021`/`$D020` palette, `Modern` `#000000` film green, `Real Color CRT` / `Crisp Color` per-column vivid primaries).
- **Curvature Cache**: When `plasma_distortion` or `crt_distortion` is active, a `(w × h)` barrel/cylindrical lookup table is precomputed once per `(w, h, mode)` change and applied in a single pass.

---

## 14. SFTP Virtual Filesystem and WebDAV Bridge

- **`rt_sftp_vfs`**: Provides a thread-safe virtual filesystem abstraction (`readdir`, `stat`, `read`, `write`, `create`, `unlink`, `rename`, `mkdir`, `rmdir`) over an active `libssh2_sftp` session.
- **Windows (`rt_win_sftp_drive`, `rt_winfsp_loader`)**: Dynamically loads `winfsp-x64.dll` / `winfsp-a64.dll` at runtime (so RetroTerm launches cleanly even when WinFsp is not installed) and translates WinFsp filesystem callbacks onto `rt_sftp_vfs`, enforcing `STATUS_MEDIA_WRITE_PROTECTED` when read-only is selected.
- **macOS & Linux (`rt_webdav_sftp`)**: Runs a local multi-threaded HTTP/1.1 + RFC 4918 WebDAV server (`OPTIONS`, `PROPFIND`, `GET`, `HEAD`, `PUT`, `DELETE`, `MKCOL`, `MOVE`, `COPY`, `LOCK`/`UNLOCK`) bound exclusively to `127.0.0.1` on an ephemeral port, backed by `rt_sftp_vfs`. macOS mounts it via `/sbin/mount_webdav`; Linux mounts it via `gio mount`.

---

## 15. PlatoLives Optical Rendering Engines

*Adapted from the PlatoLives 5.0.0 Technical Appendices.*

All optical renderers share a single BGRA `uint32_t` pixel pipeline (`2048×2048` for `512×512` PLATO sessions; `2560×1600` for `640×400` VT sessions, i.e., `4×` linear oversampling).

### 15.1 Real Plasma (`PLATO_DISPLAY_REAL_PLASMA`)

Models the Owens-Illinois Digivue `512-60` AC neon gas-discharge panel:
- **4×4 Sub-Pixel Footprint**: Each logical pixel expands to a calibrated `4×4` weighted cell kernel (`0.65` corners, `0.82` edges, `1.00` center) multiplied by a fixed spatial gain `[0.98, 1.02]` to reproduce static panel variation without temporal shimmer.
- **Neon Persistence**: Active cell energy decays exponentially as `E(t + dt) = E(t) * exp(-dt / tau)` with `tau = T_decay / 10`, clamping below `0.0001` so static screens reach zero-work idle state.
- **Dual Separable Blur & Density Fusion**: Two running-sum separable box blurs (`R = 4` for inter-cell diffusion, `R = 12` for front-glass scatter) combine with orange-biased core fusion (`fused = core + 0.82 * n * (1 - core)` for dense areas).
- **Distortion & Inverse Touch**: Cylindrical (`x_d = x(1 + k*x^2)`) or barrel distortion preserves the host touch contract by running pointer coordinates through the inverse transform prior to PLATO coordinate conversion.

### 15.2 Real Color CRT (`PLATO_DISPLAY_REAL_CRT`)

- **3-Tap Gaussian Beam**: Horizontal energy distribution across `{-1, 0, +1}` with selectable `sigma` (`1.8`, `2.4`, `2.8` for Standard/High/Ultra).
- **Chromatic Shadow Mask & Scanlines**: Four-position repeating sub-pixel mask (`R`, `G`, `B`, neutral matrix wire) modulated by soft cathode scanlines `m_scan(y_s) = 1 - d * (1 - cos(phi)) / 2` (`d` = `0.22` / `0.20` / `0.18`).
- **Continuous P22 Decay & Calibrated Curvature**: Logical color `L` and energy `E(t)` are stored separately (`tau = T_CRT / 10`), followed by chromatic additive bloom (`B_4 + B_12`) and cylindrical (`k = 0.018`) or 4:3 barrel distortion (`k_x = 0.018`, `k_y = 0.012`) with bounded Newton/iterative inversion for touch/mouse hit-testing.

### 15.3 Tektronix 4014 DVBST (`PLATO_DISPLAY_TEKTRONIX`)

Emulates a **Direct-View Bistable Storage Tube** without an anachronistic vector display list:
- **8192-Slot Ring Buffer & Token-Bucket Baud Pacer**: Vector, character, and page-erase events enter `term->vector_queue` (`8192` slots) and are consumed using `credit_bits += dt * baud` (clamped to `baud * 0.250s`), billing `40` bits per vector line, `10` bits per character, and `20` bits per Page Erase (`ESC FF`).
- **Dual-Grid Spatial Decoupling**: Two `2048×2048` floating-point grids (`~48 MB` total):
  - `tek_stored`: Clamped `[0.0, 1.0]` bistable electrostatic charge sustained indefinitely by flood guns.
  - `tek_flare`: Uncapped transient photon flux decaying with `tau_flare = 0.045 s`.
- **Sub-Pixel Splatting & Vertex Dwell**: `plato_tek_splat_line()` advances in `0.8` sub-pixel steps with a `2D` Gaussian kernel (`sigma = 1.0`, `r = 2`) and injects endpoint dwell spots to reproduce electrostatic deflection settling.
- **Dual-Mode Erase**: Individual erase vectors carve a `r = 3` sub-pixel void through `tek_stored` (preserving severed line intersections), while character/block updates run `plato_tek_resync_box_from_fb()` over an expanded 1-PLATO-pixel halo margin.
- **Non-Linear P31 Synthesis**: Single-pass compositing combines flood-gun background bias (`0.015`), stored luminance (`* 0.92`), linear green halo (`f * 2.20`), quadratic white-hot core saturation (`f^2 * 2.00`), and the 400 ms quadratic Page Erase flash (`(1 - t/0.400)^2`).

### 15.4 DEC PDP-1, Commodore 64 VIC-II, and MicroTutor

- **DEC PDP-1 Type 30 (`plato_pdp1`)**: Point-plotting scope simulation with dual-stage P7 phosphor (fast blue-white excitation transitioning to long-decay yellow-green persistence) and selectable 1961/1964 glyph sets.
- **Commodore 64 VIC-II (`plato_c64`)**: Authentic VIC-II color and raster presets (`RF / Composite`, `1702 CRT`, `Crisp Hires`), `12.5%` border rendering with optional Turbo Loader raster bars, and Classic / Multi-Color / Amber / Green palettes.
- **MicroTutor (`plato_microtutor`, `z80emu`)**: Integrates the open-source `z80emu` CPU core by Lin Ke-Fong with an emulated CDC IST-II floppy controller to boot `.mte` disk images directly against `plato_terminal_t`.

---

## 16. Building, Cross-Compilation, and Test Suite

### Universal Build (`build_all.sh`)

Run from macOS with Xcode Command Line Tools, CMake, `libssh2`, and `zig` installed (plus Homebrew `sdl2` and `openssl` headers for the Linux desktop targets):

```bash
./build_all.sh
```

Outputs:
- `build/RetroTerm-windows-x86_64.exe`
- `build/RetroTerm-windows-arm64.exe`
- `build/RetroTerm-linux-x86_64` & `build/RetroTerm-linux-arm64`
- `build/RetroTerm-linux-desktop-install.tar.gz`
- `build-mac/RetroTerm.app`

### Native macOS CMake Build & CTest

```bash
cmake -S . -B build-mac
cmake --build build-mac -j
ctest --test-dir build-mac --output-on-failure
```

The automated test suite executes **15 CTest targets**:
`smoke`, `vt`, `adapter`, `all_engines`, `ssh`, `sftp_vfs`, `script`, `keymap`, `telnet`, `pty`, `settings`, `modern`, `conn_diag`, `plato_core`, `screensaver`.

---

## 17. Credits, Upstream Components, and References

- **Author**: Fabio Montarsolo
- **Upstream & Open-Source Components**:
  - **PlatoLives** — Vintage optical engines, PLATO protocol/terminal core, and MicroTutor integration
  - **libvterm** — VT100/VT220/xterm terminal state machine
  - **libssh2** — Asynchronous SSH2 and SFTP transport
  - **z80emu** (Lin Ke-Fong) — Z80/8080 CPU emulation for offline MicroTutor
  - **stb_truetype** (Sean T. Barrett) — Software TrueType font rasterization
  - **WinFsp** — Windows File System Proxy support for SFTP drive mounting
- **Historical & Technical References**:
  - Tom Hunter, Paul Koning, Joe Stanton, Bill Galcher, Steve Zoppi, Dale Sinder — *Pterm: A PLATO Terminal Emulator*
  - Donald L. Bitzer & H. Gene Slottow — *"The Plasma Display Panel — A Digitally Addressable Display with Inherent Memory"*, AFIPS, 1966
  - Jack Stifle — *The PLATO IV Architecture* (CERL Report X-20, 1971/1972) & *The PLATO IV Terminal: Description of Operation* (1974)
  - Control Data Corporation — *CDC IST-III Terminal Reference Manual* (pub. 84002390)
  - Brian Dear — *The Friendly Orange Glow: The Untold Story of the PLATO System and the Dawn of Cyberculture*, Pantheon Books, 2017
  - Tektronix 4014 Direct-View Bistable Storage Tube technical documentation
