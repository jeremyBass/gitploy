gitploy
=======

#install

If you have rights, you can run

`curl https://raw.github.com/jeremyBass/gitploy/master/gitploy | sudo sh -s -- install`

anywhere.  It will unpack itself and locate itself with in `/usr/sbin/gitploy` and set so that it is executable.  Once you have the command run from above run this to check the install

`gitploy -v`

That will check the version.  For more options read below in the option section.





##using ssh for private repos

###Option 1 - deployment keys

1. Start by making sure your server is set up to make a ssh connection from your user.  Look to https://help.github.com/articles/generating-ssh-keys for more information
2. Follow the directions here https://developer.github.com/guides/managing-deploy-keys/#deploy-keys
3. After these first step then you are ready to star managing your private repos.  To get your repo in run something like 
	
	```shell
	gitploy repoName "git@github.com:username/repo.git"
	```
	
	***NOTE***
	If it is the first time you have made the connection (which if you just got done with the first two step it will be), you must run the first repo addition like this
	
	```shell
	gitploy clone repoName "git@github.com:username/repo.git"
	```
	
	Why you must run the first one with the explicit `clone` action is that you will be asked to finalize the connection.  This will look something like
	
	```shell 
	RSA key fingerprint is 16:27:23:43:d4:eb:27:23:43:d4:eb:xx:xx:xx:df:a6:48.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'github.com,192.255.255.255' (RSA) to the list of known hosts.
	Enter passphrase for key '/root/.ssh/id_rsa':
	
	```
4. Add the user so the passphase is only needed to be entered once

	```shell
	ssh-add 
	Enter passphrase for /root/.ssh/id_rsa:
	```
	
	With the `/root/.ssh/id_rsa` make sure it's under the user you are user ie: if it was `foo.bob` user it would be `/home/foo.bob/.ssh/id_ras`.  If that doesn't exist then you'll need to create that.  To do so try this:
	
	```shell
	touch /home/foo.bob/.ssh/id_ras
	chmod 600 /home/foo.bob/.ssh/id_ras
	```
	Another tip if you are still are having to enter your passphrase each time is to set the passphase to blank.  This may seem like your breaking the security, but if your system is secure it is argued that you have already authenticated twice at the very least before you would have used the passphase.  Do this only if you are sure you know what you are doing.  Lots of chat on google about this, but do what you feel is best.

###Option 1 - ssh agent forwarding

There is only one step that needs to be done:

1. Add `ForwardAgent yes` to your `/.ssh/config` file on your local machine.  When you coonect to your production server you will have access to your git repos.  

	```shell
	Host foo.com
	  ForwardAgent yes
	  User foo.user
	  Hostname 111.111.111.111
	  IdentityFile ~/.ssh/github_rsa
	```
	
1. (optional ***if not working yet***) You most likely are going to need to `sudo -s` to do anything with the server including using `gitploy`.  In order to still have the ssh agent forwarded an edit will need to be done.  With in `/etc/sudoers` you need to add:

	```shell
	Defaults    env_keep+=SSH_AUTH_SOCK
	```
	After this when the `foo.user` logs in and then `sudo -s` you can run `ssh-add -L`  and you should see that you are now getting you ssh key alther way from your local machine to your server user that is sudoed in.
	
1. (optional ***if still not working yet***) it may be that you are getting a `ssh: connect to host github.com port 22: Connection refused` message, if so you just need to point your host to the ssl for github (443) on the server you are working on.  This can be done like this:

	```shell
	vi ~/.ssh/config
	```
	and add:
	
	```shell
	Host github.com
	  Hostname ssh.github.com
	  Port 443
	```
	

####Trouble Shooting agent forwarding
There are a few cases that the agent forwarded can be lost.  If you  `ssh-add -L` and don't see your key, then try to log off the server and log back in.   That normally will correct the issue.


##Moving from tag to branch
This is no issue as you just need to run an update with the `-b` option.  For example

```shell
gitploy up -b master repoName
```

If you were on a tag then you would have been moved off this tag and to the head of the repo under the branch of `master` (in this example)



##options

<pre>
  Usage :  gitploy [<b><i>&gt;git_args&lt;</i></b>] [options]

  Actions:
    init           => Initialize .gitploy/ folder
    ls/list        => List installed modules
    rm/remove      => Remove specified module
    up/update      => Update specified module
    re/refresh     => Refresh files of specified module
    info           => Show information about a specific module
    files          => List deployed files of specified module
    proxy          => Run git command into specified module
    install        => Install gitploy
    update_gitploy => This will update gitploy it's self


  Command Examples:
    ${__ScriptName} <module> <repository> //(note this installs the repo)
    ${__ScriptName} init  
    ${__ScriptName} ls
    ${__ScriptName} rm <module>
    ${__ScriptName} up <module>
    ${__ScriptName} re <module>
    ${__ScriptName} info <module>
    ${__ScriptName} files <module>
    ${__ScriptName} proxy <module> <git_args>
    ${__ScriptName} install
    ${__ScriptName} update_gitploy

  Options:
  -v   Show gitploy version
  -h   Show this help
  -d   (Dry run) Dry run mode (show what would be done)
  -i   (Include) Only deploys items that match the filters
  -c   (Callback) Point to an action file with in the repo.  If none set,
                  default is installer
  -e   (Exclude) Filters out listed items.  May use regex syntax
  -b   (Branch) Specify a repository branch (only for add command)
  -t   (Tag) Specify a repository tag (only for add and update command
             also overpowers -b ) 
  -z   (Reset) Reset a repo on operation.  Really only applies to updates.
               You will move from tag to branch at head release 
  -f   (Folder root) Set the folder in the repo to root the tracking from. 
           (** NOTE:: setting to false will clear the root used adn return the
                      repo back to a normal checkout ** )
  -r   (Reqursive) Move through the modules tracked reqursively
  -u   (Unattended) This flag will accept all promots.
  -q   (Quite) If you must hide the verbose output, this flag will 
               block all stdout
  -w   (Wall Broadcasting) tell anyone on the server you are about to
                           run an action
  -p   (Path installed) Where is the repo installed to from root (beta)
  -g   (user group) if set then only that user that is in that group can operate that repo  
             
</pre>

##ideas on usage

1. use [`incron`](http://inotify.aiken.cz/?section=incron&page=doc&lang=en) to watch a folder for change, and if something changes, then redeploy the orginal files.  Ex:
	```shell
	/var/stores/magento/app/Mage.php IN_MODIFY cd /var/stores/ && gitploy re MAGE
	```
	This assumes a few things, one being that you have a full copy of Magento set up as a repo and versioned in with gitploy.  See [`magento-mirror`](https://github.com/washingtonstateuniversity/magento-mirror) where you would have ran 
	
	`gitploy -t 1.9.1.0 -p magneto MAGE https://github.com/washingtonstateuniversity/magento-mirror.git`
	
	Another thing that is assumed, other then the paths, is that you have rooted the gitploy from the `/var/stores/` folder where you'd find `/var/stores/.gitploy/MAGE` which deployed to the `magento` folder.



##Future Features

 - [ ] To be able to accept a git push deployment
 - [ ] Be able to have hooks and events to monitor for changes ***(still spec'in the out but the thought is the cron jobs run `re` to ensure the code is always what it should be.  This would be done to lower malware infection scope of damage)***
 - [ ] To be able to accept a git push deployment

Please request any features and feel free to send a pull request


##Change Log

###0.4.0
**Features**

- adds check for user group if group set

**Bug Fixes**

- fixes the update to break the cache by date and time
- displays that version while updating

