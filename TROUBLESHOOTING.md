# Troubleshooting

Common issues and solutions for OpenNews website development and deployment.

## Start Here: Quick Triage

**Site won't build?**

1. Read the error message carefully - it usually tells you the filename and line number
2. Check YAML syntax in the mentioned file
3. Look for unclosed Liquid tags in templates
4. Run `bundle exec rake clean && bundle exec rake build` for a fresh build

**Site builds but looks wrong?**

1. Hard refresh your browser (Ctrl+Shift+R or Cmd+Shift+R)
2. Check browser console (F12) for 404 errors
3. Verify file paths and case sensitivity
4. Check `_config.yml` baseurl and url settings

**Made changes but not seeing them?**

1. Some changes need server restart: `_config.yml`, `_data/*.yml`, new files
2. Check that Jekyll is done building (watch terminal output)
3. Hard refresh browser
4. Clear Jekyll cache: `bundle exec rake clean`

## Quick Reference: Rake Tasks

Most common issues can be resolved using built-in Rake tasks. See [tasks/README.md](tasks/README.md) for complete documentation.

1. `bundle exec rake build` - Build the Jekyll site
2. `bundle exec rake serve` - Build and serve the site locally at http://localhost:4000
   - 2a. Server auto-rebuilds on file changes (except `_config.yml` and `_data/*.yml`)
3. `bundle exec rake clean` - Clear Jekyll cache and built files
4. `bundle exec rake validate_yaml` - Validate all YAML files for syntax errors and duplicate keys
5. `bundle exec rake check` - Run configuration checks
6. `bundle exec rake test` - Run all validation tests
7. `bundle exec rake lint` - Check code formatting (Ruby + other files)
8. `bundle exec rake format` - Auto-fix all formatting issues
9. `bundle exec rake outdated` - Check for outdated gems
10. `bundle exec rake review:external_links` - Check external links (requires network)
11. `bundle exec rake deploy:precheck` - Run all pre-deployment checks

## Common Error Messages

| Error                                        | Solution                                                          |
| -------------------------------------------- | ----------------------------------------------------------------- |
| "Could not find gem..."                      | `bundle install` or delete `Gemfile.lock` and retry               |
| "Liquid syntax error"                        | Check for unclosed tags/mismatched delimiters; see error filename |
| "Could not locate included file"             | Verify file exists in `_includes/` with exact case                |
| "YAML Exception: mapping values not allowed" | Use spaces (not tabs), quote strings with colons                  |
| "Port 4000 already in use"                   | `lsof -ti:4000 \| xargs kill -9` or use `--port 4001`             |
| "Permission denied"                          | Never use `sudo` with bundle/gem; use rbenv/rvm on macOS          |

## Resources

- [README.md](README.md) - Development setup & deployment workflow
- [tasks/README.md](tasks/README.md) - Complete Rake task reference
- [Jekyll Docs](https://jekyllrb.com/docs/) - Template language reference

## Setup & Configuration

### Configuration validation errors

**Run `bundle exec rake check` to identify issues**, then:

- Update placeholder values in `_config.yml`
- Verify deployment bucket names and CloudFront distribution ID are set

## Environment & Dependencies

### Ruby version mismatch

- Check versions: `cat .ruby-version` vs `ruby --version`
- Required version is specified in `.ruby-version` file (currently 3.2.6)
- Install correct version:
  - rbenv: `rbenv install $(cat .ruby-version) && rbenv local $(cat .ruby-version)`
  - rvm: `rvm install $(cat .ruby-version) && rvm use $(cat .ruby-version)`

### Gem installation issues

- Run `bundle install` after updating Ruby version
- Delete `Gemfile.lock` and re-run `bundle install` if errors persist
- **Never use `sudo`** - install gems in user directory or use rbenv/rvm

### Node.js required for formatting

- Check: `node --version` (need 20+)
- Install: macOS `brew install node`, Linux `apt install nodejs npm`
- Run `npm install` for Prettier dependencies

### Platform-specific notes

- **Windows:** Use WSL for better compatibility
- **Case sensitivity:** Use exact case for file paths (matters on Linux/production)
- **Permissions:** Never use `sudo` with Ruby/Bundler on macOS/Linux

## Build & Cache Issues

### Build fails or server won't start

1. Run `bundle exec rake clean` to clear caches
2. Run `bundle install` to update dependencies
3. Check port 4000: `lsof -i :4000` (kill with `lsof -ti:4000 | xargs kill -9`)
4. Try with verbose output: `bundle exec jekyll serve --verbose`

### Stale or corrupted cache

- Run `bundle exec rake clean` (clears `.jekyll-cache` and `_site`)
- Hard refresh browser: Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac)
- Use incognito/private window for testing

