# 🍷 comet-wine

<p align="center">
  <img src="https://img.shields.io/badge/platform-linux%20%7C%20wine%2011-8B0000?style=flat-square&logo=linux&logoColor=white&labelColor=333" alt="Platform">
  <img src="https://img.shields.io/badge/status-works%20on%20my%20machine-brightgreen?style=flat-square&labelColor=333" alt="Status">
  <img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square&labelColor=333" alt="License">
  <img src="https://img.shields.io/badge/debugging%20time-3%20hours-red?style=flat-square&labelColor=333" alt="Debug time">
  <img src="https://img.shields.io/badge/stack%20overflows%20hunted-1-yellow?style=flat-square&labelColor=333" alt="Stack overflows hunted">
  <img src="https://img.shields.io/badge/red%20herrings-DComposition-ff69b4?style=flat-square&labelColor=333" alt="Red herrings">
  <img src="https://img.shields.io/badge/PRs-welcome-cyan?style=flat-square&labelColor=333" alt="PRs welcome">
</p>

<p align="center"><strong>Run <a href="https://www.perplexity.ai/comet">Perplexity Comet</a> — Chromium 145 AI browser — on Linux under Wine 11.</strong></p>

## The problem

Comet crashes 10–12 seconds after launch on Wine. The logs show `DCompositionCreateDevice3: Not implemented` — but that's a red herring. The real killer is Wine's debug infrastructure leaking stack space on every `fixme:` log call, combined with an onboarding intro video that Wine can't decode.

## The fix

Two ingredients:

| Issue | Fix |
|---|---|
| Stack overflow from ~5,000 `fixme:` calls | `WINEDEBUG=-all` prevents Wine from logging, so no stack leak |
| Onboarding video kills browser process | Pre-seed `Preferences` with onboarding marked complete |

## Requirements

- Wine 11 stable (WineHQ)
- `cabextract` (for winetricks codec install, optional)

## Quick start

```bash
# Install Wine 11
sudo apt install --install-recommends winehq-stable

# Download Comet from perplexity.ai/comet and install it in ~/.wine

# Clone and run
git clone https://github.com/AbdurRahman-cybersec/Perplexity-Comet-in-Linux-Wine.git
cd Perplexity-Comet-in-Linux-Wine
./comet
```

Or just copy these three lines into your shell:

```bash
export WINEDEBUG=-all
mkdir -p "$HOME/.wine/drive_c/users/$USER/AppData/Local/Perplexity/Comet/User Data/Default"
echo '{"browser":{"has_seen_welcome_page":true,"onboarding":{"completed":true,"skipped":true}},"perplexity_intro":{"shown":true,"completed":true},"session":{"restore_on_startup":4,"startup_urls":["https://www.perplexity.ai"]}}' > "$HOME/.wine/drive_c/users/$USER/AppData/Local/Perplexity/Comet/User Data/Default/Preferences"
wine "C:\\users\\$USER\\AppData\\Local\\Perplexity\\Comet\\Application\\comet.exe" --no-sandbox --disable-gpu
```

## What works

- Full browser chrome (tabs, address bar, settings)
- Browsing, searching, Perplexity AI chat
- Extensions
- Multiple tabs and windows

## What doesn't

- GPU acceleration (`--disable-gpu` required — Wine has no `DCompositionCreateDevice3`)
- The onboarding intro video (skipped via prefs)
- Chromium sandbox (`--no-sandbox` required — no Windows kernel)

## How it was found

Full diagnosis and triage in [DIAGNOSIS.md](DIAGNOSIS.md).

## License

MIT
