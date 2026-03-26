# Rake Tasks (Ruby)

This directory contains task modules that are loaded by the main `Rakefile`.

Note: several top-level tasks are defined directly in the root `Rakefile` (for example: `validate_yaml`, `check`, `build`, `serve`, `clean`, and `deploy:*`).

## Available Task Files

### `format.rake` - Code Formatting & Linting

Format and lint Ruby and non-Ruby files to maintain consistent code style.

- `rake lint` - Check all code formatting (Ruby with StandardRB + HTML/CSS/JS/YAML/Markdown with Prettier)
- `rake format` - Auto-fix all formatting issues
- `rake format:ruby` - Check Ruby code style with StandardRB
- `rake format:ruby_fix` - Auto-fix Ruby formatting
- `rake format:prettier` - Check non-Ruby files with Prettier via npm/NodeJS
- `rake format:prettier_fix` - Auto-fix non-Ruby files

### `test.rake` - Site Validation Tests

Comprehensive testing suite for the built site.

- `rake test` - Run all tests
- `rake test:html_proofer` - Test built site with html-proofer (checks links, images, etc.)
- `rake test:templates` - Check for Liquid template issues
- `rake test:page_config` - Validate page frontmatter configuration
- `rake test:placeholders` - Check for placeholder content (TODO, FIXME, etc.)
- `rake test:a11y` - Test for common accessibility issues
- `rake test:performance` - Check for performance issues

### `review.rake` - External Link Validation

Validate external/public URLs in the built site (slower, requires network access).

- `rake review:external_links` - Check all external links in the built site for validity
- `rake review:compare_deployed_sites` - Compare deployed staging vs production page content

**Notes:**

- `review:external_links` requires the site to be built first (`rake build`) and performs real HTTP requests to external URLs.
- `review:compare_deployed_sites` requires deployment bucket config in `_config.yml` and compares URLs derived from those settings.
- `review:compare_deployed_sites` also supports optional `EXTRA_PATHS` and `EXTRA_PATHS_FILE` environment variables for legacy/archive URLs not present in local `_site`.

### `outdated.rake` - Dependency Updates

Check for outdated Ruby gems.

- `rake outdated` - Check directly used outdated dependencies
- `rake outdated:direct` - Check only direct dependencies from Gemfile
- `rake outdated:all` - Check all outdated dependencies (including transitive)

## Adding New Tasks

To add a new task file:

1. Create a new `.rake` file in this directory
2. Use Ruby's `namespace` to organize related tasks
3. Add descriptive `desc` comments for user-facing tasks
4. The main `Rakefile` will automatically load it

Example:

```ruby
namespace :myfeature do
  desc "Do something useful"
  task :action do
    puts "Doing something useful..."
    # Your code here
  end
end
```

## Task Dependencies

Tasks can depend on other tasks using the dependency syntax:

```ruby
desc "Run all checks before deploying"
task precheck: %i[validate_yaml build test] do
  puts "All checks passed!"
end
```

## Testing Tasks Locally

Run any task with:

```bash
bundle exec rake <task_name>
```

List all available tasks:

```bash
bundle exec rake -T
```

Show detailed task descriptions:

```bash
bundle exec rake -D
```
