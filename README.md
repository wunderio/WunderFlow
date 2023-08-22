# Git WunderFlow

## Description

WunderFlow is a Git workflow that tries to make it easier to have multiple ongoing development tracks simultaneously while still allowing clean releases and steady hotfixes. It also makes it easy to show any unfinished work to customers.

## Main branches

### Develop

- Branch for showing new features to customer etc.
- Deployed to `dev` server.
- Feature / hotfix branches can be merged here anytime for testing purposes.
- This branch can be reset to `main` anytime, so no code living only in this branch will end up in `production`.
- It is also possible to skip this branch if it is possible to use environment per branch.

### Main

- Integration / QA for next release.
- Deployed to `stage` server.
- Base branch for all new features.

### Production

- Production.
- Always equal to the actual code currently running on `production` servers.

## Development workflow

### New feature

- Create new `feature/[issue/ticket id]-issueTitle` branch from `main`.
- Merge to `develop` for testing / acceptance.
- Create a pull request against `main` for peer review.
- Merge to `main` once finished / accepted.

### Epic features

Sometimes there might be bigger project that needs to be developed separately and where different features are dependant from each other.

- Create new `epic` branch `epic/[EpicName]` from `main`.
- Create all the feature branches that belong to this project from the `epic` branch as follows: `feature/[EpicName]/[featureInThisEpic]`.
- Merge feature branches to `epic` branch, then `epic` branch to `develop` for testing.
- If needed `epic` branch can be reset to `main` to clean up experimental features.
- Only delete feature branches after the `epic` branch is accepted to `main`.
- For release treat `epic` branch as any other feature branch.

### Hotfix

- Create new `hotfix/[ticket id]-issueTitle` branch from `production`.
- Merge to `develop` for testing.
- Create a pull request against `production` for peer review.
- Merge to `production` for release.
- Rebase `main` to `production`.

### Release

- Merge all accepted / finished feature branches to `main` (that are not yet merged).
- Delete finished feature branches.
- Run tests on `main`.
- Tag new release.
- Merge to `production`.

## Examples

### Create a new feature and push it to develop for testing

```sh
git checkout main
git pull origin main
git checkout -b feature/ABC-1234-adding_this
git add
git commit [... reiterate as many time as needed]
git push origin feature/ABC-1234-adding_this
git checkout develop
git pull origin develop
git merge --no-ff feature/ABC-1234-adding_this
git push origin develop
```

### Create a hotfix

```sh
git checkout production
git pull origin production
git checkout -b hotfix/ABC-1234-fix
git add
git commit [... reiterate as many time as needed]
git push origin hotfix/ABC-1234-fix
git checkout develop
git pull origin develop
git merge --no-ff hotfix/ABC-1234-fix
git push origin develop
[test again]
git checkout production
git pull origin production
git merge --no-ff hotfix/ABC-1234-fix
git push origin production
git rebase production main
git push origin main
```

### Publish a feature to main for the pre-release

```sh
git checkout main
git pull origin main
git merge --no-ff feature/ABC-1234-adding_this
git push origin main
```

### Push main to production

```sh
git checkout production
git pull origin production
git merge --no-ff main
git tag -a mytag -m “mytag”
git push origin production
git push --tags
```

## Diagrams

### WunderFlow

![WunderFlow](https://raw.githubusercontent.com/wunderio/wunderflow/master/img/WunderFlow1.png)

### WunderFlow Epic

![WunderFlow Epic](https://raw.githubusercontent.com/wunderio/wunderflow/master/img/WunderFlow_epic1.png)
