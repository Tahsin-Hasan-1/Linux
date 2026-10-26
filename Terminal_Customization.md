# Linux Terminal Customization: Kali Linux, Zsh, Kitty, Powerlevel10k, Pokémon Colorscripts , Syntax hightlighting and Auto Suggestion

This guide documents terminal customization on **native Linux** and **Kali Linux on WSL2/WSLg**.

---

## 0. Before you start

Open a terminal and identify your environment:

```bash
cat /etc/os-release | head
echo "Shell: $SHELL"
uname -a
```

For WSL, check from **Windows PowerShell**:

```powershell
wsl --version
wsl --update
```

To restart WSL after updating, from PowerShell:

```powershell
wsl --shutdown
```

Reopen your Linux distribution afterwards.

### Back up existing configuration

Do this before editing files:

```bash
mkdir -p ~/.config/kitty
[ -f ~/.zshrc ] && cp ~/.zshrc ~/.zshrc.backup.$(date +%Y%m%d-%H%M%S)
[ -f ~/.config/kitty/kitty.conf ] && cp ~/.config/kitty/kitty.conf ~/.config/kitty/kitty.conf.backup.$(date +%Y%m%d-%H%M%S)
```

---

## 1. Install Zsh, Git, and curl

### Debian / Ubuntu / Kali Linux (native or WSL)

```bash
sudo apt update
sudo apt install zsh git curl -y
```

Verify:

```bash
zsh --version
git --version
curl --version
```

### Set Zsh as the default login shell

```bash
chsh -s "$(command -v zsh)"
```

If prompted, enter your Linux user's password. Log out and back in, or close and reopen the terminal.

Verify:

```bash
echo "$SHELL"
ps -p $$ -o comm=
```

The current shell process should be `zsh` after you start a Zsh session. `echo "$SHELL"` may not update in an already-open session.

If `chsh` does not change the shell used by WSL, start Zsh explicitly:

```bash
zsh
```

For WSL, you can configure the default user shell in `/etc/wsl.conf` only if needed; do not change WSL settings unless `chsh` and a fresh session fail.

---

## 2. Install Kitty

### Debian / Ubuntu / Kali

```bash
sudo apt update
sudo apt install kitty -y
kitty --version
```

Launch it:

```bash
kitty
```

### WSL note

Kitty is a graphical terminal emulator. In WSL2, a graphical app generally needs WSLg or a configured external X server. WSLg is included in current Windows 11 WSL setups and some updated Windows 10 setups.

From Windows PowerShell:

```powershell
wsl --version
wsl --update
wsl --shutdown
```

Reopen the distribution and try Kitty again.

### WSLg graphics troubleshooting

Common errors that may appear during setup include:

- `Wayland: Failed to open libwayland-cursor`
- `EGL: Library not found`
- `kitty requires working OpenGL 3.1 drivers`
- Mesa/Zink warnings such as `failed to choose pdev` or `failed to create dri2 screen`
- `org.freedesktop.portal.Desktop` / `org.freedesktop.Notifications` service warnings

Install common graphics libraries:

```bash
sudo apt update
sudo apt install libwayland-cursor0 libwayland-client0 libwayland-egl1 libegl1 libgl1 libglx0 libopengl0 libegl-mesa0 libgl1-mesa-dri mesa-utils -y
```

Check display variables:

```bash
echo "DISPLAY=$DISPLAY"
echo "WAYLAND_DISPLAY=$WAYLAND_DISPLAY"
```

Check OpenGL:

```bash
glxinfo -B
```

Check dynamic libraries:

```bash
ldconfig -p | grep -E 'libEGL\.so\.1|libwayland-cursor\.so\.0'
```

If `glxinfo -B` fails, save the exact error and troubleshoot WSLg/graphics before trying more Kitty settings. Do not assume all D-Bus portal or notification warnings are fatal; focus first on whether the Kitty window actually renders. Avoid installing random GPU drivers inside WSL.

If Kitty's GLFW supports the selected backend, you can test a backend override:

```bash
kitty --override linux_display_server=wayland
```

If this makes the problem worse, run plain `kitty` again.

---

## 3. Kitty's interactive theme browser

Once Kitty opens correctly, use its built-in theme selector:

