# deepakguntoju.github.io

Personal portfolio site — Jekyll + minima, published via GitHub Pages. No React, no backend, no analytics.

Live at: https://deepakguntoju.github.io

## Structure

- `index.md` — home
- `about/` — about page
- `projects/` — projects landing + two case studies (Security Enablement Platform, Native SAST & Reachability Engine)
- `resume/` — resume page + `assets/resume/Guntoju_Deepak_Resume.pdf`

Content sourced from `SETUP_DAY/PORTFOLIO_REVIEW/website-content/` (private staging, not in this repo). Technical evidence for both case studies lives in the public [security-enablement-platform-poc](https://github.com/deepakguntoju/security-enablement-platform-poc) repo — this site links to it rather than duplicating it.

## Local development

```bash
docker run --rm -v "$PWD":/srv/jekyll -p 4000:4000 jekyll/jekyll jekyll serve
```
