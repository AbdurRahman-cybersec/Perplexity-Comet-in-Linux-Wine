# Diagnosis: Comet on Wine

## Timeline

### 1. Vanilla Wine 11 — no window, 600MB RAM

Comet installed via Wine 11. Browser loaded to 600MB RAM but never painted a window.
`DCompositionCreateDevice3: Not implemented` in logs.

**Hypothesis:** DirectComposition missing → no compositing → no pixels.

### 2. Proton-GE / GE-Proton — stack overflow

Switched to Proton Experimental (Wine 11 + Proton patches) and GE-Proton10-34
(Wine 10 staging). Browser launched, renderers loaded, but main process crashed
after 10–12 seconds with:

```
uisettings2_get_TextScaleFactor stack overflow 1920 bytes
```

After 5,283 `fixme:uisettings2_get_TextScaleFactor` calls from thread `00d0`.

**Finding:** Wine's debug infrastructure (`__wine_dbg_output`) leaks a few bytes
of stack per call. Chromium calls this API thousands of times per second. Stack
guard page is hit after ~5,000 calls → SIGSEGV.

### 3. WINEDEBUG=-all — no more stack overflow

Suppressing all Wine debug output prevents the leak path entirely.
Main process now survives indefinitely.

But the full browser still exits with code 5 at ~12 seconds.

### 4. Exit code 5 — onboarding video

Full browser mode always loads `chrome://perplexity-onboarding/`. This page
tries to play an intro video. Wine has no Media Foundation codecs, so the
`<video>` element fails. The JavaScript error propagates as an unhandled
promise rejection, which Comet treats as fatal.

`--app` mode worked because it skips onboarding entirely.

### 5. Pre-seeded Preferences — fixed

Creating `Default/Preferences` with `"onboarding": {"completed": true, "skipped": true}`
and `"perplexity_intro": {"shown": true, "completed": true}` before launch tells
Comet to skip the onboarding page. Browser opens to a normal tab.

### Working configuration

```bash
WINEDEBUG=-all
wine comet.exe --no-sandbox --disable-gpu
```

Plus the pre-seeded Preferences file.

### Why `--disable-gpu` is needed

Wine 11 does not implement `DCompositionCreateDevice3` (DirectComposition).
Without `--disable-gpu`, Chromium tries to use it for window compositing and
falls through to a software path. Passing `--disable-gpu` skips the attempt
entirely. The DComposition error still appears in logs but is non-fatal.

### Remaining rough edges

- `--no-sandbox` warning banner in the browser (harmless — Chromium's sandbox
  requires a real Windows kernel, not Wine's)
- No hardware video decode (Media Foundation codec chain incomplete)
- NahimicOSD crash reporter logging benign errors
