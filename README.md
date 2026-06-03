# comet-wine

Run [Perplexity Comet](https://www.perplexity.ai/comet) (Chromium 145-based AI browser) on Linux under Wine 11.

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

## File Attached - Analysis
**URL**: https://www.virustotal.com/gui/file/24b76e0cb7af695f33f5caeba2fd5e34cdeb135093e05749afffee4082b6b0e3/details

**File Name**: comet_installer_latest.exe

**MD5**: de44b07c8cdc84f03df851e22226e5fa
 
**SHA-1**: 3ed1cfb115319b5861b8d60b8c033e21e5318b42
 
**SHA-256**: 24b76e0cb7af695f33f5caeba2fd5e34cdeb135093e05749afffee4082b6b0e3

**Verdict**: 0/69

## Quick start

```bash
# Install Wine 11
sudo apt install --install-recommends winehq-stable

# Download Comet from perplexity.ai/comet and install it in ~/.wine

# Clone and run
git clone github.com/AbdurRahman-cybersec/Perplexity-Comet-in-Linux-Wine
cd comet-wine
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
