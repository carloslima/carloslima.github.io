---
layout: post
title: 'Hosting your GIT repository on DreamHost'
date: 2010-02-25
comments: true
permalink: /2010/02/hosting-your-git-repository-on.html
---

A few weeks ago I needed a private remote [Git](http://git-scm.com) repository.  
Since I already have a [DreamHost](http://www.dreamhost.com) account, I didn't want to pay for [GitHub](http://github.com) just to be able to set my repository as private.  

DreamHost does not officially support Git on their Web Panel, so you must set things up manually.

There is a [Git entry on their Wiki](http://wiki.dreamhost.com/Git) but it looked a bit messy, so I decided to write a cleaner tutorial.

Keep in mind that I'm using [WebDAV](http://www.kernel.org/pub/software/scm/git/docs/howto/setup-git-server-over-http.txt).  
It allows more than one user to push to the same repository, but it's also slower.

> You can use SSH but it's a lot more complex to setup.<br>
> If that's what you want, read the [DreamHost Wiki](http://wiki.dreamhost.com/Git#Setup_Three).

## Setup a WebDAV folder
Login to [DreamHost Web Panel](https://panel.dreamhost.com/)  
Go to [Goodies -&gt; Htaccess/WebDAV](https://panel.dreamhost.com/index.cgi?tree=goodies.webdav)  
Select the domain name you want to use  
Click "Set Up A New Directory"

![DH Panel/WebDAV](/images/hosting-your-git-repository-on-dreamhost/dh-webpanel.png)

##Create the remote repository

Git cannot create the remote repository, it only operates on existing ones, so we need to create an empty repository locally and
manually upload it to DreamHost.

So, there we go, open a console and..

    carlos@ubuntu:~/dev$ mkdir blank.git
    carlos@ubuntu:~/dev$ cd blank.git/
    carlos@ubuntu:~/dev/blank.git$ git --bare init
    Initialized empty Git repository in /home/carlos/dev/blank.git/
    carlos@ubuntu:~/dev/blank.git$ touch git-daemon-export-ok
    carlos@ubuntu:~/dev/blank.git$ git --bare update-server-info
    carlos@ubuntu:~/dev/blank.git$ mv hooks/post-update.sample hooks/post-update
    carlos@ubuntu:~/dev/blank.git$

### Upload the blank repository and rename it

Use Nautilus or any other file manager that supports WebDAV to upload blank.git to Dreamhost and rename it to something
meaningful. For the purpose of this example, let's call it project.git

On Ubuntu:

![Places->Connect](/images/hosting-your-git-repository-on-dreamhost/places-connect.png)  
![Places->Connect](/images/hosting-your-git-repository-on-dreamhost/connect-to-server.png)  
![Places->Connect](/images/hosting-your-git-repository-on-dreamhost/copy-files.png)  
![Places->Connect](/images/hosting-your-git-repository-on-dreamhost/copy-files2.png)  

After it finishes uploading, rename the folder from blank.git to project.git

> When I renamed the file, I got an error message saying it failed, but refreshing showed it actually worked.

## Ready!

If you're starting fresh and all you need is a blank repository, then you're set.  
Just clone your repository and start working!

    carlos@ubuntu:~/dev$ git clone http://bob@www.example.com/git/project.git
    Initialized empty Git repository in /home/carlos/dev/project/.git/
    Password:
    warning: You appear to have cloned an empty repository.

    carlos@ubuntu:~/dev$ cd project
    carlos@ubuntu:~/dev/project$ (work, work, work)
    carlos@ubuntu:~/dev/project$ git add README
    carlos@ubuntu:~/dev/project$ git commit -m "your commit message"
    [master (root-commit) 5bbe5f6] your commit message
    0 files changed, 0 insertions(+), 0 deletions(-)
    create mode 100644 README
    carlos@ubuntu:~/dev/project$ git push origin master
    Password:
    Fetching remote heads...
    refs/
    refs/tags/
    refs/heads/
    updating 'refs/heads/master'
    from 0000000000000000000000000000000000000000
    to 5bbe5f6507fa39293bdc9674ca4ae2e0a1d2f15e
    sending 3 objects
    done
    Updating remote server info

    carlos@ubuntu:~/dev/project$ git pull
    Password:
    From http://bob@www.example.com/git/project
    * [new branch] master -&gt; origin/master
    Already up-to-date.
    carlos@ubuntu:~/dev/project (master)$

## Optional: Push your project repository to DreamHost

Otherwise, if you already have a project repository that you've been working locally, now is the time to push it to
DreamHost.

Instead of cloning the new repository, push your project there first:

    carlos@ubuntu:~/dev$ cd real_project/
    carlos@ubuntu:~/dev/real_project$ git config remote.upload.url http://bob@www.example.com/git/project.git/

      # It is important to put the last '/'; Without it, the server will send a redirect which git-http-push
      # does not (yet) understand, and git-http-push will repeat the request infinitely.
    
    carlos@ubuntu:~/dev/real_project$ git push upload master
    Password:
    Fetching remote heads...
    refs/
    refs/tags/
    refs/heads/
    updating 'refs/heads/master'
    from 0000000000000000000000000000000000000000
    to a10703d8e400ca9df1b19345975718935c083905
    sending 107 objects
    done
    Updating remote server info
    carlos@ubuntu:~/dev/real_project$

Then confirm it worked and start fresh by cloning it.

    carlos@ubuntu:~/dev/real_project$ cd ..
    carlos@ubuntu:~/dev$ git clone http://bob@www.example.com/git/project.git/
    Initialized empty Git repository in /home/carlos/dev/project/.git/
    Password:
    got a10703d8e400ca9df1b19345975718935c083905
    walk a10703d8e400ca9df1b19345975718935c083905
    got 574596c4cc435461515aa1a4c3cdd0e93af947f3
    got 067f993be7432ac27e8a6e9636dea53dcc3d8632
    got 475be0881778acd2de7404175fa323823e4d1ac0
    walk 067f993be7432ac27e8a6e9636dea53dcc3d8632
    (...)
    got b7b5d32db9dd30c9ea28434b125781eb4a3e95b2

    carlos@ubuntu:~/dev$ cd project/
    carlos@ubuntu:~/dev/project$ git log --oneline
    a10703d Adds a beautiful whitespace! :)
    067f993 Adds project description.. or sort of
    80c2e22 removes rerun.txt
    07e3cd2 Initial commit

    carlos@ubuntu:~/dev/project$ git pull
    Password:
    Already up-to-date.

    carlos@ubuntu:~/dev/project$ git push origin master
    Password:
    Fetching remote heads...
    refs/
    refs/tags/
    refs/heads/
    'refs/heads/master': up-to-date
    carlos@ubuntu:~/dev/project$

And you're good to resume working on your project :)

## Bonus: saving your WebDAV password

Now, if you're thinking that typing the WebDAV password over and over again kind of suck, you can save it so that git won't ask
you anymore.

    carlos@ubuntu:~/dev$ echo "machine www.example.com login bob password secret" >> ~/.netrc

There is only one thing to keep in mind.
When you save your password like this, you need to drop the `bob@`
from the urls.  
So, instead of referring to your repository as `http://bob@www.example.com/git/project.git/` you need to use just `http://www.example.com/git/project.git/`  
If you do this after you finished everything and cloned your repository, then git will have already saved the "wrong" url into
its config file and will keep asking you for the password.

To fix this you can either clone again using the correct url or fix git's config manually by doing:

    carlos@ubuntu:~/dev/project$ git config remote.origin.url http://www.example.com/git/project.git/
    carlos@ubuntu:~/dev/project$

Now, just for reference :)

    carlos@ubuntu:~/dev$ more /etc/issue.net
    Ubuntu 9.10
    carlos@ubuntu:~/dev$ git --version
    git version 1.6.3.3
    carlos@ubuntu:~/dev$ date
    Thu Feb 25 08:55:18 BRT 2010
    carlos@ubuntu:~/dev$