```bash
kitty +kitten themes
```

Browse themes and choose one. Use the selector's permanent-apply option if available so the theme is saved for future sessions.

Kitty's main configuration file is:

```text
~/.config/kitty/kitty.conf
```

Open it with:

```bash
nano ~/.config/kitty/kitty.conf
```

**Avoid overwriting a theme-generated configuration blindly.** Add only settings you want to customize, and back up the file first.

### Example optional Kitty settings

Add these to `~/.config/kitty/kitty.conf` if desired:

```conf
font_size 12.0
cursor_shape beam
cursor_blink_interval 0
remember_window_size yes
initial_window_width 1100
initial_window_height 650
window_padding_width 10
scrollback_lines 10000
```

Reload Kitty's config with **Ctrl+Shift+F5** in an open Kitty window.

### Fonts

Powerlevel10k icons work best with a Nerd Font or a compatible MesloLGS NF font. Kitty's font name must match an installed font. Do not assume a font package name is identical across distributions.

On Debian/Kali, you can inspect available fonts with:

```bash
fc-list | grep -i -E 'meslo|nerd|powerline' | head
```

If a font is installed on Windows but not visible in WSL, install it in the environment where Kitty's GUI renders, then restart Kitty.

---

## 4. Install Oh My Zsh

First, ensure Zsh, Git, and curl are installed (Section 1).

Install Oh My Zsh without automatically switching the shell or launching a new Zsh session:

```bash
RUNZSH=no CHSH=no sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

If `~/.zshrc` already exists, the installer may ask whether to overwrite it. Do not overwrite your customized file without a backup. Review the installer prompt carefully.

Confirm the directory exists:

```bash
ls -ld ~/.oh-my-zsh
```

---

## 5. Install Powerlevel10k

Clone Powerlevel10k into Oh My Zsh's custom themes directory:

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

If the directory already exists, do not clone over it. Check it first:

```bash
ls -ld "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

Open Zsh configuration:

```bash
nano ~/.zshrc
```

Find the `ZSH_THEME` setting and set it to:

```zsh
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Make sure Oh My Zsh is loaded using the standard line (normally already present):

```zsh
source $ZSH/oh-my-zsh.sh
```

Save the file, then start Zsh:

```bash
zsh
```

Run the configuration wizard:

```zsh
p10k configure
```

Follow the prompts to choose the prompt layout and symbols. If the wizard does not start automatically, running `p10k configure` manually is fine.

### Powerlevel10k font check

If icons appear as squares or question marks, install/configure a compatible Nerd Font (often MesloLGS NF), set it in Kitty's `kitty.conf`, and restart Kitty.

---

## 6. Install Pokémon Colorscripts with Git

The `pokemon-colorscripts` package may not be available in Kali's APT repositories. Clone the upstream project and run its installer instead. The upstream project documents this method in its [README](https://gitlab.com/phoneybadger/pokemon-colorscripts).

```bash
cd ~/Downloads
git clone https://gitlab.com/phoneybadger/pokemon-colorscripts.git
cd pokemon-colorscripts
less install.sh
sudo ./install.sh
```

Review the installer before running it with `sudo`. Once installation finishes, verify the command:

```bash
command -v pokemon-colorscripts
pokemon-colorscripts --help
pokemon-colorscripts -r
pokemon-colorscripts --list
pokemon-colorscripts --name grookey
```

If `git clone` says the destination directory already exists, inspect it first or clone into a different directory. If the command is still not found after installation, open a new terminal and check `echo "$PATH"`; the installer normally places a command in a directory already on `PATH`.

---

## 7. Install autosuggestions and syntax highlighting as Oh My Zsh plugins

To manage both features through the Oh My Zsh plugin system, clone them into its custom plugins directory:

```bash
mkdir -p "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins"
git clone https://github.com/zsh-users/zsh-autosuggestions "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
```

If either directory already exists, do not clone over it. Check it first, or update an existing clone with `git -C /path/to/plugin pull`.

Add both plugin names to the `plugins=(...)` list in `~/.zshrc`. Keep `zsh-syntax-highlighting` **last** in the list because it should be loaded after other plugins that modify the command line. The configuration example in the next section shows the correct order.

---

## 8. Configure `~/.zshrc`

Back up your current file before editing:

```bash
cp ~/.zshrc ~/.zshrc.backup.$(date +%Y%m%d-%H%M%S)
nano ~/.zshrc
```

### Important ordering rules

1. Zsh configuration must run in **Zsh**, not Bash.
2. Powerlevel10k's instant-prompt preamble is generated near the top of `.zshrc`.
3. Any command that prints output before or during initialization (including Pokémon Colorscripts) can trigger the Powerlevel10k instant-prompt warning.
4. The configuration below places `pokemon-colorscripts -r` at the very top after the comments and before `typeset`. This preserves the desired startup order, although Powerlevel10k's instant-prompt warning may return.

### Example `.zshrc` layout

Merge the following into your existing `.zshrc`; do not blindly replace the entire file, because it may contain settings created by `p10k configure`.

```zsh
# Terminal customization: Pokémon, Powerlevel10k, Oh My Zsh, and plugins

