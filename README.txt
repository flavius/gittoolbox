A pluggable toolbox for distributing, updating and managing all your git needs.

By default, it contains a plug-in which generates a simple blog from your
(longer) commit messages.

Installation instructions
=========================

Git toolbox can run on both your local repository, as well as on a remote
repository, if you are hosting it on your own server (and you have ssh access
to it). This means that "github pages" or similar are excluded.

You can publish your blog to github pages, as they take care of the deployment
part for you.

Assumptions:
  * you can clone and push without entering your ssh passphrase; your agent is
    running in the background already
  * if you host your blog on your own server, you can ssh into it

Follow the appropiate instructions below for your scenario.

With a new project from scratch (no commits have been made)
-----------------------------------------------------------

Clone the toolbox:

    git clone https://github.com/flavius/gittoolbox.git

Initialize your project:

    git init --template=/path/to/gittoolbox myproject
    cd myproject/

Customize `.git/config` to your needs.

If you will deploy to a server you control, the toolbox can also deploy
for you. You must tell it the path on the server too. For nginx on ArchLinux,
this can be configured as the default with:

    git config devblog.remote-deploy-path /usr/share/nginx/html

To create the repository on your server, run:

    ssh git@myserver.tld "git init --bare myproject.git"

Then set the remote:

    git remote add origin git@myserver.tld:myproject.git

The blog files live in the `blog` branch, an orphan branch unrealated to your
project. The branch can be renamed in the `.git/config` file, before you start
making your initial commit.


Make your regular commits. If they contain a body (along with a first line),
a new article will be generated based on the first line (dashes instead spaces)
and the body as the text content of that file. If you don't want to blog,
simply limit your commit message to an one-liner.

The implication of this is that you can edit a blog entry by using the same
first line in the commit, but the new commit body instead. Yes, it also means
that you cannot edit a blog post without modifying something in the actual
project in order to have something to commit, but remember: this blogging
engine is for software projects, not for personal blogging.

If you want to automatically publish your blog article (if there is one) when you commit, activate it:

    git config devblog.auto-push true

If you don't, you will have to push manually:

    git push origin blog

Note that on github, this is:

    git push origin blog:gh-pages

Or you can configure the branch name to be `gh-pages` in your `.git/config`.

With an existing project
------------------------

I will cover here the instructions for github-hosted projects, as it is the
more common scenario, and customizing the parameters is easier for you if you
have an existing project on your own server - since you know what you're doing.

Remove any sample hooks:

    rm -rf .git/hooks/*.sample

If you have any hooks already, make a backup.

Now copy new hooks:

    git init --template=/path/to/gittoolbox myproject

(Note: At the moment, there is no infrastructure in place to forward calls to
multiple hooks, but it is a planned feature)

Open the config file and append the default config, for instance in vim:

    vim .git/config
    G
    :r /path/to/gittoolbox/config

For github or any other machine you can't ssh into with only a ssh keypair,
disable the automatic deployment of hooks and configurations:

    git config --unset devblog.auto-sync-gitfiles
    git config --unset devblog.auto-sync-gitconfigs

Tailor the values to your needs:

    git config devblog.blog-title "My Blog"
    git config devblog.blogging-branch gh-pages
    git config devblog.auto-push true

    
Customizations
==============
Custom template:

Get the current template:

    git config --get devblog.template.article|base64 -d|gunzip > my-template.html

Store back the template:

    git config -z devblog.template.article `cat my-template.html|gzip|base64 -w 0`
