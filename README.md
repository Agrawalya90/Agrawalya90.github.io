# Portfolio

A static one-page site (`index.html` + `favicon.svg`, no build step). Open `index.html` in a browser to preview it.

| Address | What | Where it runs |
|---|---|---|
| `yourdomain.com`, `www.yourdomain.com` | This portfolio | Cloudflare Pages (free) |
| `realty.yourdomain.com` (+ `*.realty.yourdomain.com` for brokerages) | Realty AI Agent + CRM | Existing Ubuntu server, Caddy → port 8100 |
| `billing.yourdomain.com` | YTC Billing demo | Same server, Caddy → port 4310 |

## 1. Fill in your details

Replace these placeholders in `index.html` (and the `YN` initials in `favicon.svg`):

`Your Name` · `yourdomain.com` · `you@yourdomain.com` · `your-github` · `your-linkedin`

## 2. Buy the domain (Cloudflare Registrar)

dash.cloudflare.com → **Domain Registration → Register Domains**. Cloudflare charges the registry price with no markup, and the domain's DNS is managed in the same dashboard.

## 3. DNS records (Cloudflare → your domain → DNS)

`SERVER_IP` is the public IP of the server that runs the CRM. Set the server records to **DNS only (grey cloud)**, so that Caddy can get the https certificates itself.

| Type | Name | Content | Proxy |
|---|---|---|---|
| A | `realty` | `SERVER_IP` | DNS only |
| A | `*.realty` | `SERVER_IP` | DNS only |
| A | `billing` | `SERVER_IP` | DNS only |

The root and `www` records are created by Pages in step 4.

## 4. Publish the portfolio (Cloudflare Pages)

```bash
cd portfolio
npx wrangler login
npx wrangler pages deploy . --project-name portfolio
```

Then go to Workers & Pages → `portfolio` → **Custom domains** and add `yourdomain.com` and `www.yourdomain.com`. To update the site later, run the deploy command again. You can also connect a GitHub repo, so that every push deploys.

## 5. Realty CRM on its subdomain (on the server)

```bash
cd /opt/realty-crm && git pull && bash deploy/setup.sh   # installs the updated Caddyfile (adds /etc/caddy/sites import)
```

In `/opt/realty-crm/.env`:

```
PLATFORM_DOMAIN=realty.yourdomain.com
PLATFORM_SCHEME=https
PUBLIC_BASE_URL=https://realty.yourdomain.com
```

Then run `sudo systemctl restart realty-crm`. Brokerages get `<slug>.realty.yourdomain.com` automatically. If you ship the Android app, change `defaultServer` in `android/gradle.properties` to the new address.

## 6. YTC Billing on its subdomain (on the server)

From your laptop:

```bash
rsync -az --exclude node_modules --exclude data --exclude android --exclude .env \
  ~/Bugs_Villa/ytc-billing/ ubuntu@SERVER_IP:/opt/ytc-billing/
```

On the server: `sudo install -d -o ubuntu /opt/ytc-billing` (first time, before the rsync), then:

```bash
cd /opt/ytc-billing && bash deploy/setup.sh billing.yourdomain.com
```

The demo has no login, so anyone can change its data. The `ytc-billing-reset.timer` restores the sample data every night at 03:30 server time.

## 7. Check

```bash
curl -I https://yourdomain.com
curl -I https://realty.yourdomain.com
curl -I https://billing.yourdomain.com
```

The first request to each server subdomain takes a few seconds while Caddy gets its certificate.
