# Flipper

Flipper is a browser-based coin-toss tracker that logs every flip, measures streaks, and compares observed results with fair-coin probabilities. It runs as a single HTML file with no build process, framework, backend, or account required.

## Features

- Manual coin tosses with animated heads and tails results
- Automatic tossing by seconds, minutes, hour, or day
- Persisted auto-toss settings and countdown to the next flip
- Catch-up for up to 500 missed scheduled tosses after reopening the page
- Lifetime heads, tails, percentages, streaks, run counts, and fairness z-score
- Exact two-sided binomial fairness test with an explained p-value
- A 95% Wilson score interval and optional Bayesian credible interval for heads
- Latest milestone details for every achieved streak length from 4 through the record
- A gold outline identifying the highest retained streak
- Milestone timeline and lifetime frequency summaries for qualifying runs
- A lifetime podium showing the three longest individual streaks
- A 52-week calendar heatmap with daily toss counts and longest streaks
- Optional education mode explaining formulas and common probability misconceptions
- Exact probability that a run of each length has appeared by the current toss count
- Expected-versus-observed maximal run distribution
- Recent-history strip and detailed history list
- JSON export and full local reset
- Responsive layout for desktop and mobile
- Local browser storage, with no data sent to a server

## Quick Start

No installation is required.

1. Clone the repository:

   ```bash
   git clone https://github.com/paoshea/Flipper.git
   cd Flipper
   ```

2. Open `index.html` in a modern browser.

On macOS, you can also run:

```bash
open index.html
```

## Using Flipper

### Manual flips

Select **Toss coin**, or press `Space` or `Enter` while focus is not on an interactive control.

### Automatic flips

1. Select a frequency: seconds, minutes, hourly, or daily.
2. Enter the interval when using seconds or minutes.
3. Check **On - Auto flips**.
4. Keep the page open for live scheduled flips.

When the page is reopened after a scheduled time, Flipper backfills missed flips up to a limit of 500. After reaching that limit, scheduling resumes from the current time.

### Exporting data

Select **Export JSON** to download the settings, toss history, milestone history, total toss count, and export timestamp. The generated filename uses the format:

```text
flipper-YYYY-MM-DD.json
```

### Resetting data

Select **Reset** and confirm the prompt to permanently remove toss history, milestone history, and settings from the current browser profile.

## Statistics and Probability

Flipper assumes independent tosses from a fair coin, with heads and tails each having probability $1/2$.

### Streak probability

The probability that a specified sequence ending at the current toss consists of either all heads or all tails for length $k$ is:

$$
P(\text{matching run of length } k) = 2^{1-k}
$$

The average wait from a fresh start until either side reaches length $k$ is:

$$
E[T_k] = 2^k - 1
$$

### Chance by now

For every run length from 3 through 20, Flipper calculates the exact probability that at least one run of that length has appeared somewhere within $N$ tosses. It uses a recurrence for the complementary probability of no qualifying run rather than an independence approximation.

### Run-length distribution

A run is a maximal block of identical results. For $k < N$, the expected number of maximal runs of exactly length $k$ is:

$$
E[R_k] = \frac{N-k+3}{2^{k+1}}
$$

Boundary cases and the cumulative `6+` bucket are handled separately. The fairness statistic is the standardized heads-versus-tails difference:

$$
z = \frac{H-T}{\sqrt{N}}
$$

A z-score is descriptive evidence, not proof that the coin or random generator is fair or unfair.

### Exact fairness test

Flipper also reports an exact two-sided binomial p-value under the null hypothesis that heads and tails each have probability $1/2$. It sums the probabilities of fair-coin outcomes with an imbalance at least as large as the one observed.

A small p-value means the observed imbalance would be unusual if the coin were fair. It is not the probability that the coin is fair, and it does not by itself prove bias. The calculation is exact for the binomial model rather than a large-sample normal approximation.

### Heads proportion intervals

The Heads card always shows a 95% Wilson score interval for the underlying heads proportion. Wilson intervals remain well behaved for small samples and proportions near zero or one, unlike the basic normal interval.

The optional Bayesian interval can be enabled under **Analysis**. It is a 95% equal-tailed credible interval from a $\mathrm{Beta}(H+1,T+1)$ posterior, corresponding to a uniform $\mathrm{Beta}(1,1)$ prior. Under that model, the interval contains 95% of the posterior probability for the heads proportion; this interpretation differs from a frequentist confidence interval.

### Streak calendar

The calendar displays 52 weeks of daily activity. Cell intensity represents the number of tosses on that date; hovering or focusing a cell shows its exact toss count and longest streak. Daily streak calculations use the browser's local calendar date and reset at midnight.

### Milestone timeline and frequency

The milestone panel shows a chronological timeline of the latest 24 maximal runs that reached at least four matching tosses. Its lifetime frequency summary counts how many maximal runs reached each threshold (`≥4`, `≥5`, and upward), so a run of length 6 contributes once to each threshold from 4 through 6.

### Education mode

Enable **Education mode** under **Analysis** to open an expandable probability guide inside the dashboard. It explains the fairness z-score and exact p-value, Wilson and Bayesian intervals, streak formulas, the law of large numbers, and why independent tosses do not make an opposite result “due” after a streak.

## Data Storage and Privacy

Flipper stores its state in browser `localStorage` under the legacy key `daily-toss-v1`. The key remains unchanged so existing users retain their history after the app rename.

