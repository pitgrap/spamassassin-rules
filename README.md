# spamassassin-rules

Custom SpamAssassin rules to mitigate spam mails and phishing mails.

Rules are maintained in the [`rules/`](rules/) folder and automatically published as an `sa-update`-compatible channel via [GitHub Pages](https://pitgrap.github.io/spamassassin-rules/).

## Channel subscription

### Quick start

Subscribe to the channel without GPG verification (suitable for testing):

```bash
sa-update --channel pitgrap.github.io/spamassassin-rules --no-gpg
```

### Persistent subscription

Create a channel configuration file so `sa-update` picks up the channel on every run:

```bash
# /etc/spamassassin/channel.d/pitgrap.conf
channel pitgrap.github.io/spamassassin-rules
```

Then run `sa-update` as usual (e.g. from a cron job):

```bash
sa-update
service spamassassin restart   # or: systemctl restart spamassassin
```

### DNS TXT record (version discovery)

`sa-update` discovers the current version via a DNS TXT record on the channel
hostname.  If you host this channel on a custom domain you control, add a TXT
record in the format expected by `sa-update`:

```
<major>.<minor>.<patch>.<channel-hostname>  TXT "<version-integer>"
```

For the default GitHub Pages hostname (`pitgrap.github.io`) no DNS control is
available, so you can pass `--version <N>` explicitly or use the `VERSION` file
served at <https://pitgrap.github.io/spamassassin-rules/VERSION> to determine
the latest version:

```bash
VER=$(curl -fsSL https://pitgrap.github.io/spamassassin-rules/VERSION)
sa-update --channel pitgrap.github.io/spamassassin-rules --version "$VER" --no-gpg
```

## Repository layout

```
rules/                   SpamAssassin rule files (.cf)
  00_custom_rules.cf     Custom rules for spam and phishing detection
.github/
  workflows/
    publish.yml          CI/CD: lint → tar → sha256 → GitHub Pages deploy
```

## CI/CD pipeline

Every push to `main` that touches the `rules/` folder triggers the workflow in
[`.github/workflows/publish.yml`](.github/workflows/publish.yml):

| Step | Description |
|------|-------------|
| **Lint** | `spamassassin --lint -C rules/` — validates rule syntax |
| **Archive** | `tar -czf <version>.tar.gz` of all rules |
| **Checksum** | `sha256sum` of the archive saved as `<version>.tar.gz.sha256` |
| **VERSION** | Plain-text file containing the current version integer |
| **Deploy** | Artifacts pushed to the `gh-pages` branch (GitHub Pages) |

The version number is the monotonically increasing git commit count
(`git rev-list --count HEAD`), which ensures `sa-update` always picks up
newer versions.

## Adding or modifying rules

1. Edit or create `.cf` files inside `rules/`.
2. Commit and push to `main`.
3. The workflow lints your changes and, if lint passes, publishes the updated
   channel automatically.

SpamAssassin rule syntax reference:
<https://spamassassin.apache.org/full/3.4.x/doc/Mail_SpamAssassin_Conf.html>

## License

[MIT](LICENSE)