# Pokémon startup (kept at the top after comments)
# Use this only if your installed pokemon-colorscripts supports -r.
pokemon-colorscripts -r

# Suppress the Powerlevel10k instant-prompt warning (does not prevent prompt jump)
typeset -g POWERLEVEL9K_INSTANT_PROMPT=quiet

# Oh My Zsh location
export ZSH="$HOME/.oh-my-zsh"

# Powerlevel10k theme
ZSH_THEME="powerlevel10k/powerlevel10k"

# Oh My Zsh plugins; keep syntax highlighting last
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)

# Load Oh My Zsh
source "$ZSH/oh-my-zsh.sh"

# Powerlevel10k configuration generated by `p10k configure`
[[ -f ~/.p10k.zsh ]] && source ~/.p10k.zsh

# Autosuggestions and syntax highlighting are loaded through plugins=(...) above.
# Keep zsh-syntax-highlighting last in that plugin list.

# Built-in command spelling correction (asks before correcting)
setopt CORRECT
```

**Important:** The normal Oh My Zsh template has additional content and Powerlevel10k's instant-prompt block. Keep that existing content. This example shows the relevant settings and ordering; merge it carefully instead of deleting the instant-prompt preamble.

### Verify the `-r` option

If this errors:

```text
invalid option -- r
```

use the supported long option:

```zsh
pokemon-colorscripts --random
```

---

## 9. Git plugin

Oh My Zsh includes a Git plugin. Keep it along with autosuggestions and syntax highlighting in the same plugin list:

```zsh
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

If you already have other Oh My Zsh plugins, preserve them in the same list and keep `zsh-syntax-highlighting` last. For example:

```zsh
plugins=(git docker zsh-autosuggestions zsh-syntax-highlighting)
```

After saving:

```zsh
source ~/.zshrc
```

Some useful Git aliases provided by the Git plugin include:

| Alias | Command |
|---|---|
| `gst` | `git status` |
| `gaa` | `git add --all` |
| `gco` | `git checkout` |
| `gcb` | `git checkout -b` |
| `gcm` | `git commit -m` |
| `gp` | `git push` |
| `gl` | `git pull` |
| `glog` | `git log --oneline --decorate --graph` |

---

## 10. Powerlevel10k instant-prompt warning and Pokémon output

### The warning

```text
[WARNING]: Console output during zsh initialization detected.
```

This occurs because Pokémon prints output during shell initialization, while Powerlevel10k's instant prompt expects the initialization phase to avoid terminal output.

### Option A — Keep Pokémon at the top ( I recommend)

Keep these lines in this order:

```zsh
pokemon-colorscripts -r
typeset -g POWERLEVEL9K_INSTANT_PROMPT=quiet
```

This hides the warning, but the prompt can jump down after initialization. Confirm that the installed command accepts `-r`.

### Option B — Recommended for a clean prompt

For the cleanest startup, move the Pokémon command below the Powerlevel10k instant-prompt preamble and the rest of the initialization:

```zsh
pokemon-colorscripts --random
```

This may still cause the warning if it runs during initialization. The most reliable way to avoid output during initialization is to arrange for Pokémon to run after the prompt is ready, using a Zsh hook or another post-prompt method. That setup is more advanced and may need adjustment for the installed Zsh/Powerlevel10k versions.

