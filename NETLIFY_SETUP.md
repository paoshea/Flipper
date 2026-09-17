# Deploy Flipper to Netlify

This guide publishes Flipper from its GitHub repository and connects it to a domain you already own: `milagro-nexus.com`.

## Recommended URL

Use:

```text
https://flipper.milagro-nexus.com
```

A subdomain is recommended because it does not replace any website or service already using `milagro-nexus.com` or `www.milagro-nexus.com`. Use the apex-domain instructions later in this guide only if Flipper should replace the main website at `https://milagro-nexus.com`.

## Before You Begin

You need:

- Access to the GitHub repository at `https://github.com/paoshea/Flipper`
- A Netlify account that can connect to that private GitHub repository
- Access to the DNS settings for `milagro-nexus.com`
- The name of the company currently providing DNS for the domain

Your registrar and DNS provider may be different companies. To find the active DNS provider, inspect the domain's nameservers in the registrar dashboard or run:

```bash
dig NS milagro-nexus.com +short
```

## 1. Make Flipper Load at the Site Root

Netlify serves `index.html` automatically at `/`. The repository currently uses `CoinFlip.html`, so rename it before deployment:

```bash
git mv CoinFlip.html index.html
git add README.md NETLIFY_SETUP.md
git commit -m "Prepare Flipper for Netlify deployment"
git push origin main
```

After the rename, open `index.html` locally once to confirm the app still works:

```bash
open index.html
```

If you keep the name `CoinFlip.html`, the app will be available at `/CoinFlip.html`, but the custom domain's home page may return a 404.

## 2. Import the GitHub Repository into Netlify

