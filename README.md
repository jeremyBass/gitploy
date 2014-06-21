gitploy
=======

<pre>
  Usage :  gitploy [<b><i>&gt;git_args&lt;</i></b>] [options]

  Command Examples:

    $>_ gitploy <b>install</b>
                => Install gitploy itself
    $>_ gitploy <b>init</b>
                => Initialize .gitploy/ folder
    $>_ gitploy <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => Install a module by cloning specified git repository
    $>_ gitploy <b>clone</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => Install gitploy itself
    $>_ gitploy <b>ls</b>
                => List installed modules
    $>_ gitploy <b>rm</b> <b><i>&gt;module&lt;</i></b>
                => Remove specified module
    $>_ gitploy <b>up</b> <b><i>&gt;module&lt;</i></b>
                => Update specified module
    $>_ gitploy <b>re</b> <b><i>&gt;module&lt;</i></b>
                => Refresh files of specified module
    $>_ gitploy <b>info</b> <b><i>&gt;module&lt;</i></b>
                => Show information about a specific module
    $>_ gitploy <b>files</b> <b><i>&gt;module&lt;</i></b>
                => List deployed files of specified module
    $>_ gitploy <b>proxy</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;git_args&lt;</i></b>
                => Run git command into specified module

                
PLACE HOLDER EXAMPLE VALUES:
<b><i>&gt;repository&lt;</i></b>  ==> https://github.com/account/repository.git
<b><i>&gt;module&lt;</i></b>      ==> repository_name
<b><i>&gt;git_args&lt;</i></b>    ==> reset --hard upstream/master

  Options:
  -v   Show gitploy version
  
  -h   Show this help
  
  -d   (Dry run) Dry run mode (show what would be done)
  
  -i   (Include) Only deploys items that match the filters
       EX:
       $>_ gitploy <b>-i lib/ -i foo/:bar/</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will deploy only lib/ (to lib/) and foo/ (to bar/)
                
  -c   (Callback) Point to an action file with in the repo.  If none set, default is ` installer `
       EX:
       $>_ gitploy <b>-a</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will add the repo, then look for the installer file and run it
                                
  -e   (Exclude) Filters out listed items.  May use regex syntax
       EX:
       $>_ gitploy <b>-e lib/tests/ -e *.txt</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will exclude both directory lib/tests/ and file lib/README.txt
                
  -b   (Branch) Specify a repository branch (only for add command)
       EX:
       $>_ gitploy <b>-b 1.0-stable</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will checkout 1.0-stable branch of 
                    specified repository
                
  -t   (Tag) Specify a repository tag (only for add and update command also <b>overpowers -b</b> )
        Note: a value of `<b>latest</b>` will use the latest tag released
       EX:
       $>_ gitploy <b>-t 1.2.0</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will checkout 1.2.0 tag of specified repository
                
  -f   (Folder root) Set the folder in the repo to root the tracking from. 
       EX:
       $>_ gitploy <b>-f</b> foo/Folder <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will checkout specified repository but only starting at that folder in the repo

  -r   (Reqursive) Move through the modules tracked reqursively
       EX:
       $>_ gitploy rm <b>-r</b>
                => will remove all repositories logged
                   <b>NOTE::</b> all repo removals will be meet with a prompt
       
  -u   (Unattended) This flag will accept all promots.  Use wisely as 
                    unattended in this form, `$>_ rm -ru`, would be wiping
                    all logged repos
       EX:
       $>_ gitploy rm <b>-u</b> <b><i>&gt;module&lt;</i></b>
                => will remove named repository <b> 
                   WITHOUT a prompt<b>
       
  -q   (Quite) If you must hide the verbose output,
                this flag will block all stdout
       EX:
       $>_ gitploy <b>-q</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will add a new repository with out any stdout messaging
 
                
  -w   (Wall Broadcasting) Some times it's important that you
                            don't have others doing anything 
                            while you upgrade which wall will let
                            them know you doing something, the flag stop that
       EX:
       $>_ gitploy <b>-w</b> <b><i>&gt;module&lt;</i></b> </b><i>&gt;repository&lt;</i></b>
                => will add a new repository and prevent 
                    everyone that is logged in knowing about it       
</pre>




##using ssh for private repos

1. Start by making sure your server is set up to make a ssh connectiong over your user.  look to https://help.github.com/articles/generating-ssh-keys
2. follow the directions here https://developer.github.com/guides/managing-deploy-keys/#deploy-keys
3. After these first step then you are ready to star managing your private repos.  To get your repo in run something like 
	
	```shell
	gitploy repoName "git@github.com:username/repo.git"
	```
	
	***NOTE***
	If it is the first time you have made the connection (which if you just got done with the first two step it will be), you must run the first repo addition like this
	
	```shell
	gitploy clone repoName "git@github.com:username/repo.git"
	```
	
	Why you must run the fist one with the explict `clone` action is that you will be asked to finalize the connection.  This will look something like
	
	```shell 
	RSA key fingerprint is 16:27:23:43:d4:eb:27:23:43:d4:eb:xx:xx:xx:df:a6:48.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'github.com,192.255.255.255' (RSA) to the list of known hosts.
	Enter passphrase for key '/root/.ssh/id_rsa':
	
	```
4. add the user so the passphase is only needed to be entered once

	```shell
	ssh-agent bash
	ssh-add 
	Enter passphrase for /root/.ssh/id_rsa:
	```

##Moving from tag to branch
This is no issue as you just need to run an update with the `-b` option.  For example

```shell
gitploy up -b master repoName
```

If you were on a tag then you would have been moved off this tag and to the head of the repo under the branch of `master` (in this example)


##Future Features

1. To be able to accept a git push deployment 
2. be able to have hooks and events to monitor for changes (still spec'in the out)

Please request any features and feel free to send a pull request




Inspiration for this script was derivate from

https://github.com/saltstack/salt-bootstrap

https://github.com/colinmollenhour/modman

https://github.com/jreinke/modgit

https://github.com/composer/composer

There are ideas that echo from all points, but the goal is that one can simply managed repos that lay over each other.  As long as you will not want to make a pull request on the files, you can think of this as a sudo git.  You get dry runs, compare, and even a git proxy.  Some of the reference projects are geared to Magento, but this is not so restrictive.  The aim is to be able to handle any form of deployment strategy