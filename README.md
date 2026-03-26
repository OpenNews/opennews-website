## OpenNews

This is the website for [OpenNews](https://opennews.org).

OpenNews is a Knight Foundation-funded project that supports the growing community of news developers, designers, and data analysts helping journalism thrive on the open web.

## Development Setup

### Prerequisites

- Ruby 3.2.6 (specified in [.ruby-version](.ruby-version))
- Node.js 20+
- Bundler (`gem install bundler`)

### Initial Setup

1. Clone this repository to your local machine
2. Install Ruby dependencies: `bundle install`
3. Install Node dependencies: `npm install`

### Local Development Commands

**Start local development server:**

```bash
bundle exec rake -T               # see all available tasks
bundle exec rake serve            # serve locally at localhost:4000
bundle exec rake check            # runs various pre-deploy checks
bundle exec rake build            # build static site ready to deploy
bundle exec rake test             # run all tests
bundle exec rake deploy:precheck  # runs validate_yaml, check, build, and test
bundle exec rake lint             # list code formatting errors (ruby et al)
npm run format:check              # list code formatting errors (node)
bundle exec rake format           # auto-fix all formatting issues

# less frequent
bundle exec rake validate_yaml    # validate all YAML
bundle exec rake clean            # kill caches and modules
bundle exec rake outdated         # check for outdated dependencies

# rare (requires local build in _site/)
bundle exec rake review:external_links # get list of bad links (401s, 403s, 404s, timeouts)
bundle exec rake review:compare_deployed_sites # compare current build to production
```

For detailed task documentation, see [tasks/README.md](tasks/README.md).

## How to update the OpenNews site

### Configuration

Before deploying, ensure the AWS S3 bucket names and CloudFront distribution ID are configured in [\_config.yml](_config.yml):

```yaml
deployment:
  bucket: YOUR_PRODUCTION_BUCKET_HERE
  staging_bucket: YOUR_STAGING_BUCKET_HERE
  cloudfront_distribution_id: YOUR_CLOUDFRONT_ID_HERE
```

### Workflow

- For minor updates, work directly in the `staging` branch
- For major updates or long-term changes, create a new feature branch
- Test your changes locally using `bundle exec rake serve`
- your local `_site` will never be committed, it's just a way to see what Jekyll sees

### Pushing to staging

- When ready for review, push to the `staging` branch in GitHub
- If working in a separate feature branch, open a pull request into `staging` and merge it
- A push to `staging` triggers an automatic GitHub Actions workflow that:
  1. Validates YAML files
  2. Builds the Jekyll site
  3. Deploys to the staging S3 bucket and the [staging site](http://staging.opennews.org/)
- The workflow typically completes in 2-3 minutes
- View the [staging site](http://staging.opennews.org/) to smoke-test your changes

### Pushing to production

- Open a pull request from `staging` into `main`
- Merging into `main` triggers an automatic deployment that:
  1. Validates YAML files
  2. Builds the Jekyll site
  3. Deploys to the production S3 bucket
  4. Invalidates the CloudFront cache for the [production site](https://www.opennews.org/)
- The production site is served via Amazon CloudFront (HTTPS-enabled)
- Cache invalidation may take up to 10 minutes to propagate fully

### Manual deployment (local, very rarely needed)

**Note:** Manual deployments require AWS CLI configured with appropriate credentials.

For testing deployments locally **when automated deployment isn't available**:

**Dry-run first (recommended):**

```bash
bundle exec rake deploy:staging        # Test staging deploy
bundle exec rake deploy:production     # Test production deploy
```

`deploy:staging` and `deploy:production` are aliases for each namespace's default dry-run task.

**Then deploy for real:**

```bash
bundle exec rake deploy:staging:real      # Deploy to staging (requires confirmation)
bundle exec rake deploy:production:real   # Deploy to production (requires "yes")
```

### Optional promotion safety check

After deploying to staging, you can compare deployed staging and production content before promoting:

```bash
bundle exec rake review:compare_deployed_sites
```

This fetches both deployed environments and reports significant page-level differences.

## CI/CD

### GitHub Actions Workflows

**Test Workflow** ([.github/workflows/test.yml](.github/workflows/test.yml))

- Runs on pull requests and non-deployment branches
- Validates YAML, builds site, runs HTML checks
- Tests deployment command with dry-run

**Deploy Workflow** ([.github/workflows/deploy.yml](.github/workflows/deploy.yml))

- Runs on push to `main` (production) or `staging` branches
- Uses AWS OIDC authentication (no long-lived credentials)
- Automatically deploys to appropriate environment

**Health Check Workflow** ([.github/workflows/health-check.yml](.github/workflows/health-check.yml))

- Runs weekly on Mondays at 9am UTC
- Reports outdated dependencies
- Creates GitHub issues when updates are available

### AWS Authentication

Deployment uses OpenID Connect (OIDC) for secure AWS authentication.
The `AWS_ROLE_ARN` secret **is** and must be configured at the organization level in GitHub.

## Code Quality

### Formatting & Linting

- **Prettier**: Formats HTML, CSS, JavaScript, JSON, YAML, Markdown, and Ruby files
- **StandardRB**: Ruby linting (configured in [.standard.yml](.standard.yml))
- **EditorConfig**: Universal editor settings
- **VS Code**: Recommended settings in [.vscode/settings.json](.vscode/settings.json) enable format-on-save

Run `bundle exec rake lint` to check formatting, or `bundle exec rake format` to auto-fix issues.

### Testing

The site includes comprehensive automated tests:

- **HTML Proofer**: Validates internal links, images, and HTML structure
- **Template Validation**: Checks for Liquid syntax errors and unclosed tags
- **Accessibility**: Basic a11y checks for alt text, lang attributes, form labels
- **Performance**: Checks for large files and inline styles
- **Placeholders**: Detects TODO/FIXME markers in built site
- **External Links**: Validates external URLs (requires network access)

Run `bundle exec rake test` to run all tests, or see [tasks/README.md](tasks/README.md) for individual test commands.

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common issues and solutions.