1. Sign in at [Netlify](https://app.netlify.com/).
2. Select **Add new project**.
3. Choose **Import an existing project** or **Import from Git**.
4. Select **GitHub** as the Git provider.
5. Authorize the Netlify GitHub app if prompted.
6. Grant Netlify access to the private `paoshea/Flipper` repository.
7. Select the **Flipper** repository.
8. Configure the deployment:

   | Setting | Value |
   | --- | --- |
   | Production branch | `main` |
   | Base directory | Leave blank |
   | Build command | Leave blank |
   | Publish directory | `.` |

9. Select **Deploy**.
10. Wait for the production deploy to finish.

Netlify will assign a temporary address similar to:

```text
https://your-site-name.netlify.app
```

Open that address and verify all of the following before changing DNS:

- The page title says **Flipper**.
- The dolphin icon appears.
- Manual tosses work.
- Auto flips work when enabled.
- A page reload preserves toss history.
- **Export JSON** downloads a backup.

Browser `localStorage` is scoped to each origin. History created from the local `file:` page will not automatically appear on the new Netlify URL, and history on the temporary `netlify.app` URL will not transfer to the custom domain.

## 3. Add the Recommended Custom Domain

1. Open the Flipper site in Netlify.
2. Select **Domain management** in the site sidebar.
3. Select **Add a domain**, then **Add a domain you already own**.
4. Enter:

   ```text
   flipper.milagro-nexus.com
   ```

5. Select **Verify**, then confirm **Add domain**.
6. Choose **External DNS Provider** unless you intentionally want to move management of the entire domain to Netlify DNS.
7. In Netlify, select **Pending DNS verification** beside the new domain.
8. Note the customized CNAME target shown by Netlify. It normally matches the temporary site address, such as `your-site-name.netlify.app`.

Do not enter `https://` or a path in a DNS record.

## 4. Configure DNS for the Subdomain

At the active DNS provider for `milagro-nexus.com`, add this record using the exact target shown by Netlify:

| Type | Host/Name | Target/Value | TTL |
| --- | --- | --- | --- |
| `CNAME` | `flipper` | `your-site-name.netlify.app` | Automatic or 300 seconds |

Important checks:

- Replace `your-site-name.netlify.app` with the actual Netlify hostname.
- Some DNS dashboards expect the full host `flipper.milagro-nexus.com`; most expect only `flipper`.
- Remove any conflicting `A`, `AAAA`, or `CNAME` record for the `flipper` host.
- Do not modify unrelated mail, verification, apex, or `www` records.
- If Cloudflare manages DNS, initially set the record to **DNS only** rather than proxied until Netlify verifies the domain and provisions HTTPS.

DNS changes often appear within minutes but can take several hours. Netlify advises allowing up to 24 hours for broad propagation.

## 5. Verify DNS and HTTPS

Check the CNAME from a terminal:

```bash
dig CNAME flipper.milagro-nexus.com +short
```

The result should end with your Netlify hostname. Then verify HTTPS:

```bash
curl -I https://flipper.milagro-nexus.com
```

In Netlify:

1. Return to **Domain management**.
2. Confirm that the domain no longer shows **Pending DNS verification**.
3. Open **Domain management > HTTPS**.
4. Confirm that the Netlify-managed TLS certificate is active.
5. Open `https://flipper.milagro-nexus.com` in a private browser window.

Netlify automatically provisions and renews a Let's Encrypt certificate after the DNS records resolve correctly. Do not enable HSTS preload during initial setup; it is difficult to reverse and affects subdomains.

## 6. Set the Primary Domain

Under **Domain management > Production domains**, confirm that:

```text
flipper.milagro-nexus.com
```

is the primary domain. Keep the generated `netlify.app` address as a secondary domain so Netlify can redirect it to the primary URL.

## 7. Publish Future Updates

Because the site is connected to GitHub, every push to `main` triggers a production deployment:

```bash
git add .
git commit -m "Describe the Flipper update"
git push origin main
```

Monitor the deployment under **Deploys** in Netlify. Each pull request can also receive a temporary Deploy Preview if that feature is enabled.

## Optional: Use the Apex Domain

Use this option only if Flipper should replace the current site at both:

```text
https://milagro-nexus.com
https://www.milagro-nexus.com
```

First add `milagro-nexus.com` under **Domain management** in Netlify. Netlify normally adds `www.milagro-nexus.com` as well.

With an external DNS provider, configure the exact values displayed under **Pending DNS verification**. Standard Netlify values are:

| Type | Host/Name | Target/Value |
| --- | --- | --- |
| `ALIAS`, `ANAME`, or flattened `CNAME` | `@` | `apex-loadbalancer.netlify.com` |
| `CNAME` | `www` | `your-site-name.netlify.app` |

If the provider does not support an apex `ALIAS`, `ANAME`, or flattened CNAME, Netlify documents this fallback:

| Type | Host/Name | Target/Value |
| --- | --- | --- |
| `A` | `@` | `75.2.60.5` |

Always prefer the customized records shown in the Netlify dashboard over generic values in this guide. Changing apex or `www` records can take an existing website offline, so record the current DNS values before replacing them.

## Optional: Move DNS Management to Netlify

You can choose Netlify DNS instead of retaining the current DNS provider. Netlify will provide nameservers that must replace the existing nameservers at the domain registrar.

Before changing nameservers:

1. Copy every existing DNS record, especially `MX`, `TXT`, email authentication, verification, and service records.
2. Recreate those records in Netlify DNS.
3. Confirm that email and other subdomains will remain operational.
4. Replace the nameservers at the registrar only after the Netlify zone is complete.

For a single `flipper` subdomain, retaining the current DNS provider and adding one CNAME is usually simpler and lower risk.

## Troubleshooting

### The domain shows a 404

Confirm that the deployed repository contains `index.html` at its root and that the Netlify publish directory is `.`.

### Netlify cannot see the private repository

In GitHub, review the installed Netlify GitHub app and grant it access to `paoshea/Flipper`, then retry the import.

### DNS verification remains pending

- Open **Pending DNS verification** and compare every value with the DNS provider.
- Remove conflicting records for the same host.
- Confirm that the CNAME target does not contain `https://`.
- Run `dig CNAME flipper.milagro-nexus.com +short`.
- Allow up to 24 hours before escalating.

### HTTPS provisioning fails

- Confirm that DNS points only to the expected Netlify target.
- Check for restrictive `CAA` records that do not allow Let's Encrypt.
- Review **Domain management > HTTPS** for the exact error.
- If using Cloudflare, keep the CNAME set to **DNS only** during verification.

### Auto flips pause in the background

Browsers throttle timers in inactive tabs and do not run the page after it is closed. Flipper catches up overdue tosses when reopened, up to its configured limit of 500.

### Toss history appears empty on the custom domain

This is expected when changing from the local file or temporary Netlify address. Each URL has separate browser storage. Export the old history first; importing backups is listed as a future enhancement and is not currently supported.

## Official References

- [Deploy an existing project](https://docs.netlify.com/start/quickstarts/deploy-from-repository/)
- [Assign a domain to a Netlify site](https://docs.netlify.com/manage/domains/manage-domains/assign-a-domain-to-your-site-app/)
- [Configure external DNS](https://docs.netlify.com/manage/domains/configure-domains/configure-external-dns/)
- [Netlify HTTPS certificates](https://docs.netlify.com/manage/domains/secure-domains-with-https/https-ssl/)
