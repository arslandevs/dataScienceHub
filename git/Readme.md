Git Commands
============

<p align='right'>
<img src='https://git-scm.com/images/logos/downloads/Git-Icon-1788C.png?raw=true' alt='drawing' width='100' height='100'/>
</p>


## **A list of commonly used Git commands**

### **Getting & Creating Projects**

| Command | Description |
| ------- | ----------- |
| `git init` | Initialize a local Git repository |
| `git clone ssh://git@github.com/[username]/[repository-name].git` | Create a local copy of a remote repository |

### **Basic Snapshotting**

| Command | Description |
| ------- | ----------- |
| `git status` | Check status |
| `git add [file-name.txt]` | Add a file to the staging area |
| `git add -A` | Add all new and changed files to the staging area |
| `git commit -m "[commit message]"` | Commit changes |
| `git rm -r [file-name.txt]` | Remove a file (or folder) |

### **Branching & Merging**

| Command | Description |
| ------- | ----------- |
| `git branch` | List branches (the asterisk denotes the current branch) |
| `git branch -a` | List all branches (local and remote) |
| `git branch [branch name]` | Create a new branch |
| `git branch -d [branch name]` | Delete a branch |
| `git checkout -b [branch name]` | Create a new branch and switch to it |
| `git checkout -b [branch name] origin/[branch name]` | Clone a remote branch and switch to it |
| `git branch -m [old branch name] [new branch name]` | Rename a local branch |
| `git checkout [branch name]` | Switch to a branch |
| `git checkout -` | Switch to the branch last checked out |
| `git checkout -- [file-name.txt]` | Discard changes to a file |
| `git merge [branch name]` | Merge a branch into the active branch |
| `git merge [source branch] [target branch]` | Merge a branch into a target branch |
| `git stash` | Stash changes in a dirty working directory |
| `git stash clear` | Remove all stashed entries |
| `git branch -vv`| gives the tracking branch|


### **Sharing & Updating Projects**

| Command | Description |
| ------- | ----------- |
| `git push origin [branch name]` | Push a branch to your remote repository |
| `git push -u origin [branch name]` | Push changes to remote repository (and remember the branch) |
| `git push` | Push changes to remote repository (remembered branch) |
| `git pull` | Update local repository to the newest commit |
| `git pull [remote] [branch name]` | Pull changes from remote repository |
| `git pull [remote] [branch] --allow-unrelated-histories` | Git lets you merge unrelated branches |
| `git fetch [remote] [branch-name]` | fetches changes to see what others are working on. |

### **Remotes**
| Command | Description |
| ------- | ----------- |
| `git remote add origin ssh://git@github.com/[username]/[repository-name].git` | Add a remote repository |
| `git remote set-url origin ssh://git@github.com/[username]/[repository-name].git` | Set a repository's origin branch to SSH |
| `git remote add [remote name] [remote url]` | add a remote with URL |
| `git remote --v` | list all remotes |
| `git push [remote] --delete [branch name]` | Delete a remote branch |
| `git remote rm [remote]`| Delete remote from a local repository|
| `git remote show origin`| gives remote details|
| `git branch -u upstream/foo foo`| make local branch track remote branch | 

### **Inspection & Comparison**

| Command | Description |
| ------- | ----------- |
| `git log` | View changes |
| `git log --summary` | View changes (detailed) |
| `git log --oneline` | View changes (briefly) |
| `git diff [source branch] [target branch]` | Preview changes before merging |


--------------------------------
## **Adding ssh private key to git agent in windows**
1. Open git bash (Use the Windows search. To find it, type "git bash")
2. Type cd ~/.ssh. This will take you to the root directory for Git (Likely C:\Users\[YOUR-USER-NAME]\.ssh\ on Windows)
3. Within the .ssh folder, there should be these two files: **id_rsa** and **id_rsa.pub**.
4. To create the SSH keys, type ssh-keygen -t rsa -C "your_email@example.com". This will create both id_rsa and id_rsa.pub files.
5. Now, go and open id_rsa.pub in your favorite text editor.
6. Copy the contents--exactly as it appears, with no extra spaces or lines--of id_rsa.pub and paste it into GitHub and/or BitBucket under the Account Settings > SSH Keys

## **Auto-launching ssh-agent on Git for Windows**
You can run `ssh-agent` automatically when you open bash or Git shell. Copy the following lines and paste them into your `~/.profile` or `~/.bashrc` file in Git shell:
```bash
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env
```
If your private key is not stored in one of the default locations (like `~/.ssh/id_rsa`), you'll need to tell your SSH authentication agent where to find it. To add your key to ssh-agent, type `ssh-add ~/path/to/my_key`.
