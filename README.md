# Jekyll IS Publisher

[![GitHub License](https://img.shields.io/github/license/jekyll-is/action-jekyll-is-publish)](LICENSE)

**Jekyll IS Publisher** is a secure, composite GitHub Action for publishing pre-built Jekyll websites (or individual files) to GitHub Pages. It supports full directory synchronization, additional file copying, and optional Telegram notifications with detailed Git change logs.

## Features

- 🔄 **Synchronize entire Jekyll `_site` directory** with `--delete` option to clean obsolete files
- 📁 **Copy additional files** (e.g., sitemaps, robots.txt, custom assets)
- 🛡️ **Secure token handling** — uses Git credential helper (no tokens in logs)
- 📱 **Telegram notifications** with Git diff summary (limited to 20 files)
- ⚡ **Fail-fast execution** with `set -euo pipefail`
- 🏷️ **Custom commit messages** with labels and timestamps
- `--force` push to ensure clean deployment

## Usage

### Basic Jekyll Deployment

```yaml
name: Deploy Jekyll Site
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Jekyll
        run: |
          gem install bundler
          bundle install
          JEKYLL_ENV=production bundle exec jekyll build
      
      - name: Publish to GitHub Pages
        uses: jekyll-is/action-jekyll-is-publish
        with:
          label: "Jekyll CI/CD"
          sync-dir: "_site"
          github-actor: ${{ github.actor }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pages-repository: ${{ github.repository }}
          pages-branch: "gh-pages"
```

### With Additional Files + Telegram

```yaml
      - name: Publish with extras
        uses: jekyll-is/action-jekyll-is-publish
        with:
          label: "Jekyll + Extras"
          sync-dir: "_site"
          copy-dir: "_extras"  # robots.txt, sitemap.xml, etc.
          github-actor: ${{ github.actor }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pages-repository: ${{ github.repository }}
          telegram-bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          telegram-user-id: ${{ secrets.TELEGRAM_USER_ID }}
```

## Inputs

| Input              | Description                                      | Required | Default     |
|--------------------|--------------------------------------------------|----------|-------------|
| `label`            | Commit message label & notification prefix       | ✅ Yes   | -           |
| `sync-dir`         | Main directory to sync (e.g., `_site/`)          | ❌ No    | -           |
| `copy-dir`         | Additional files to copy (no delete)             | ❌ No    | -           |
| `github-actor`     | Git author name (e.g., `github-actions[bot]`)    | ✅ Yes   | -           |
| `github-token`     | Personal Access Token or `${{ secrets.GITHUB_TOKEN }}` | ✅ Yes | -           |
| `pages-repository` | Target repo (e.g., `user/repo`)                  | ✅ Yes   | -           |
| `pages-branch`     | Target branch (e.g., `gh-pages`, `pages`)        | ❌ No    | `gh-pages`  |
| `telegram-bot-token` | Telegram Bot API Token                        | ❌ No    | -           |
| `telegram-user-id` | Telegram Chat ID (user or channel)               | ❌ No    | -           |

## Security Notes

✅ **Token-safe**: Uses `git config --global url."https://ACTOR:TOKEN@github.com/".insteadOf` — tokens never appear in logs[5]

✅ **Scoped permissions**: Use `GITHUB_TOKEN` with `contents: write` & `pages: write` workflow permissions

⚠️ **Known limitation**: `git clone --branch` fails if branch doesn't exist. Create target branch manually first or use a setup step.

## Telegram Notifications

Sends formatted message with:
```
Publishing by Jekyll CI/CD @ Fri Nov 28 20:18:00 UTC 2025

5 file(s) changed:
​```
A       index.html
M       css/style.css
D       old-page.html
​```
```

**MarkdownV2** formatting with full escaping for Git filenames/symbols.

## Workflow Permissions

Add to your workflow YAML:
```yaml
permissions:
  contents: write
  pages: write
  id-token: write  # For GitHub Pages deployment
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `fatal: Remote branch not found` | Create target branch manually: `git checkout --orphan gh-pages` |
| `rsync: No such file or directory` | Verify `sync-dir`/`copy-dir` exist in runner workspace |
| Telegram fails silently | Check bot token permissions & chat ID format |
| No changes committed | Use `git status` after `rsync` to debug |

## Development

1. **Test locally**:
   ```bash
   act -j deploy --container-architecture linux/amd64
   ```

2. **Release new version**:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

## License

[GPLv3+](LICENSE) © 2025 [Ivan Shikhalev](https://github.com/shikhalev)

