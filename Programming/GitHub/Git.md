Related to [[Git]], [[programming]], [[GH]]
#git

`git remote set-url origin git@github-work:organization-or-username/repo-name.git`


## When you have commits on the local branch and you need the new changes from the remote branch
`git pull --rebase origin main`

## Clone Repo

`git clone <repository/url>`

## Checks for Existing Git Repo in File

 Type: find ______ XXX -type d -name ".git"
- XXX is the target directory


if [ -d ".git" ]; 
	then echo "This is a Git repository." 
else 
	echo "This is not a Git repository."
fi

- Important step is to change the remote origin of push and fetch so as not to push to the wrong repository



## Check, Change, or Add The Remote Repository That Git is Connected To

1. Navigate to desired directory
	 cd /user/desired/place
2. Check what Github Repo Git is connected to 
	 git remote -v
   
   3. If Change is wanted (Should Verify Change After See Step #2)
	    git remote set-url origin https://new-url-of-your-repo.git

		- Change the url to new desired Repo url
4. Add new connection
		git remote add origin "-repository-url-"


## Removing git repos 

1. Navigate to desired directory:
	'cd /Users/yourusername/Projects/my-repo'
2. Force remove files:
	rm -rf .git
3.  Check Status, should throw fatal error message since .git file no longer exists
	 'git status'


## Staging - Committing - Push

<h1>To stage: </h1>
	git add < file>
	git add .

<h1>Commit: </h1>
	 git commit -m ""         //-m is a message

<h1> Push: </h1>
OR
git push origin developLocal:Develop

To push to a remote repository branch named `Develop`, use the following command:

`git push origin Develop`

If `Develop` is the default branch or a common branch you often push to, you can set it as your default push branch:

`git push -u origin Develop`

Using `-u` sets `Develop` as the upstream branch for your current branch, so in future pushes, you can simply run:

`git push`

And it will push to `Develop` by default.


-=[ IF THE LOACCAL BRANCH AND REMOTE BRANCH NAMES ARE DIFFERENT] =- 

git push origin HEAD:develop

The upstream branch of your current branch does not match
the name of your current branch.  To push to the upstream branch
on the remote, use

## Updating 'Git' Repo 

- Pull From GitHub Root Branch:
	 git pull origin name 


## Pulling Git Repo

`git pull <remote> <branch>`

Replace `<remote>` with the name of the remote repository (e.g., `origin`) and `<branch>` with the branch you want to pull from (e.g., `main` or `develop`).

- If you want to see the specific changes made by the pull (especially the merge commit), you can check the commit history with diffs:


`git log -p`

The command `git config pull.rebase false` configures Git to use the merge strategy by default when you run `git pull`. Here’s a breakdown of what this setting does:

### Understanding `git config pull.rebase false`

1. **Setting Explanation**:
    
    - **`git config pull.rebase false`**: This sets the `pull.rebase` configuration option to `false`, which means that when you perform a `git pull`, Git will use the merge strategy to integrate changes from the remote branch into your local branch.
2. **Effect on `git pull`**:
    
    - When `pull.rebase` is set to `false`, `git pull` will perform a `git fetch` followed by a `git merge`. This means Git will fetch the changes from the remote branch and then create a merge commit to combine these changes with your local commits.
    - The merge commit will have two parent commits: one from the current branch and one from the remote branch. This results in a non-linear history that shows a clear record of when changes from different sources were merged.

## Creating git ignore file

1.  **Navigate to the Root Directory**

Use the `cd` command to navigate to the root directory of your Git repository. This is the directory that contains the `.git`folder.

bash

Copy code

`cd /path/to/your/repository`


2. **Create or Edit the `.gitignore` File**

**Using Terminal Commands**

You can use the `touch` command to create a new `.gitignore` file if it doesn’t already exist:

bash

Copy code

`touch .gitignore`

Alternatively, if you want to edit the `.gitignore` file or add specific entries to it, you can use a text editor. For example, using `nano`:

bash

Copy code

`nano .gitignore`

In the `nano` editor, you can add patterns to ignore specific files or directories. For example:

plaintext

Copy code

``# Ignore the `msc-api` directory msc-api/  # Ignore all `.log` files *.log  # Ignore the `node_modules` directory node_modules/``

To save and exit `nano`, press `Ctrl + X`, then `Y` (to confirm saving), and `Enter`.

##  Showing Connected Remote and Local Branch
	
	 git branch -vv
		show the remote branch that you are connected to in
		addition to the current branch

	git remote show origin 
	
	 Provides detailed information about the remote repository.

	Use `git branch -r` to list all remote branches names.

		Show what LOCAL branch you are connected to:
	
	git branch


## Switch to a different Remote Branch

Git does not allow you to directly check out a remote branch (because remote branches are not local branches yet). Instead, you need to **create a local tracking branch** for the remote branch. This links your local branch to the remote one. YOU HAVE TO CREATE NEW LOCAL BRANCH FOR DIFFERENT REMOTE BRANCHES

To switch to a remote branch and create a local tracking branch for it, use the following command
newer `git switch` command:

`git switch -c <local-branch-name> origin/<remote-branch-name>`

## Update Main Branch to Development Branch State

To update the `main` branch to the latest version of the `development` branch, follow these steps:

### Using Merge (Preserves History)

1. Switch to `main`:
			git checkout main  (or git switch main)

2. Ensure it's up to date:
    
    ```sh
    git pull origin main
    ```
    
3. Merge `development` into `main`:
    
    ```sh
    git merge development
    ```
    
4. Push changes:
    
    ```sh
    git push origin main
    ```
    

---

### **Using Rebase (Rewrites History)**

If you want a linear history:

1. Switch to `main`:
    
    ```sh
    git checkout main
    ```
    
2. Ensure it's up to date:
    
    ```sh
    git pull origin main
    ```
    
3. Rebase `development` onto `main`:
    
    ```sh
    git rebase development
    ```
    
4. Push changes (force push may be required if rebasing changes history):
    
    ```sh
    git push origin main --force  # Be careful with --force
    ```
    

---

### **Using Hard Reset (Overwrites `main`)**

If you want `main` to exactly match `development`, discarding any differences:

1. Switch to `main`:
    
    ```sh
    git checkout main
    ```
    
2. Reset `main` to `development`:
    
    ```sh
    git reset --hard development
    ```
    
3. Push changes (force push required to update remote history):
    
    ```sh
    git push origin main --force
    ```
    

⚠️ **Warning:** Hard reset and force push can rewrite history, affecting collaborators.

Which method you choose depends on your workflow and whether you want to keep history (`merge`), keep a linear history (`rebase`), or completely overwrite (`reset --hard`).


## Making Doxygen CrossCompatible (WINDOWS and MAC) in git using .gitattributes

Since you haven't committed anything yet, this is the perfect time to configure Git properly for **cross-platform development** (PC & Mac). Here's the best setup to ensure consistency:

---

### **1️⃣ Set Git to Use LF Line Endings for Cross-Platform Compatibility**

Since you're working on **both Windows and macOS**, you should **prevent Git from automatically converting line endings** by setting:

```sh
git config --global core.autocrlf input
```

🔹 **Why?**

- This **keeps LF (`\n`) line endings in the repo** (which is the standard for most development).
- On macOS/Linux, nothing changes.
- On Windows, Git **converts CRLF to LF when committing** but **does NOT convert LF to CRLF on checkout** (avoiding unnecessary warnings).

---

### **2️⃣ (Recommended) Use a `.gitattributes` File to Enforce LF**

To **ensure your project always uses LF**, create a `.gitattributes` file in the root of your repo and add:

```gitattributes
# Force LF for text-based files
* text=auto
*.c text eol=lf
*.h text eol=lf
Doxyfile text eol=lf
*.sh text eol=lf
```

🔹 **Why?**

- This forces Git to **always use LF** for source code files.
- Prevents accidental CRLF commits from Windows.

---

### **3️⃣ Verify Line Endings Before Committing**

Run:

```sh
git add --dry-run .
```

If you see warnings about LF/CRLF conversion, re-check your settings.

---

### **4️⃣ Optional: Re-clone If Needed**

If you've already cloned the repo and suspect line ending issues, you might want to **re-clone it after setting `core.autocrlf input`**:

```sh
# Backup your work first!
rm -rf my-repo
git clone <repo-url>
```

---

### **Final Setup Summary**

|Setting|Command|Purpose|
|---|---|---|
|**Set LF line endings globally**|`git config --global core.autocrlf input`|Keeps LF in repo, avoids CRLF issues|
|**Enforce LF for specific files**|Add `.gitattributes` file|Prevents inconsistent line endings|
|**Verify before committing**|`git add --dry-run .`|Detects line-ending changes|

---

### **🚀 You’re All Set!**

Now, your project will **always use LF**, no matter where you edit it. Let me know if you need help! 😊

## Create the local `develop` branch and set it to track the remote one

git checkout -b develop origin/develop

## GIT FLOW

1. You should always pull before pushing
2. when looking to merge branches, merge the local branches first then push up to the remote branches

# Git Stash

Delete stash from stash list:
`git stash pop stash@{n}`