- Data stays in the browser profile where it was created.
- Different browsers, devices, profiles, and private windows have separate data.
- Clearing site data removes the saved history.
- Exported JSON files provide a portable backup, but importing is not yet supported.
- Browser storage quotas vary. Very large histories may eventually exceed the available quota.

The dolphin icon is stored locally, so the interface does not depend on an image CDN. All application logic is contained in `index.html`.

## Project Structure

```text
Flipper/
├── assets/
│   └── dolphin.svg # Local app and favicon artwork
├── index.html      # Markup, styles, probability logic, and persistence
├── README.md       # Project documentation
└── NETLIFY_SETUP.md # Netlify and custom-domain deployment guide
```

## Deployment

For step-by-step deployment using `flipper.milagro-nexus.com`, see the [Netlify setup guide](NETLIFY_SETUP.md).

### GitHub Pages

1. Open the repository on GitHub.
2. Go to **Settings > Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**.
4. Select the `main` branch and `/ (root)` folder.
5. Save the configuration.

Netlify and GitHub Pages serve `index.html` automatically at the site root.

### Custom domain

After enabling GitHub Pages, enter the domain or subdomain under **Settings > Pages > Custom domain**. Configure the DNS records requested by GitHub:

- Use a `CNAME` record for a subdomain such as `flipper.example.com`.
- Use GitHub's documented `A` and `AAAA` records for an apex domain such as `example.com`.
- Enable **Enforce HTTPS** after DNS validation succeeds.

The same static file can also be hosted by Cloudflare Pages, Netlify, Vercel, Amazon S3, or a conventional web server.

## Browser Support

Flipper targets current versions of Chrome, Edge, Firefox, and Safari. It requires JavaScript, `localStorage`, CSS transforms, and Blob downloads.

## Current Limitations

- Auto flips run live only while the page is open; browser background throttling can delay timers.
- Catch-up is capped at 500 tosses per reopening.
- Random outcomes use `Math.random()` and are not cryptographically secure.
- Data does not synchronize across devices or browser profiles.
- JSON exports cannot currently be imported.
- There is no automated test suite or continuous deployment workflow.

## Possible Future Enhancements

The first group builds most directly on Flipper's exact probability calculations and lifetime statistics.

### Priority: statistical foundations

- Add the Wald-Wolfowitz runs test and lag autocorrelation checks for independence
- Add confidence bounds around expected maximal-run counts
- Correct milestone significance for repeated checking and multiple comparisons
- Chart cumulative heads proportion with a 95% interval and z-score over time
- Plot observed versus expected run-length distributions using the exact boundary formulas
- Add CUSUM or EWMA control charts for detecting gradual probability drift
- Offer a sequential probability ratio test with configurable error rates for early bias detection
- Explore waiting times for overlapping patterns such as `HTH` and `HHT`, including Penney's game

For a binary outcome, the exact binomial test should be the primary fairness test. A chi-square goodness-of-fit result may be included for education or comparison when its sample-size assumptions are satisfied.

### Data and persistence

- Import and validate JSON or CSV through a guided preview and conflict-resolution flow
- Merge past exports or external logs while preserving source and timestamp metadata
- Move large histories to IndexedDB while retaining lifetime aggregates
- Detect storage quota failures and show storage use, backup status, and visible warnings
- Add automatic backup reminders and restore previews
- Add optional encrypted cloud synchronization across devices
- Add session names, custom tags, and filters by date, mode, session, or tag
- Offer an append-only audit mode with checksums or a hash chain
- Offer an explicitly disclosed rolling retention window alongside lifetime totals
- Record trimming events and show lifetime, retained-window, and storage totals separately

### Auto-toss and scheduling

- Add pause and resume controls that retain the next scheduled time
- Show a catch-up summary with the number and time span of backfilled tosses
- Add timezone- and daylight-saving-aware daily schedules
- Support schedules such as weekdays at 9:00 or every 90 minutes during working hours
- Add optional milestone and auto-toss desktop notifications
- Add optional sound and haptic feedback
- Show clearer status when browser background throttling delays timers
- Support an installable Progressive Web App with offline assets
- Investigate deferred PWA catch-up or a server-backed scheduler for reliable background operation

Browser Background Sync does not guarantee exact periodic execution, so it cannot by itself promise a flip every few seconds or at an exact wall-clock time while the app is closed.

### User experience and visualization

- Add user-defined streak milestones and notification thresholds
- Support custom outcome labels such as Yes/No, Up/Down, or team names
- Complete an accessibility pass covering ARIA labels, focus management, reduced motion, and remappable shortcuts
- Add light, dark, and high-contrast themes
- Add localization for labels, dates, numbers, and probability descriptions

### Advanced and experimental

- Add an adjustable probability mode where $p \ne 0.5$ and visualize test-detection power
- Add Monte Carlo and randomization tests for comparison with exact results
- Provide deterministic seeded simulations for reproducible experiments
- Offer `crypto.getRandomValues()` as a stronger random source where supported
- Compare multiple coins or independent sequences side by side
- Export raw history and summaries as spreadsheet-friendly CSV
- Add optional webhooks or an API for external milestone logging

### Platform and development

- Add unit tests for recurrences, milestone detection, and migrations
- Add browser tests for persistence, keyboard behavior, and auto tossing
- Add GitHub Actions for validation and GitHub Pages deployment
- Introduce versioned data migrations and a documented export schema
- Add an open-source license and contribution guidelines

## Contributing

Before proposing a change, keep the app's core properties intact: it should remain understandable, privacy-conscious, responsive, and usable without a backend. Probability changes should include a derivation or a test against exact enumeration where practical.
