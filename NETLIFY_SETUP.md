# Deploy Flipper to Netlify

This guide publishes Flipper from its GitHub repository and connects it to a domain you already own: `milagro-nexus.com`.

## Recommended URL

Use:

```text
https://flipper.milagro-nexus.com
```

A subdomain is required for this deployment because it leaves the existing websites and services on `milagro-nexus.com` and `www.milagro-nexus.com` unchanged.

> **Important:** Create Flipper as a new Netlify site. Do not import it into, replace, or reconfigure the existing Milagro Nexus site, which publishes `frontend/dist`.

## Before You Begin

You need:

- Access to the GitHub repository at `https://github.com/paoshea/Flipper`
- A Netlify account with permission to create sites in the `paoshea` team
- Access to the Netlify-managed DNS zone for `milagro-nexus.com`

The repository is public, the Netlify GitHub App is already installed for the `paoshea` account, and the domain already uses Netlify DNS. You can confirm the active nameservers with:

```bash
dig NS milagro-nexus.com +short
```

## 1. Confirm the Repository Is Ready

The repository already contains `index.html` at its root. No rename or build step is required. Confirm the file is present and the working tree is current:

```bash
git pull --ff-only origin main
test -f index.html && echo "Flipper entry page is ready"
```

Open it locally once to confirm the app still works:

```bash
open index.html
```

Netlify serves `index.html` automatically at `/`.

## 2. Import the GitHub Repository into Netlify

1. Sign in at [Netlify](https://app.netlify.com/).
2. Select **Add new project**.
3. Choose **Import an existing project** or **Import from Git**.
4. Select **GitHub** as the Git provider.
5. Select the existing Netlify GitHub App connection. The repository is public and should already be visible.
6. If the repository is not listed, configure the GitHub App and grant access to `paoshea/Flipper`.
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
6. Netlify will detect the existing `milagro-nexus.com` DNS zone in the same team and offer to create the required record.
7. Accept Netlify's generated DNS record for the `flipper` host.
8. Do not alter the apex, `www`, `app`, mail, verification, or other DNS records.

The `flipper` host is currently unused, so attaching it does not replace an existing service.

## 4. Verify the Netlify DNS Record

No external DNS-provider work is required. Under the Netlify DNS zone for `milagro-nexus.com`, verify that Netlify created the record it proposed for the new site.

- Confirm `flipper.milagro-nexus.com` is assigned only to the new Flipper site.
- Leave `milagro-nexus.com`, `www.milagro-nexus.com`, and `app.milagro-nexus.com` unchanged.
- Do not edit the existing Milagro Nexus build command or `frontend/dist` publish directory.

DNS changes often appear within minutes but can take several hours. Netlify advises allowing up to 24 hours for broad propagation.

## 5. Verify DNS and HTTPS

Check the DNS response from a terminal:

```bash
dig flipper.milagro-nexus.com +short
```

The result should contain a Netlify-managed answer. Then verify HTTPS:

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

## 7. Enable Anonymous Access

The `paoshea` team currently enforces a team-wide Netlify SSO requirement. This rule takes precedence over Flipper's site-level setting, so anonymous requests return HTTP 401 even though Flipper has no site password and its own SSO flag is off.

A team Owner or admin must make this change in the Netlify dashboard:

1. Confirm that every other site that must remain private has its own site-level SSO protection enabled.
2. Open **Team settings** for the `paoshea` team.
3. Go to **Access & security > Site protection**. Netlify may label this **Visitor access** or **Netlify SSO**.
4. Turn off the team-wide SSO or login requirement.
5. Do not change Flipper's site-level SSO flag; it should remain off.

On the current team configuration, all eleven other sites have site-level SSO enabled, while Flipper is the only site with it disabled. Removing the blanket team rule should therefore make only Flipper public. Verify that assumption immediately after the change rather than relying on configuration alone.

Confirm anonymous Flipper access:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://flipper.milagro-nexus.com
curl -s https://flipper.milagro-nexus.com | grep -o '<title>.*</title>'
```

Expected output:

```text
200
<title>Flipper — probability-logged coin flips</title>
```

Then spot-check that the existing protected site still returns HTTP 401:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://milagro-nexus.com
```

If Flipper still returns 401 or its response mentions `edge-access`, the team-level gate remains active. If an existing private site becomes public, restore the team-wide rule immediately and verify its site-level protection before trying again.

This setting cannot be changed through repository files or `netlify.toml`. A site-scoped automation token also cannot modify it.

## 8. Publish Future Updates

Because the site is connected to GitHub, every push to `main` triggers a production deployment:

```bash
git add .
git commit -m "Describe the Flipper update"
git push origin main
```

Monitor the deployment under **Deploys** in Netlify. Each pull request can also receive a temporary Deploy Preview if that feature is enabled.

## Do Not Change the Apex Site

The existing Milagro Nexus application must remain available at:

```text
https://milagro-nexus.com
https://www.milagro-nexus.com
```

Do not attach either hostname to the Flipper site. Do not modify `app.milagro-nexus.com`, the Milagro Nexus build command, or its `frontend/dist` publish directory. Flipper is an independent sibling site using only `flipper.milagro-nexus.com`.

## Troubleshooting

### The domain shows a 404

Confirm that the deployed repository contains `index.html` at its root and that the Netlify publish directory is `.`.

### Netlify cannot see the repository

The repository is public. If it is absent from the import list, review the installed Netlify GitHub App and grant it access to `paoshea/Flipper`, then retry.

### Site creation returns HTTP 401

A site-scoped agent token cannot create another Netlify site or modify the shared DNS zone. Sign in to the Netlify UI as a `paoshea` team member with site-creation and DNS permissions. Alternatively, use a Netlify personal access token belonging to a team Owner. Do not reuse or reconfigure the existing Milagro Nexus site as a workaround.

### The deployed site returns HTTP 401

If the custom domain, generated `netlify.app` URL, branch URL, and deploy permalink all return Netlify's `edge-access` challenge, the team-wide visitor protection rule is still active. Follow **Enable Anonymous Access** above. Changing DNS, redeploying, or adding headers will not remove this gate.

### DNS verification remains pending

- Open the `milagro-nexus.com` zone in Netlify DNS and confirm the generated `flipper` record exists.
- Confirm the custom domain is assigned to the Flipper site and no other site.
- Run `dig flipper.milagro-nexus.com +short`.
- Allow up to 24 hours before escalating.

### HTTPS provisioning fails

- Confirm that DNS points only to the expected Netlify target.
- Check for restrictive `CAA` records that do not allow Let's Encrypt.
- Review **Domain management > HTTPS** for the exact error.

### Auto flips pause in the background

Browsers throttle timers in inactive tabs and do not run the page after it is closed. Flipper catches up overdue tosses when reopened, up to its configured limit of 500.

### Toss history appears empty on the custom domain

This is expected when changing from the local file or temporary Netlify address. Each URL has separate browser storage. Export the old history first; importing backups is listed as a future enhancement and is not currently supported.

## Official References

- [Deploy an existing project](https://docs.netlify.com/start/quickstarts/deploy-from-repository/)
- [Assign a domain to a Netlify site](https://docs.netlify.com/manage/domains/manage-domains/assign-a-domain-to-your-site-app/)
- [Netlify DNS](https://docs.netlify.com/manage/domains/why-netlify-dns/)
- [Netlify HTTPS certificates](https://docs.netlify.com/manage/domains/secure-domains-with-https/https-ssl/)
