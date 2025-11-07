# CircleCI configuration for Drupal projects

Complete CircleCI configuration using [Silta orb](https://circleci.com/developer/orbs/orb/silta/silta) that supports the WunderFlow branching strategy.

## Overview

The configuration supports four environments:

- **Feature branches**: Deploy to feature environments (manual approval required)
- **Test branch**: Deploy to test environment (automatic)
- **Main branch**: Deploy to staging environment (automatic)
- **Production tags**: Deploy to production (manual approval required)

## Complete configuration

Create `.circleci/config.yml`:

```yaml
version: 2.1

orbs:
  silta: silta/silta@1

executors:
  silta:
    docker:
      - image: wunderio/silta-cicd:circleci-php8.3-node22-composer2-v1

# Define shared branch filters
branch-filters:
  # Branches that skip feature deployment
  ignored-branches: &ignored-branches
    branches:
      ignore:
        - production
        - main
        - test
        - /dependabot\/.*/

workflows:
  commit:
    jobs:
      # Validation runs on all branches and tags
      # IMPORTANT: Must include tag filters to allow production build job to run
      - silta/drupal-validate:
          name: validate
          executor: silta
          post-validation:
            - run: echo "Add additional validation here if needed"
          filters:
            tags:
              only: /^[0-9]+\.[0-9]+\.[0-9]+$/
            branches:
              only: /.*/

      # Manual approval for feature branch deployments
      - approval:
          type: approval
          name: approve-deployment
          filters:
            <<: *ignored-branches

      # Build job for feature environments
      - silta/drupal-build: &build
          name: build
          executor: silta
          codebase-build:
            - silta/drupal-composer-install
            - silta/npm-install-build:
                path: web/themes/custom/theme_name
          context: silta_dev
          requires:
            - validate
            - approve-deployment
          filters:
            <<: *ignored-branches

      # Deploy job for feature environments
      - silta/drupal-deploy: &deploy
          name: deploy
          executor: silta
          silta_config: silta/silta.yml,silta/secrets
          pre-release:
            - silta/decrypt-files:
                files: silta/secrets
          context: silta_dev
          requires:
            - build
          filters:
            <<: *ignored-branches

      # Build job for test environment
      - silta/drupal-build:
          <<: *build
          name: build-test
          requires:
            - validate
          filters:
            branches:
              only: test

      # Deploy job for test environment
      - silta/drupal-deploy:
          <<: *deploy
          name: deploy-test
          silta_config: silta/silta.yml,silta/secrets
          pre-release:
            - silta/decrypt-files:
                files: silta/secrets
          requires:
            - build-test
          filters:
            branches:
              only: test

      # Build job for staging environment (main branch)
      - silta/drupal-build:
          <<: *build
          name: build-staging
          requires:
            - validate
          filters:
            branches:
              only: main

      # Deploy job for staging environment (main branch)
      - silta/drupal-deploy:
          <<: *deploy
          name: deploy-staging
          silta_config: silta/silta.yml,silta/silta-staging.yml,silta/secrets
          pre-release:
            - silta/decrypt-files:
                files: silta/secrets
          requires:
            - build-staging
          filters:
            branches:
              only: main

      # Build job for production environment
      # Runs only on semantic version tag pushes (1.0.0, 1.1.0, etc.)
      - silta/drupal-build:
          <<: *build
          name: build-prod
          context: silta_finland
          requires:
            - validate
          filters:
            tags:
              only: /^[0-9]+\.[0-9]+\.[0-9]+$/
            branches:
              ignore: /.*/

      # Manual approval for production deployment
      - approval:
          type: approval
          name: approve-prod-deployment
          requires:
            - build-prod
          filters:
            tags:
              only: /^[0-9]+\.[0-9]+\.[0-9]+$/
            branches:
              ignore: /.*/

      # Deploy job for production environment
      # Runs only after manual approval on tag pushes
      - silta/drupal-deploy:
          <<: *deploy
          name: deploy-prod
          silta_config: silta/silta.yml,silta/silta-prod.yml,silta/secrets-prod
          pre-release:
            - silta/decrypt-files:
                files: silta/secrets-prod
          context: silta_finland
          requires:
            - approve-prod-deployment
          filters:
            tags:
              only: /^[0-9]+\.[0-9]+\.[0-9]+$/
            branches:
              ignore: /.*/

      # Build job for dependabot branches (no deployment)
      - silta/drupal-build:
          <<: *build
          name: build-dependabot
          skip-deployment: true
          filters:
            branches:
              only: /dependabot\/.*/
```

## Configuration breakdown

### Executors

```yaml
executors:
  silta:
    docker:
      - image: wunderio/silta-cicd:circleci-php8.3-node22-composer2-v1
```

Defines the Docker image used for all jobs. Update PHP and Node versions as needed.

### Branch filters

```yaml
branch-filters:
  ignored-branches: &ignored-branches
    branches:
      ignore:
        - production
        - main
        - test
        - /dependabot\/.*/
```

Defines which branches skip feature deployment (they have their own workflows).

### Validation job

Runs on all branches and tags:

```yaml
- silta/drupal-validate:
    name: validate
    executor: silta
    post-validation:
      - run: echo "Add additional validation here if needed"
```

### Feature branch workflow

1. **Validate** - Code quality checks
2. **Manual approval** - Prevents unnecessary deployments
3. **Build** - Composer install, npm build
4. **Deploy** - Deploy to feature environment

**Environment URL**: `https://BRANCH-NAME.project-name.dev.wdr.io`

### Test branch workflow

Automatic deployment to test environment:

```yaml
filters:
  branches:
    only: test
```

**Environment URL**: `https://test.project-name.dev.wdr.io`

### Main branch workflow (staging)

Automatic deployment to staging environment:

```yaml
filters:
  branches:
    only: main
```

**Environment URL**: `https://main.project-name.dev.wdr.io`

### Production workflow

Triggered by semantic version tags:

```yaml
filters:
  tags:
    only: /^[0-9]+\.[0-9]+\.[0-9]+$/
  branches:
    ignore: /.*/
```

**Tag format**: `1.2.3` (not `v1.2.3`)

**Workflow**:

1. Validate
2. Build production
3. Manual approval (required)
4. Deploy to production

## Customization

### Update theme path

Change the npm build path to match your theme if needed:

```yaml
- silta/npm-install-build:
    path: web/themes/custom/YOUR_THEME_NAME
```

### Add custom validation

Add custom checks in the validation job:

```yaml
post-validation:
  - run: npm run lint
```

### Different contexts

Use different Silta contexts for different environments:

```yaml
context: silta_dev        # For feature/test/staging
context: silta_finland    # For production
context: silta_custom     # For custom setup
```

## Environment URLs

Based on this configuration:

| Branch/Tag | Environment | URL | Approval |
|------------|-------------|-----|----------|
| `TICKET-123-feature` | Feature | `https://ticket-123-feature-a1b2c3.project.dev.wdr.io` | Required |
| `test` | Test | `https://test.project.dev.wdr.io` | Automatic |
| `main` | Staging | `https://main.project.dev.wdr.io` | Automatic |
| `1.2.3` | Production | `https://production.example.com` | Required |

**Note**: Feature branch URLs are sanitized (lowercase, hyphens) with a hash suffix for uniqueness.

## Troubleshooting

### Feature deployment not starting

**Issue**: Pushed feature branch but no deployment

**Solution**: Approve deployment in CircleCI web interface

### Production deployment not triggered

**Issue**: Pushed tag but workflow didn't run

**Solution**: Ensure tag format is `1.2.3` (not `v1.2.3`)

### Build fails on validation

**Issue**: Validation job fails

**Solution**: Run validation locally before pushing:

```bash
ddev grumphp run
ddev phpstan analyze
ddev phpcs
```

### Wrong environment deployed

**Issue**: Branch deployed to wrong environment

**Solution**: Check branch filters in config.yml match your branch name

## See also

- [WunderFlow README](../README.md)
- [Silta orb](https://circleci.com/developer/orbs/orb/silta/silta)
- [Silta documentation](https://github.com/wunderio/silta)
