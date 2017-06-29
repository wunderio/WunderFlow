#Git WunderFlow

##Description
WunderFlow is a git workflow that tries to make it easier to have multiple ongoing development tracks simultaneously while still allowing clean releases and steady hotfixes. It also makes it easy to show any unfinished work to customers. 

##Main branches

###Develop
- Branch for showing new features  to customer etc.
- deployed to dev server
- Feature / hotfix branches can be merged here anytime for testing purposes
- This branch can be reset to master anytime, so no code living only in this branch will end up in production
- It is also possible to skip this branch if it is possible to use environment per branch

###Master
- Integration / QA for next release
- Deployed to stage server
- Base branch for all new features

###Production
- Production
- Always equal to the actual code currently running on production servers


##Development workflow

###New feature
- create new branch from master 
  - feature/#[issue/ticket id]-issueTitle (where # is C for extra/cont dev cases and I for incidents)
- merge to develop for testing / acceptance
- merge to master once finished /accepted (or create merge / review request)

###Epic features
Sometimes there might be bigger project that needs to be developed separately and where different features are dependant from each other
- create new epic branch from master epic/[EpicName]
- create all the feature branches that belong to this project from the epic branch feature/[EpicName]/[featureInThisEpic]
- merge feature branches to epic branch, then epic branch to develop for testing
- if needed epic branch can be reset to master to clean up experimental features
- only delete feature branches after the epic branch is accepted to master
- for release treat epic branch as any other feature branch 

###Hotfix
- create new branch from production
- hotfix/#[ticket id]-issueTitle (where # is C for extra/cont dev cases and I for issues, so usually I)
- merge to develop for testing
- merge to production for release (or create merge / review request)
- rebase master to production


###Release
- Merge all accepted / finished feature branches to master (that are not yet merged)
- Cleanup finished feature branches
- Run tests on master
- tag new release
- Merge to production

##examples

create a new feature and push it to develop for testing
```
git checkout master
git pull origin master
git checkout -b feature/c1234-adding_this
git add
git commit [... reiterate as many time as needed]
git push origin feature/c1234-adding_this
git checkout develop
git pull origin develop
git merge --no-ff feature/c1234-adding_this
git push origin develop
```
create a hotfix
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
git rebase production master
git push origin master
```
publish a feature to master for the pre-release
```
git checkout master
git pull origin master
git merge --no-ff feature/c1234-adding_this
git push origin master
```

push master to production
```
git checkout production
git pull origin production
git merge --no-ff master
git tag -a mytag -m “mytag”
git push origin production
git push --tags
```

### Diagrams

![](https://raw.githubusercontent.com/wunderkraut/wunderflow/master/img/WunderFlow1.png)
![](https://raw.githubusercontent.com/wunderkraut/wunderflow/master/img/WunderFlow_epic1.png)
