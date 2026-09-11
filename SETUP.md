# Publishing the support site on GitHub Pages

Three pages, one stylesheet, no build step and no external requests — a password
manager's privacy policy shouldn't itself pull a font from a CDN. Publishing takes
about five minutes.

## Before you publish: two placeholders to replace

Both appear in all three HTML files.

1. **`support@example.com`** → your real support address. Apple's reviewers do
   check that the support page offers a working way to reach you, and a dead
   address is a guideline 1.5 rejection.
2. **`Last updated: 10 September 2026`** in `privacy.html` → the date you actually
   publish, if it differs.

Replace the address in place:

```bash
cd AppStore/site
sed -i '' 's/support@example\.com/you@yourdomain.com/g' index.html privacy.html support.html
grep -c "support@example.com" index.html privacy.html support.html   # expect 0 0 0
```

Grep for the full address, not for `example.com`. `privacy.html` uses a bare
`<code>example.com</code>` as the illustrative domain in the icon-fetching section,
and that one should stay — matching on the shorter string makes a clean replacement
look like a failed one.

`privacy.html` also links to DuckDuckGo's own privacy policy, which is intentional
and should stay too — it's the one third party the app ever contacts.

## Publish

```bash
mkdir -p ~/Source/mywords-support && cd ~/Source/mywords-support
cp /Users/beam/Source/mywords/mobile/AppStore/site/{index,privacy,support}.html style.css .
git init -b main
git add .
git commit -m "MyWords support site: landing, privacy policy, support/FAQ"
gh repo create mywords-support --public --source=. --push
```

Then turn Pages on. Either in the web UI — **Settings → Pages → Source: Deploy from
a branch → `main` / `(root)` → Save** — or from the CLI:

```bash
gh api -X POST repos/:owner/mywords-support/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
```

The repository must be **public** for Pages on a free account, which is fine —
these pages are meant to be world-readable.

## Your URLs

Substitute your GitHub username:

| App Store Connect field | URL |
|---|---|
| **Support URL** | `https://danklet.github.io/mywords/support.html` |
| **Privacy Policy URL** | `https://danklet.github.io/mywords/privacy.html` |
| Marketing URL *(optional)* | `https://danklet.github.io/mywords/` |

## Verify before you paste them into App Store Connect

First deploy takes a minute or two. Then check all three return `200`:

```bash
U=<username>
for p in "" privacy.html support.html; do
  printf '%s → ' "$p"
  curl -s -o /dev/null -w '%{http_code}\n' "https://$U.github.io/mywords-support/$p"
done
```

Apple fetches both URLs during review. A 404 on either — including from a Pages
deploy that hasn't finished — is a straightforward rejection, so confirm the `200`s
before submitting rather than after.
