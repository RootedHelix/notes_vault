Related to [[programming]], [[Git]], 
#gh #git

create a new repo from inside your current project folder
```shell
gh repo create repo-name --source=. --remote=origin --push --private
``` 
- or you can choose public

- use `git init`


create a new empty repo under your account: 
`gh repo create repo-name`

create a private repo
`gh repo create repo-name --private`

create a public repo :
`gh repo create repo-name --public`