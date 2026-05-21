# syncsheet

OAuth redirect handler for **myM-Pesa Ledger** — a lightweight GitHub Pages page that receives the Smartsheet OAuth callback and forwards the auth code back to the app via deep link.

## How it works

1. User taps **Log in with Smartsheet** in the app
2. A Chrome Custom Tab opens Smartsheet's OAuth consent page
3. After the user approves, Smartsheet redirects to this page:
   ```
   https://neuralquantam.github.io/mpesa-auth/?code=XXX&state=YYY
   ```
4. The page's JavaScript immediately redirects the browser to:
   ```
   mpesatracker://auth?code=XXX&state=YYY
   ```
5. Android intercepts the `mpesatracker://` deep link, closes the browser tab, and returns the code to the app
6. The app exchanges the code for an access token and begins setup

## Files

| File | Purpose |
|------|---------|
| `index.html` | The redirect page — reads `?code=` from the URL and forwards to the app deep link |

## Setup

### 1. Create the repo

Create a **public** GitHub repository named `mpesa-auth` under the `NeuralQuantam` account.

### 2. Add index.html

Create `index.html` in the repo root:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Signing in to M-Pesa Ledger…</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <p>Redirecting back to the app…</p>
  <script>
    const p = new URLSearchParams(window.location.search);
    const code = p.get('code');
    if (code) {
      window.location.href = 'mpesatracker://auth?' + window.location.search.substring(1);
    } else {
      document.body.innerHTML = '<p style="color:red;font-family:sans-serif">Authentication error: '
        + (p.get('error') || 'No code returned')
        + '<br><br>Please close this tab and try again.</p>';
    }
  </script>
</body>
</html>
```

### 3. Enable GitHub Pages

Go to **Settings → Pages → Source: Deploy from branch → main / root** and save. The page will be live at `https://neuralquantam.github.io/mpesa-auth/` within a couple of minutes.

### 4. Register the redirect URI in Smartsheet

In the [Smartsheet Developer Portal](https://developers.smartsheet.com/), set the **App Redirect URL** for your app to:

```
https://neuralquantam.github.io/mpesa-auth/
```

> The trailing slash is required — Smartsheet performs an exact match.

## Registered redirect URI

```
https://neuralquantam.github.io/mpesa-auth/
```

## App deep link scheme

```
mpesatracker://auth
```

Defined in `app.json` → `"scheme": "mpesatracker"` and registered in `AndroidManifest.xml` via an intent filter for the `mpesatracker` scheme.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Page shows "Authentication error: No code returned" | Smartsheet redirect URI mismatch | Confirm the URI in the developer portal matches exactly, including the trailing slash |
| Page loads but app doesn't open | `mpesatracker://` intent filter missing | Ensure `AndroidManifest.xml` has `<data android:scheme="mpesatracker"/>` in an intent filter on `MainActivity` |
| "invalid_client" on the Smartsheet login page | Wrong client ID | Check `SS_CLIENT_ID` in `src/utils/smartsheet.ts` |
| App receives `state mismatch` error | State parameter not passed through | Confirm the `index.html` uses `window.location.search.substring(1)` to forward all params unchanged |
