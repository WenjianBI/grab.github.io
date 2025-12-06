# 🎉 GRAB Documentation Site - Complete Setup Guide

## ✅ What Has Been Done

### 1. **Math Support Fixed** ✨
- Created `_includes/head_custom.html` with MathJax configuration
- LaTeX equations now render properly: `$inline$` and `$$display$$`
- Example: `$$\sum_{i=1}^n X_i R_i = 0$$` will display beautifully

### 2. **Theme & Appearance Improved** 🎨
- Updated to official Just the Docs theme
- Added custom CSS for better appearance:
  - Code blocks with purple left border
  - Styled tables with header backgrounds
  - Enhanced blockquotes/notes with backgrounds
  - Better inline code formatting
  - Proper badge alignment

### 3. **Configuration Fixed** ⚙️
- Fixed `baseurl` from `grab.github.io/` to `/grab.github.io`
- Added search functionality
- Enabled kramdown with MathJax
- Added professional footer with copyright
- Added "Edit on GitHub" links
- Improved SEO with jekyll-seo-tag

### 4. **Files Cleaned Up** 🧹
- Removed `_site/` (build directory)
- Removed `.jekyll-cache/` (cache)
- Removed `.lh/` (local history)
- Removed `Gemfile.lock` (will be regenerated)
- Created `.gitignore` to prevent these from being committed
- Updated README.md with clear structure

### 5. **Documentation Structure** 📚
```
grab.github.io/
├── _config.yml          ← Site configuration
├── _includes/
│   └── head_custom.html ← Math & custom styling
├── docs/                ← All documentation files
│   ├── mainpage.md      ← Home page
│   ├── approach*.md     ← Method documentation
│   ├── GRM.md           ← GRM documentation
│   └── simulation*.md   ← Simulation docs
├── assets/              ← Site assets
├── img/                 ← Images
├── Gemfile              ← Ruby dependencies
└── README.md            ← Repository info
```

## 🚀 Deploy Your Changes

### Step 1: Review Changes
```bash
cd /c/Users/Woody/OneDrive/Code/WenjianBI/grab.github.io
git status
```

### Step 2: Add All Changes
```bash
git add .
```

### Step 3: Commit
```bash
git commit -m "Major documentation improvements: math support, better theme, and cleanup"
```

### Step 4: Push to GitHub
```bash
git push origin grab-pages
```

### Step 5: Wait & Check
- Wait 1-2 minutes for GitHub Pages to rebuild
- Visit: https://wenjianbi.github.io/grab.github.io/
- Check that:
  - ✅ Math equations render properly
  - ✅ Site looks professional
  - ✅ Navigation works
  - ✅ Search works

## 🎯 What's Improved

### Before ❌
- Math equations showed as raw LaTeX
- Plain, basic appearance
- Wrong baseurl causing routing issues
- Build artifacts committed to git
- Cluttered file structure

### After ✅
- Math renders beautifully with MathJax
- Professional Just the Docs theme
- Custom styling for better readability
- Clean git repository
- Organized documentation structure
- Working search
- Mobile-responsive
- SEO optimized

## 📝 Optional Next Steps

### Remove Unused Files (Optional)
If you want an even cleaner repository, you can remove:

1. **about.markdown** - Already excluded from navigation
   ```bash
   git rm about.markdown
   ```

2. **index.markdown** - Can use mainpage.md directly
   ```bash
   git rm index.markdown
   ```
   Then update `_config.yml` to set `docs/mainpage.md` as the home page.

### Customize Further
Edit `_includes/head_custom.html` to:
- Change color scheme (currently purple: `#7253ed`)
- Adjust font sizes
- Add custom fonts
- Modify spacing

## 🧪 Test Locally (Optional)

```bash
# Install dependencies
bundle install

# Run local server
bundle exec jekyll serve

# Visit in browser
# http://localhost:4000/grab.github.io/
```

## 📋 File Purpose Reference

| File/Folder | Purpose | Keep? |
|-------------|---------|-------|
| `_config.yml` | Site configuration | ✅ Required |
| `_includes/` | Custom HTML (MathJax) | ✅ Required |
| `docs/` | All documentation | ✅ Required |
| `assets/` | Site assets | ✅ Required |
| `img/` | Documentation images | ✅ Required |
| `Gemfile` | Ruby dependencies | ✅ Required |
| `README.md` | Repository info | ✅ Recommended |
| `.gitignore` | Git exclusions | ✅ Recommended |
| `404.html` | Error page | ✅ Recommended |
| `about.markdown` | About page (excluded) | ⚠️ Optional |
| `index.markdown` | Landing page | ⚠️ Optional |
| `cleanup.bat/sh` | Cleanup scripts | ⚠️ Can remove after use |
| `DEPLOYMENT_SUMMARY.md` | This guide | ⚠️ Can remove after reading |

## ❓ Troubleshooting

### Math Still Not Rendering?
1. Clear browser cache (Ctrl+Shift+R)
2. Wait a few minutes for GitHub Pages CDN to update
3. Check browser console (F12) for JavaScript errors
4. Verify MathJax CDN is loading (check Network tab)

### Site Looks Broken?
1. Verify `baseurl: "/grab.github.io"` in `_config.yml`
2. Check GitHub Pages settings (should be on `grab-pages` branch)
3. Look at GitHub Actions for build errors
4. Try force-refreshing: Ctrl+Shift+R

### Search Not Working?
1. Wait for GitHub Pages to rebuild (1-2 minutes)
2. Clear cache
3. Check that `search_enabled: true` in `_config.yml`

### Links Not Working?
- Use relative URLs: `[link](file.md)` not `[link](/file.md)`
- For cross-references between docs, use: `[text](other-doc.md)`

## 🎓 Writing Tips

### Math Equations
```markdown
Inline: $x^2 + y^2 = z^2$

Display:
$$
\sum_{i=1}^n X_i R_i = 0
$$
```

### Code Blocks
````markdown
```r
# R code
library(GRAB)
```

```bash
# Shell code
conda install grab
```
````

### Notes/Blockquotes
```markdown
> **Note:**  
> Important information here.
```

### Tables
```markdown
| Column 1 | Column 2 |
|----------|----------|
| Value 1  | Value 2  |
```

## 📞 Support

- **Package Issues:** https://github.com/GeneticAnalysisinBiobanks/GRAB/issues
- **Documentation Issues:** https://github.com/WenjianBI/grab.github.io/issues
- **Email:** wenjianb@pku.edu.cn

## 🎉 You're Done!

Your documentation site is now:
- ✅ Professional looking
- ✅ Math-enabled
- ✅ Well-organized
- ✅ Mobile-friendly
- ✅ Searchable
- ✅ SEO-optimized

Just commit and push to see the improvements live!