## Jekyll & Template Errors

### Liquid syntax errors

- Check for unclosed tags: `{% if %}` needs `{% endif %}`
- Check mismatched delimiters: `{{` needs `}}`, `{%` needs `%}`
- Build locally to see specific error: `bundle exec rake build`
- Run `npm run format` to fix formatting issues

### Include or layout not found

- Verify file exists in `_includes/` or `_layouts/` with exact case
- Include with extension: `{% include navigation.html %}`
- Check build output for specific filename: `bundle exec rake build`

### Variables not rendering

- Use `default` filter: `{{ page.title | default: "No Title" }}`
- Debug with: `{{ page | jsonify }}`
- Check variable scope: `site.*` (config) vs `page.*` (front matter)

## YAML & Data Files

### YAML syntax errors

- Run `bundle exec rake validate_yaml` to check all YAML files
- Validate specific files at [yamllint.com](http://www.yamllint.com/)
- Use spaces (not tabs) for indentation
- Quote strings with colons: `title: "Session: Building Tools"`
- Build locally to see parsing errors: `bundle exec rake build`

### Front matter issues

- Must start/end with `---` (three dashes) at top of file
- Must be valid YAML (test at [yamllint.com](http://www.yamllint.com/))
- Strings with colons need quotes: `title: "Event: OpenNews Summit"`

## Deployment

### Site shows old content after deploy

- CloudFront cache takes 5-10 minutes to propagate after invalidation
- Check GitHub Actions logs to confirm deployment succeeded and cache invalidation triggered
- Staging updates appear immediately (no CloudFront caching)

### Deployment fails

- Verify `_config.yml` has correct S3 bucket names:
  - `deployment.bucket` for production
  - `deployment.staging_bucket` for staging
  - `deployment.cloudfront_distribution_id` for cache invalidation
- Check repository has access to `AWS_ROLE_ARN` secret
- Review GitHub Actions logs for specific errors

### Manual deployment (local)

**Dry-run first (recommended):**

```bash
bundle exec rake deploy:staging          # Dry-run to staging
bundle exec rake deploy:production       # Dry-run to production
```

**Then deploy for real:**

```bash
bundle exec rake deploy:staging:real     # Deploy to staging (requires confirmation)
bundle exec rake deploy:production:real  # Deploy to production (requires "yes")
```

**Run all pre-deployment checks:**

```bash
bundle exec rake deploy:precheck
# Runs: check, build, and test
```

## Git & GitHub

### Cannot push to branch

- Protected branches require PRs - create feature branch: `git checkout -b feature-name`
- For staging updates, push to `staging` branch
- For production updates, merge `staging` into `main` via PR

### Merge conflicts

- Check status: `git status`
- Resolve conflicts in marked files, then: `git add <file> && git commit`
- Or abort: `git merge --abort`

### GitHub Actions not triggering

- Check workflow file is in `.github/workflows/` and valid YAML
- Verify branch names match workflow triggers
- Check Actions tab and ensure Actions enabled (Settings → Actions)
- Push to `staging` branch triggers staging deployment
- Merge to `main` branch triggers production deployment

## Assets & Resources

### Missing images or broken links

- Verify file exists with exact case: `Logo.png` ≠ `logo.png`
- Check paths are from site root: `/media/img/logo.png`
- Load page locally and check browser console (F12) for 404 errors
- Visually inspect pages for broken images (broken icon or alt text)

### CSS/JS not loading

- Check browser console (F12) for 404 errors
- Hard refresh: Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac)
- Verify files exist in `media/css/` or `media/js/`
- Inspect page source (Ctrl+U) to verify correct paths in `<link>` and `<script>` tags

## Manual Smoke Testing

### Before committing changes

1. **Validate YAML:** `bundle exec rake validate_yaml` - check for syntax errors and duplicate keys
2. **Build locally:** `bundle exec rake build` - must succeed without errors
3. **Run tests:** `bundle exec rake test` - validate site structure and content
4. **Check formatting:** `bundle exec rake lint` - ensure code style is consistent
5. **Start dev server:** `bundle exec rake serve` - visit http://localhost:4000
6. **Visual inspection:** Check that pages render correctly, images load, styles apply
7. **Browser console:** Open DevTools (F12) - check for 404s or JavaScript errors
8. **Mobile view:** Toggle device toolbar in DevTools to check responsive layout
9. **Navigation:** Click through main navigation links to verify no broken pages

**Quick pre-deployment check:**

```bash
bundle exec rake deploy:precheck
```

This runs validate_yaml, check, build, and test in sequence.

### Code formatting

- Run `bundle exec rake format` to auto-fix all formatting issues (Ruby + other files)
- Run `bundle exec rake lint` to check formatting without changes
- Run `bundle exec rake format:ruby` to check only Ruby code with StandardRB
- Run `bundle exec rake format:prettier` to check only non-Ruby files with Prettier
- Alternatively: `npm run format` (Prettier only) or `npm run format:check`
- Ensure `npm install` has been completed for Prettier dependencies

### Running tests

**Run all tests:**

```bash
bundle exec rake test
```

This runs:

- `test:html_proofer` - Check internal links, images, and HTML structure
- `test:templates` - Check for Liquid template issues (unclosed tags, etc.)
- `test:page_config` - Validate page frontmatter
- `test:placeholders` - Check for TODO/FIXME placeholders
- `test:a11y` - Basic accessibility checks
- `test:performance` - Check for performance issues

**Run individual test suites:**

```bash
bundle exec rake test:html_proofer
bundle exec rake test:templates
bundle exec rake test:a11y
```

**Check external links (slower, requires network):**

```bash
bundle exec rake review:external_links
```

## Jekyll-Specific Gotchas

### Files and folders Jekyll ignores

- Files/folders starting with `.` (hidden) are ignored
- Files/folders starting with `_` are special (except `_config.yml`, `_data/`, `_includes/`, `_layouts/`, `_posts/`, `_plugins/`, `_site/`)
- Files listed in `exclude:` in `_config.yml` are ignored (e.g., `node_modules/`, `TROUBLESHOOTING.md`)

### Changes that require server restart

**Restart required:**

- `_config.yml` changes
- `_data/*.yml` changes
- New files/folders
- Gemfile changes (also run `bundle install`)

**No restart needed:**

- Content file changes (`.md`, `.html`)
- Template changes (`_includes/`, `_layouts/`)
- Asset changes (CSS, JS, images)
- Blog post changes in `_posts/`

### URL and path issues

- Paths in content should be absolute from site root: `/media/img/logo.png` not `media/img/logo.png`
- Check `baseurl` in `_config.yml` - should be empty for root domain
- Use `{{ site.baseurl }}` in templates for path prefix if needed
- Links to pages: `/what/` not `what.html`

## Getting Better Error Messages

### Build fails with vague error

- Run with `--trace` flag: `bundle exec rake build --trace`
- Check Jekyll verbose output: `bundle exec jekyll build --verbose`
- Look for filename and line number in error output
- Errors often appear several lines up from where the command stopped

### Server crashes or hangs

- Check for infinite loops in Liquid templates
- Look for circular includes (`{% include %}` calling itself)
- Verify no missing `{% endif %}` or `{% endfor %}` tags
- Restart with verbose: `bundle exec jekyll serve --verbose`

## Additional Help

If you're still stuck after trying these solutions:

1. Check the [Jekyll troubleshooting docs](https://jekyllrb.com/docs/troubleshooting/)
2. Search for the specific error message online
3. Review recent changes in git history: `git log --oneline -10`
4. Try rolling back recent changes to isolate the issue
5. Reach out to the OpenNews team for assistance
