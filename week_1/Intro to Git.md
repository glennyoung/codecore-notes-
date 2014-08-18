# Introduction to Git & Github

[Try Git](http://try.github.io/levels/1/challenges/1) | 
[Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#links) | 
[Pro Git](http://git-scm.com/book) | 
[Git Config](http://cheat.errtheblog.com/s/git)  
  
__Getting Started__  
In the terminal start by creating a new directory, then run `git init` to initialize a git repository  
```bash
mkdir my_new_directory
ck my_new_directory
git init
```  
Now that we have initialized a git repo, we can add files, and make edits, while trackin changes along the way. We can also revert back to previous 'commits' and merge branches. We'll go into detail about these terms mean and do shortly.  
  
For this example, let's create a static website with a couple of pages, stylesheets, javascripts, and maybe even some fonts.    
  
  __Tracking Changes__  
Create a file called index from the command line, then check its git status.  
```bash
touch index.html  
git status  
```  
You should notice that index.html is in 'untracked files'. In order to add this file to be tracked, we can run `git add index.html`. If you have changed several files and want to add them all, you can add all files with `git add .`  
  
When we add files, we say they are _staged_ or have been _added to the staging area_. Add the index to the staging area and check the status again.  
```bash
git add index.html
git status
```   
You should now see that index.html a _change to be committed_. So far, we have initialized a git repository, created a file, and added it to be staged. We can now 'commit' to make a saved point we can revert back to later, if we wish. When we create a git commit, we should give it a message (preferably a meaningful message).  
```bash
git commit -m "Add index file for site homepage"
```  
Now if we try `git status` we can see there is nothing to commit. Try `git log` to see a log of the commits in this repo. This is useful for git commands such as [revert, reset, and checkout](http://schacon.github.io/gitbook/4_undoing_in_git_-_reset,_checkout_and_revert.html).  
  
__Build Out the Site__  
Open up index.html in your favorite editor ([sublime](https://www.sublimetext.com/docs/2/osx_command_line.html), [etc.](http://www.vim.org/)), and fill in some [valid HTML](http://validator.w3.org/).  
**note**: Open the file with Chrome from the command line, using `open -a "google chrome" index.html`  
  
Here, I've added a link to a stylesheet. I've also omitted some [optional tags](https://google-styleguide.googlecode.com/svn/trunk/htmlcssguide.xml); feel free to include them, if it's more comfortable.
```html
<!DOCTYPE html>

<meta charset="UTF-8">

<title>Drew's Site</title>

<link rel="stylesheet" src="assets/stylesheets/styles.css">

<h1>Drew's Site</h1>

<p>This is my site, yo!
```  
  
After saving, run `git status` and see the changes made are not yet staged for commit. `git add .` will add all changes to the staging area. Commit changes that have been added to the staging area, then check the status and the logs.  
```bash
git commit -m "Add content to homepage"
git status
git log
``` 
  
__Branches__  
When we want to make changes, rather than make them on the main working copy (master), we can create a new branch. It's a good idea to create a new branch, make some changes, merge the changes into master, then delete the branch, rather than continue working on a branch that you have already merged. Remember to commit often. This will make much more sense as it sinks in, and we will cover it in depth shortly.  

`git branch` will display the local branches, with an asterisk on the current branch.  
  
Create a new branch called *my_styles*  
```bash  
git branch my_styles  
git checkout my_styles  
```  
**Note**:these two steps can be accomplished in one command, `git checkout -b my_styles`  
  
Make a directory called assets that contains a styles directory. Then create a styles.css file inside the styles directory.  
```bash
mkdir assets  
mkdir assets/stylesheets
touch assets/stylesheets/styles.css
```  
Open the styles.css in your preferred editor, and add some styles for your page.  
```css
body {
  color: #888;
}

h1 {
  color: #cee;
  text-shadow: 2px 2px #444;
}
```  
Add all the files to the staging area, check git status, and make a commit.  
```bash
git add .
git status
git commit -m "Add stylesheet and assets directories"
```  
  
You should now have 3 commits in your `git log`, and be starting to get comfortable with adding files and making commits.  
  
__Merging Branches__  
Let's move back to the master branch then list the folders and files in our working directory  
```bash
git checkout master
ls
```  
You may notice that there is only 1 file: index.html. Try going back to the *my_styles* branch and listing the files  
```bash
git checkout my_styles
ls
```  
Now, you should notice the assets folder. The changes we made on the *my_styles* branch have not yet been merged to *master*. Once we're comfortable with our changes, and don't feel anything is broken, we can switch back to *master* and merge in the changes from *my_styles*

```bash
git checkout master
git merge my_styles
```  
This should give some output like so  
```bash
$ git merge my_styles
Updating bea8dd6..60197f1
Fast-forward
 assets/stylesheets/styles.css | 8 ++++++++
 index.html                    | 4 ++--
 2 files changed, 10 insertions(+), 2 deletions(-)
 create mode 100644 assets/stylesheets/styles.css
```  
It looks like there were no conflicts, and everything is working fine, so we can delete the branch *my_styles*  
```bash
git branch -d my_styles
```  
__Resolving Merge Conflicts__  
Sometimes, when we merge there are conflicts. We should always resolve the conflicts locally, before making a pull request for our remote repositories (more on this, when we cover Github!)  
  
Make a new branch called *conflict_styles* and switch to it  
```bash
git checkout -b conflict_styles
```  
Open up assets/stylesheets/styles.css in your preferred editor, change the body *color* to *background-color*, save it, add the changes to the git staging area, and make a commit  
```css
/* assets/stylesheets/styles.css */
body {
  background-color: blue;
}
```  
```bash
git add .
git commit -m "Change body background color to blue"
```  
Checkout the master branch `git checkout master`, and merge conflict_styles  
```bash
git checkout master
git merge conflict_styles
```  
This should give some output warning of *CONFLICT*  
```bash
Auto-merging assets/stylesheets/styles.css
CONFLICT (add/add): Merge conflict in assets/stylesheets/styles.css
Automatic merge failed; fix conflicts and then commit the result.
```  
Open up the styles.css in your editor and notice how the conflicts are indicated  
```css
body{
<<<<<<< HEAD
  color: #888;
=======
  background-color: blue;
>>>>>>> conflict_styles
}

h1 {
  color: #cee;
  text-shadow: 2px 2px #444;
}
```  
This gives us the opportunity to review the conflicts and decide what should be done. You can see the HEAD shows what is currently in the branch you are on, and bellow the ======= it shows the changes trying to be merged in. Perhaps, in this case, we want the body to have color *and* background-color defined in the stylesheet.  
```css
body{
  color: #888;
  background-color: blue;
}

h1 {
  color: #cee;
  text-shadow: 2px 2px #444;
}
```  
Correct the conflict as appropriate, save the file, add it to the staging area, and make a commit. 
```bash
git add .
git commit -m "Merge conflict_styles"
```  
You should now have a sense of why it's important to commit often (to prevent an abunadance of conflicts!), and it may be starting to make sense that we should always be up to date with *master*, so we do not overwrite newer revisions with code that was current when we created our branch. If this isn't all clear yet, don't worry! It will make much more sense very soon.  

