# Git WunderFlow

## Description
WunderFlow is a Git workflow that tries to make it easier to have multiple ongoing development tracks simultaneously while still allowing clean releases and steady hotfixes. It also makes it easy to show any unfinished work to customers. 


## Main branches


### Develop
- Branch for showing new features  to customer etc.
- Deployed to dev server
- Feature / hotfix branches can be merged here anytime for testing purposes
- This branch can be reset to main anytime, so no code living only in this branch will end up in production
- It is also possible to skip this branch if it is possible to use environment per branch

### Main
- Integration / QA for next release
- Deployed to stage server
- Base branch for all new features

### Production
- Production
- Always equal to the actual code currently running on production servers

## Development workflow

### New feature
- Create new branch from main
  - feature/#[issue/ticket id]-issueTitle (where # is C for extra/cont dev cases and I for incidents)
- Merge to develop for testing / acceptance
- Merge to main once finished /accepted (or create merge / review request)

### Epic features
Sometimes there might be bigger project that needs to be developed separately and where different features are dependant from each other
- Create new epic branch from main epic/[EpicName]
- Create all the feature branches that belong to this project from the epic branch feature/[EpicName]/[featureInThisEpic]
- Merge feature branches to epic branch, then epic branch to develop for testing
- If needed epic branch can be reset to main to clean up experimental features
- Only delete feature branches after the epic branch is accepted to main
- For release treat epic branch as any other feature branch 

### Hotfix
- Create new branch from production
- Hotfix/#[ticket id]-issueTitle (where # is C for extra/cont dev cases and I for issues, so usually I)
- Merge to develop for testing
- Merge to production for release (or create merge / review request)
- Rebase main to production

### Release
- Merge all accepted / finished feature branches to main (that are not yet merged)
- Cleanup finished feature branches
- Run tests on main
- Tag new release
- Merge to production

## Examples

Create a new feature and push it to develop for testing
```
git checkout main
git pull origin main
git checkout -b feature/c1234-adding_this
git add
git commit [... reiterate as many time as needed]
git push origin feature/c1234-adding_this
git checkout develop
git pull origin develop
git merge --no-ff feature/c1234-adding_this
git push origin develop
```
Create a hotfix
```
git checkout production
git pull origin production
git checkout -b hotfix/111-fix
git add
git commit [... reiterate as many time as needed]
git push origin hotfix/111-fix
git checkout develop
git pull origin develop
git merge --no-ff hotfix/111-fix
git push origin develop
[test again]
git checkout production
git pull origin production
git merge --no-ff hotfix/111-fix
git push origin production
git rebase production main
git push origin main
```
Publish a feature to main for the pre-release
```
git checkout main
git pull origin main
git merge --no-ff feature/c1234-adding_this
git push origin main
```

Push main to production
```
git checkout production
git pull origin production
git merge --no-ff main
git tag -a mytag -m “mytag”
git push origin production
git push --tags
```

### Diagrams

![](https://raw.githubusercontent.com/wunderio/wunderflow/main/img/WunderFlow1.png)
![](https://raw.githubusercontent.com/wunderio/wunderflow/main/img/WunderFlow_epic1.png)
