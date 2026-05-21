# syncsheet

OAuth redirect handler for **myM-Pesa Ledger**.

## What it does

Smartsheet's OAuth flow requires an HTTPS redirect URI — it won't redirect directly to a mobile app's custom scheme (`mpesatracker://`). This page acts as a lightweight bridge:

1. Smartsheet redirects here after the user approves access:
   ```
   https://neuralquantam.github.io/syncsheet/?code=XXX&state=YYY
   ```
2. The page immediately forwards to the app's deep link:
   ```
   mpesatracker://auth?code=XXX&state=YYY
   ```
3. Android intercepts the deep link, closes the browser tab, and hands the auth code back to the app to complete sign-in.

## Registered redirect URI

```
https://neuralquantam.github.io/syncsheet/
```

## App deep link scheme

```
mpesatracker://auth
```