### Option C — Disable instant prompt

In `.zshrc`, set:

```zsh
typeset -g POWERLEVEL9K_INSTANT_PROMPT=off
```

This avoids the instant-prompt warning at the cost of slower prompt initialization.

## Note: You can use these three option at the same time if using them alone does not work.

### Do not source `.zshrc` from Bash

If you see errors such as:

```text
bash: ... bad substitution
Oh My Zsh can't be loaded from: bash
bash: setopt: not a shell builtin
```

you ran `source ~/.zshrc` while Bash was active. Zsh configuration is not Bash configuration.

Start Zsh first:

```bash
zsh
```

Then reload using:

```zsh
source ~/.zshrc
```

Check your current shell:

```bash
ps -p $$ -o comm=
echo "$ZSH_VERSION"
```

If `ZSH_VERSION` is empty, you are not running Zsh.

---

## 11. Other common installation errors

### `kitty: command not found`

```bash
sudo apt update
sudo apt install kitty -y
```

### `kitty +kitten themes` opens a blank/grey window

Check whether ordinary Kitty windows render correctly. On WSL2, check WSLg and OpenGL as described in Section 2. If only the theme browser is blank, try restarting Kitty and WSLg before changing the configuration.

### `EGL: Library not found`

```bash
sudo apt install libegl1 libgl1 libglx0 libopengl0 libegl-mesa0 libgl1-mesa-dri -y
```

Then check:

```bash
glxinfo -B
```

### `Wayland: Failed to open libwayland-cursor`

```bash
sudo apt install libwayland-cursor0 libwayland-client0 libwayland-egl1 -y
```

### `pokemon-colorscripts: command not found`

```bash
command -v pokemon-colorscripts
```

If missing, clone and install it using the Git instructions in Section 6. If the install script completed but the command is still missing, check `echo "$PATH"`, open a new terminal, and review the install script's output.

### `plugin not found: zsh-autosuggestions` or `zsh-syntax-highlighting`

Check that the plugin directories exist:

```bash
ls -ld "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
ls -ld "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
```

If a directory is missing, repeat the Git clone command from Section 7. Check spelling in `plugins=(...)`, then start a fresh Zsh session.

### `ZSH_THEME="powerlevel10k/powerlevel10k"` has no effect

Check that the theme exists:

```bash
ls -ld "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

Confirm that `.zshrc` contains the correct `ZSH_THEME` setting and that Oh My Zsh is sourced after it.

### Broken `.zshrc` after edits

Restore the backup you made earlier. Replace the timestamp below with the backup you intend to restore:

```bash
cp ~/.zshrc.backup.YYYYMMDD-HHMMSS ~/.zshrc
```

If you are currently in Bash, start Zsh using:

```bash
zsh
```

---

## 12. Final verification checklist

Run these inside Zsh:

```zsh
echo "$ZSH_VERSION"
echo "$SHELL"
command -v kitty
command -v zsh
command -v pokemon-colorscripts
git --version
```

Check the Oh My Zsh plugin directories:

```zsh
ls "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins"
```

Test Pokémon:

```zsh
pokemon-colorscripts --random
```

Test Powerlevel10k:

```zsh
p10k configure
```

Test autosuggestions by typing a previously used command and accepting the grey suggestion with the **Right Arrow** key.

Test correction with a harmless misspelling of a command that you know exists; Zsh should ask whether to correct it when `setopt CORRECT` is enabled.

---

## Recommended setup summary

- **Terminal emulator:** Kitty
- **Shell:** Zsh
- **Shell framework:** Oh My Zsh
- **Prompt:** Powerlevel10k
- **Theme selection:** `kitty +kitten themes`
- **Startup art:** Pokémon Colorscripts
- **Autosuggestions:** `zsh-autosuggestions`
- **Syntax highlighting:** `zsh-syntax-highlighting`
- **Spelling correction:** Zsh `CORRECT` option
- **Git shortcuts:** Oh My Zsh `git` plugin

Keep a backup of `.zshrc` and `kitty.conf` before major edits. Native Linux and WSL2 share most Zsh configuration steps, but Kitty graphics and font availability may differ because WSL relies on WSLg or an external display server.
