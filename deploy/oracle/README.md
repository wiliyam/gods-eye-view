# Oracle deployment

Target: `gv.arkbytetech.com` → A record `192.9.166.69`.
Host: Ubuntu 24.04 ARM64, Oracle Sydney, `always-free-a1-2c12g`.
Fork: https://github.com/wiliyam/gods-eye-view

The application lives in `/opt/gods-eye-view/app`. Its isolated Node 24.21.0
runtime lives in `/opt/gods-eye-view/runtime/node-v24.21.0-linux-arm64`.
The `gods-eye-view` systemd service runs as `gev`, starts after reboot, and
serves the built application plus upstream provider middleware through Vite
preview on `127.0.0.1:4173`. Nginx supplies HTTPS and HTTP Basic authentication.
This remains the upstream app's preview runtime, not a production application
server. Provider Settings is blocked remotely. No provider keys were installed.

## Local deployment records

The local checkout's ignored `.deployment/credentials.txt` contains the login.
It is owner-readable only. `.deployment/` also contains verification output
and the Oracle firewall snapshot. These files must never be committed.

The app is served only at `gv.arkbytetech.com`. The temporary sslip.io virtual
host and certificate were removed after the custom domain became accessible.
The HTTPS virtual host rejects other hostnames and direct-IP requests.

## Custom domain

The A record named `gv` points to `192.9.166.69`. HTTPS was activated on
2026-09-25; the initial certificate expires on 2026-12-24 and renews automatically.
The following commands document activation on the Oracle server:

```sh
sudo certbot certonly --webroot -w /var/www/letsencrypt \
  -d gv.arkbytetech.com --non-interactive --cert-name gv.arkbytetech.com
sudo install -m 644 /home/ubuntu/gods-eye-view-deploy/nginx.conf \
  /etc/nginx/sites-available/gv.arkbytetech.com
sudo nginx -t && sudo systemctl reload nginx
```

Certbot's existing renewal timer and Nginx reload hook renew certificates.
Leave the existing TradingAgents virtual host untouched.

## Operations

```sh
ssh ubuntu@192.9.166.69
sudo systemctl status gods-eye-view
sudo journalctl -u gods-eye-view -n 100 --no-pager
sudo systemctl restart gods-eye-view
```

The application tree is root-owned; only `.gev-cache`, `.gev-logs`,
`node_modules/.vite-temp`, and `/var/lib/gods-eye-view` are writable by the
service. For an update, pull the reviewed revision as root, build with the
isolated Node runtime and `PUPPETEER_SKIP_DOWNLOAD=true`, then restart the
service. Keep the cache directory ownership as `gev:gev`.

Optional provider keys belong in the app's ignored, access-restricted `.env`.
Google Maps and Cesium client keys require a rebuild. Never commit keys or
reuse unrelated project credentials. Keyless imagery and available public
feeds work without them; voice, AISStream vessels, and FIRMS need keys.

## Verification

- Node setup doctor and production build passed on Oracle.
- Nginx configuration validation passed.
- Service enabled at boot; backend returned HTTP 200.
- Custom-domain HTTPS returned 401 without login and 200 with login, with a
  valid certificate. A resolver override was used during DNS propagation.
- Satellite data returned HTTP 200 with 60 lines of TLE data.
- Remote provider settings returned HTTP 404 as configured.
- Existing TradingAgents website still returned its expected login redirect.
- ADS-B military feed returned 502: direct upstream access also timed out
  intermittently from Oracle. This feed's availability is an upstream limitation.
- Interactive browser verification was unavailable in the setup session.
