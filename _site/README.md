## OpenNews
This is the website for [OpenNews](https://opennews.org).

OpenNews is a Knight Foundation-funded project that supports the growing community of news developers, designers, and data analysts helping journalism thrive on the open web.

## How to update the OpenNews site
### Testing changes locally
* Clone this repository to your local machine.
* For minor updates, work directly in the `staging` branch. For major updates, or if you're working on long-term changes to the site, create a new feature branch.
* To test your work locally, run `jekyll serve` or `jekyll build`, and view the site in a browser.

### Pushing to staging
* When you're ready to have someone review a site update, update the `staging` branch in GitHub. If you're working in `staging` locally, you just need to push your code changes. If you're working in a separate feature branch, push that branch to GitHub and then open a pull request into `staging` and merge it.
* NOTE: You do _not_ need to commit updates to your local `_site` directory after you run `jekyll build` or `jekyll serve`. You only need to commit new or updated markdown documents and templates, and new or updated static media files.
* A commit to the `staging` branch on GitHub will trigger an automatic build of the staging site. This runs its own `jekyll build` process before copying everything to S3. (So any changes you've committed to the repo's `_site` directory will be ignored.)
* The Travis CI process that handles this can take a minute or two to complete. Your changes will not be visible on the staging site immediately, but they'll be there quickly.

### Pushing to production
* Review your changes on the staging site, and if everything looks OK, come back to this repo and open a pull request from `staging` into `master`.
* Merging a pull request into `master`, or pushing any commit to the `master` branch, will trigger an automatic build of the production site at [opennews.org](https://opennews.org). Again, this runs its own `jekyll build` process before copying to S3, ignoring any committed changes to the repo's `_site` directory.
* The production site is delivered through Amazon CloudFront so that we can serve a secure, https-enabled [opennews.org](https://opennews.org). CloudFront also caches everything for performance. The rebuild process triggers an invalidation of the entire cache, but it still may take up to 10 minutes for site changes to be reflected on production.