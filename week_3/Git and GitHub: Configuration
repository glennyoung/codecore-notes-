# Git and GitHub: Configuration  
[.bashrc](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html) | [Settings the PS1](http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/setps.html) |  
Global and Local settings allow us to customize particular
repositories.  

List all settings  
```bash
git config --list
```  
To view your global settings for many tools can be found in your system's home directory. The following commands in a bash terminal will perform the same thing.   
```bash
vim ~/.gitconfig
vim $HOME/.gitconfig
```  
Example .gitconfig  
```bash
[color]
  ui = true
[color "branch"]
  current = green reverse
  local = yellow
  remote = green
[color "diff"]
  meta = yellow bold
  frag = magenta bold
  old = red bold
  new = green bold
[color "status"]
  added = yellow
  changed = green
  untracked = cyan

[user]
  name = drew
  email = 123and@gmail.com
[push]
  default = matching
[core]
  autocrlf = input
```  
Dotfiles are files beginning with a dot (.), which are 'hidden files' and are generally used for customizing settings.  
  
We use terminal a lot when developing. Bash (the Bourne Again Shell) is a shell we use in the
terminal, and a [scripting language](http://www.ibm.com/developerworks/library/l-bash/) (like Ruby). There are [alternative shells](http://www.interworx.com/community/alternative-shells-for-linux/), such as zsh, ksh.  
  
Mac OS X runs its Terminal.app a little differently from most other GUI
terminals, and a [recommended
setting](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html) for your .bash_profile is to add
the following  
```bash
if [ -f ~/.bashrc ]; then
   source ~/.bashrc
fi
```  
This script is pretty easy to undertand, and basically says "If there is
a file in the home directory .bashrc, then use it as the source."  
  
There are some intended [differences between .bashrc and
.bash_profile](http://superuser.com/questions/183870/difference-between-bashrc-and-bash-profile),
however for the scope of this course, we will only be using the .bashrc
file for setting alsiases, setting up our default editors, and system
output colors, etc.  
  
## Setting the PS1  
We can customize the command prompt in Bash using PS1="something". To
see the current code behind your prompt, try `echo $PS1`.  
  
You can make your command prompt look different for the current session
with `PS1="Hello: "`. We can set up our command line to give more
[meaningful
information](http://www.thegeekstuff.com/2008/09/bash-shell-ps1-10-examples-to-make-your-linux-prompt-like-angelina-jolie/). For example, if we wish to show the current git branch we are on, and its status, we can use a script like [.git_svn_bash_prompt.sh](https://gist.github.com/woods/31967) from Scott Woods, and customize it a little to suit our needs.  
```bash
#!/bin/bash
#
# DESCRIPTION:
#
#   Set the bash prompt according to:
#    * the branch/status of the current git repository
#    * the branch of the current subversion repository
#    * the return value of the previous command
# 
# USAGE:
#
#   1. Save this file as ~/.git_svn_bash_prompt
#   2. Add the following line to the end of your ~/.profile or ~/.bash_profile:
#        . ~/.git_svn_bash_prompt
#
# AUTHOR:
# 
#   Scott Woods <scott@westarete.com>
#   West Arete Computing
#
#   Based on work by halbtuerke and lakiolen.
#
#   http://gist.github.com/31967
 
 
# The various escape codes that we can use to color our prompt.
        RED="\[\033[0;31m\]"
     YELLOW="\[\033[0;33m\]"
      GREEN="\[\033[0;32m\]"
       BLUE="\[\033[0;34m\]"
  LIGHT_RED="\[\033[1;31m\]"
LIGHT_GREEN="\[\033[1;32m\]"
      WHITE="\[\033[1;37m\]"
 LIGHT_GRAY="\[\033[0;37m\]"
 COLOR_NONE="\[\e[0m\]"
 
# Detect whether the current directory is a git repository.
function is_git_repository {
  git branch > /dev/null 2>&1
}
 
# Detect whether the current directory is a subversion repository.
function is_svn_repository {
  test -d .svn
}
 
# Determine the branch/state information for this git repository.
function set_git_branch {
  # Capture the output of the "git status" command.
  git_status="$(git status 2> /dev/null)"
 
  # Set color based on clean/staged/dirty.
  if [[ ${git_status} =~ "working directory clean" ]]; then
    state="${GREEN}"
  elif [[ ${git_status} =~ "Changes to be committed" ]]; then
    state="${YELLOW}"
  else
    state="${RED}"
  fi
  
  # Set arrow icon based on status against remote.
  remote_pattern="# Your branch is (.*) of"
  if [[ ${git_status} =~ ${remote_pattern} ]]; then
    if [[ ${BASH_REMATCH[1]} == "ahead" ]]; then
      remote="↑"
    else
      remote="↓"
    fi
  else
    remote=""
  fi
  diverge_pattern="# Your branch and (.*) have diverged"
  if [[ ${git_status} =~ ${diverge_pattern} ]]; then
    remote="↕"
  fi
  
  # Get the name of the branch.
  branch_pattern="^# On branch ([^${IFS}]*)"    
  if [[ ${git_status} =~ ${branch_pattern} ]]; then
    branch=${BASH_REMATCH[1]}
  fi
 
  # Set the final branch string.
  BRANCH="${state}(${branch})${remote}${COLOR_NONE} "
}
 
# Determine the branch information for this subversion repository. No support
# for svn status, since that needs to hit the remote repository.
function set_svn_branch {
  # Capture the output of the "git status" command.
  svn_info="$(svn info | egrep '^URL: ' 2> /dev/null)"
 
  # Get the name of the branch.
  branch_pattern="^URL: .*/(branches|tags)/([^/]+)"
  trunk_pattern="^URL: .*/trunk(/.*)?$"
  if [[ ${svn_info} =~ $branch_pattern ]]; then
    branch=${BASH_REMATCH[2]}
  elif [[ ${svn_info} =~ $trunk_pattern ]]; then
    branch='trunk'
  fi
 
  # Set the final branch string.
  BRANCH="(${branch}) "
}
 
# Return the prompt symbol to use, colorized based on the return value of the
# previous command.
function set_prompt_symbol () {
  if test $1 -eq 0 ; then
      PROMPT_SYMBOL="\$"
  else
      PROMPT_SYMBOL="${RED}\$${COLOR_NONE}"
  fi
}
 
# Set the full bash prompt.
function set_bash_prompt () {
  # Set the PROMPT_SYMBOL variable. We do this first so we don't lose the 
  # return value of the last command.
  set_prompt_symbol $?
 
  # Set the BRANCH variable.
  if is_git_repository ; then
    set_git_branch
  elif is_svn_repository ; then
    set_svn_branch
  else
    BRANCH=''
  fi
  
  # Set the bash prompt variable.
  PS1="\u@\h \w ${BRANCH}${PROMPT_SYMBOL} "
}
 
# Tell bash to execute this function just before displaying its prompt.
PROMPT_COMMAND=set_bash_prompt
```

## Git Config Settings

Global and Local config settings allow us to specify if we want to use
the same default settings on all projects (Global), or *override* those
settings locally.  
  
To view your current settings run `git config --list`. By default, you
will be using your global settings. If you haven't set them up yet, or
if you want to change them, you can set your username and email (this
will be associated with any commits you make).  
```bash
git config global u.name drew
git config global u.email drew@drewbro.com

# or locally
git config local u.name drew
git config local u.email 123and@gmail.com
```  
Similarly, to set git to a color user interface, you can run `git config
--global color.ui true`.  
  

View settings files globally `cat ~/.gitconfig` or locally
`.git/config`.  

## Add Remote Repositories  
If you already have remote repositories set up, you can see them by
running the command `git remote -v`. This will display all remote
repositories you are currently connected to. Git has a Master branch,
and by default, we call our remote repository 'origin'. However, if you
have multiple remotes, you will have different names for each.  
  
For example, if you have a remote repository on heroku, it will be
called 'heroku' rather than remote. An example list of remote repose may
look like this  
```bash
bb https://drewbro@bitbucket.org/drewbro/my_website.git (fetch)
bb https://drewbro@bitbucket.org/drewbro/my_website.git (push)
gh git@github.com:ogryzek/my_website.git (fetch)
gh git@github.com:ogryzek/ogryzek/my_website.git (push)
heroku git@heroku.com:my_website.git (fetch)
heroku git@heroku.com:my_website.git (push)
```  
  
## Push, Pull, Merge, Fetch & Rebase  
Locally, we can create a new branch and checkout to it with `git
checkout -b my_branch`. After we've made changes we are satisfied with,
we can add and commit then, then checkout to the Master branch and
merge: `git merge my_branch`.  
  
When working with remote repos, especially in collaboration with other, we often push our branch up to the
remote, then request that it be merged to the master branch.  
  
If we want to pull any changes that may have been merged with master
since we last pulled from the remote origin, we can do `git pull origin
master`.  
  
Git fetch acts a little differently. When we 'fetch' we pull the latest
changes and are able to checkout other branches, or merge. Typically, we
can do  
```bash
git fetch
git rebase origin/master
```  
***Note***  
```bash
git pull = git fetch + git merge              # against tracking upstream branch.

git pull --rebase =  git fetch + git rebase   # against tracking upstream
branch.
```  
We can use `git pull --rebase` as a shorthand for this.  
  
