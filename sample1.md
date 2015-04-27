Title: Initial Blog Set Up
Date: 2014-12-19 11:10
Category: Blog notes

# Initial experiment with Pelican blogging system


I started by installing Pelican and Markdown, since I intended to write my Pelican content in Markdown:

    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects$ pip install Pelican
    Requirement already satisfied (use --upgrade to upgrade): Pelican in /usr/local/lib/python2.7/dist-packages
    Cleaning up...
    me@Bedrock1:~/Projects/PelicanProjects$ pip install Markdown
    Requirement already satisfied (use --upgrade to upgrade): Markdown in /usr/local/lib/python2.7/dist-packages
    Cleaning up...

(OK - they were already installed before I wrote this entry, but such is life)

Next, I created the directory where the blog content and blog
support/creation files would live 

    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects$ mkdir test1
    me@Bedrock1:~/Projects/PelicanProjects$ cd ./test1

The creation of the blog was then handled by running the
*pelican-quickstart* command. This created the directory structure in
which the blog would live and asked a series of questions that helped
create the configuration files (pelicanconf.py & publishconf.py) and
files to help speed standard tasks for creating blog entries and
publishing them (Makefile & fabfile.py - I will be using Makefile). It
will also create a directory for content and one for output files.

    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects/test2$ pelican-quickstart
    Welcome to pelican-quickstart v3.5.0.

    This script will help you create a new Pelican-based website.

    Please answer the following questions so this script can generate the files
    needed by Pelican.

    
    > Where do you want to create your new web site? [.] ./
    > What will be the title of this web site? Notes To Self
    > Who will be the author of this web site? Moonshine Mulligan
    > What will be the default language of this web site? [en] 
    > Do you want to specify a URL prefix? e.g., http://example.com   (Y/n) Y
    > What is your URL prefix? (see above example; no trailing slash)  http://tubaybb321.github.io
    > Do you want to enable article pagination? (Y/n) 
    > How many articles per page do you want? [10] 
    > Do you want to generate a Fabfile/Makefile to automate generation and publishing? (Y/n) 
    > Do you want an auto-reload & simpleHTTP script to assist with theme and site development? (Y/n) 
    > Do you want to upload your website using FTP? (y/N) 
    > Do you want to upload your website using SSH? (y/N) 
    > Do you want to upload your website using Dropbox? (y/N) 
    > Do you want to upload your website using S3? (y/N) 
    > Do you want to upload your website using Rackspace Cloud Files? (y/N) 
    > Do you want to upload your website using GitHub Pages? (y/N) y
    > Is this your personal page (username.github.io)? (y/N) y
    Done. Your new project is available at /home/me/Projects/PelicanProjects/test1

This process will create the following directory structure:

    
    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects/test2$ ls ./content
    me@Bedrock1:~/Projects/PelicanProjects/test2$ tree
    .
    ├── content
    ├── develop_server.sh
    ├── fabfile.py
    ├── Makefile
    ├── output
    ├── pelicanconf.py
    └── publishconf.py

    2 directories, 5 files

# Testing Blog 
Now, I can test my blog on localhost (i.e. http://localhost:8000) to see a blank entry and convince
myself that the machinery of the blog is working, by using one of these
commands from the Makefile:

    
    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects/test2$ make serve
    cd /home/me/Projects/PelicanProjects/test2/output && python -m pelican.server
    127.0.0.1 - - [19/Dec/2014 22:03:34] "GET / HTTP/1.1" 200 -
    WARNING:root:Unable to find file /favicon.ico/index.html or variations.
    WARNING:root:Unable to find file /favicon.ico/index.html or variations.
       C-c C-cmake: *** [serve] Interrupt

My other option is:

    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects/test1$ make devserver
    /home/me/Projects/PelicanProjects/test1/develop_server.sh restart
    Stale PID, deleting
    Stale PID, deleting
    Starting up Pelican and HTTP server
    DEBUG: Adding current directory to system path
    DEBUG: Temporarily adding PLUGIN_PATHS to system path
    DEBUG: Restoring system path
      --- AutoReload Mode: Monitoring `content`, `theme` and `settings` for changes. ---

    -> Modified: content, theme, settings. re-generating...
    DEBUG: Template list: [u'!simple/archives.html', u'!simple/article.html', u'!simple/author.html', u'!simple/authors.html', u'!simple/base.html', u'!simple/categories.html', u'!simple/category.html', u'!simple/gosquared.html', u'!simple/index.html', u'!simple/page.html', u'!simple/pagination.html', u'!simple/period_archives.html', u'!simple/tag.html', u'!simple/tags.html', u'!simple/translations.html', u'analytics.html', u'archives.html', u'article.html', u'article_infos.html', u'author.html', u'authors.html', u'base.html', u'categories.html', u'category.html', u'comments.html', u'disqus_script.html', u'github.html', u'gosquared.html', u'index.html', u'page.html', u'pagination.html', u'period_archives.html', u'piwik.html', u'tag.html', u'taglist.html', u'tags.html', u'translations.html', u'twitter.html']
    DEBUG: Read file sample1.md -> Article
    DEBUG: Signal article_generator_preread.send(ArticlesGenerator)
    DEBUG: Signal article_generator_context.send(ArticlesGenerator, <metadata>)
    -> Writing /home/me/Projects/PelicanProjects/test1/output/initial-blog-set-up.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/index.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/tags.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/categories.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/authors.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/archives.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/category/blog-notes.html
    -> Writing /home/me/Projects/PelicanProjects/test1/output/author/moonshine-mulligan.html
    Done: Processed 1 article(s), 0 draft(s) and 0 page(s) in 0.10 seconds.
    Pelican and HTTP server processes now running in background.

Running in the background is useful, since now I can still use my terminal. This is particularly helpful if I want to add content. Using my same window I can enter the command to convert (i.e. make html) the md (i.e. Markdown) files in the content directory to html and the immediately see it on my local server. I just need to refresh the page on the browser.


# Placing Blog In A Repository And On Internet
Next, I wanted to create a GitHub repository. Since this was not going
to be associated with a project, I decided to use a GitHub user page.

## Git Repository
### Creating Local Repository
    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects/test1/output$ git init
    Initialized empty Git repository in /home/me/Projects/PelicanProjects/test1/output/.git/
    me@Bedrock1:~/Projects/PelicanProjects/test1/output$ git add --all
    me@Bedrock1:~/Projects/PelicanProjects/test1/output$ git status
    On branch master

    Initial commit

    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)

	    new file:   archives.html
	    new file:   author/moonshine-mulligan.html
	    new file:   authors.html
	    new file:   categories.html
	    new file:   category/blog-notes.html
	    new file:   feeds/all.atom.xml
	    new file:   feeds/blog-notes.atom.xml
	    new file:   index.html
	    new file:   initial-blog-set-up.html
	    new file:   tags.html
	    new file:   theme/css/main.css
	    new file:   theme/css/pygment.css
	    new file:   theme/css/reset.css
	    new file:   theme/css/typogrify.css
	    new file:   theme/css/wide.css
	    new file:   theme/images/icons/aboutme.png
	    new file:   theme/images/icons/bitbucket.png
	    new file:   theme/images/icons/delicious.png
	    new file:   theme/images/icons/facebook.png
	    new file:   theme/images/icons/github.png
	    new file:   theme/images/icons/gitorious.png
	    new file:   theme/images/icons/gittip.png
	    new file:   theme/images/icons/google-groups.png
	    new file:   theme/images/icons/google-plus.png
	    new file:   theme/images/icons/hackernews.png
	    new file:   theme/images/icons/lastfm.png
	    new file:   theme/images/icons/linkedin.png
	    new file:   theme/images/icons/reddit.png
	    new file:   theme/images/icons/rss.png
	    new file:   theme/images/icons/slideshare.png
	    new file:   theme/images/icons/speakerdeck.png
	    new file:   theme/images/icons/stackoverflow.png
	    new file:   theme/images/icons/twitter.png
	    new file:   theme/images/icons/vimeo.png
	    new file:   theme/images/icons/youtube.png

    me@Bedrock1:~/Projects/PelicanProjects/test1/output$ git commit -m "Initialize blog"
    [master (root-commit) 6034ebb] Initialize blog
     35 files changed, 2472 insertions(+)
     create mode 100644 archives.html
     create mode 100644 author/moonshine-mulligan.html
     create mode 100644 authors.html
     create mode 100644 categories.html
     create mode 100644 category/blog-notes.html
     create mode 100644 feeds/all.atom.xml
     create mode 100644 feeds/blog-notes.atom.xml
     create mode 100644 index.html
     create mode 100644 initial-blog-set-up.html
     create mode 100644 tags.html
     create mode 100644 theme/css/main.css
     create mode 100644 theme/css/pygment.css
     create mode 100644 theme/css/reset.css
     create mode 100644 theme/css/typogrify.css
     create mode 100644 theme/css/wide.css
     create mode 100644 theme/images/icons/aboutme.png
     create mode 100644 theme/images/icons/bitbucket.png
     create mode 100644 theme/images/icons/delicious.png
     create mode 100644 theme/images/icons/facebook.png
     create mode 100644 theme/images/icons/github.png
     create mode 100644 theme/images/icons/gitorious.png
     create mode 100644 theme/images/icons/gittip.png
     create mode 100644 theme/images/icons/google-groups.png
     create mode 100644 theme/images/icons/google-plus.png
     create mode 100644 theme/images/icons/hackernews.png
     create mode 100644 theme/images/icons/lastfm.png
     create mode 100644 theme/images/icons/linkedin.png
     create mode 100644 theme/images/icons/reddit.png
     create mode 100644 theme/images/icons/rss.png
     create mode 100644 theme/images/icons/slideshare.png
     create mode 100644 theme/images/icons/speakerdeck.png
     create mode 100644 theme/images/icons/stackoverflow.png
     create mode 100644 theme/images/icons/twitter.png
     create mode 100644 theme/images/icons/vimeo.png
     create mode 100644 theme/images/icons/youtube.png

### Creating Remote Repository
At this point, I had my local repository. The next step was to create
a remote repository, which would not only back things up remotely, but
would also (in this case) create a web page. So, I went to github.com, created an account and then found myself facing this page:

<img src="/GitHubShot1a.png" alt="/GitHubShot1.png" style="width: 750px; height: 800px;"/>

When I selected the area I've circled in red, a drop-down menu appeared, allowing me to create a new repository., which took me to the following page:

<img src="/GitHubShot2.png" alt="/GitHubShot2.png" style="width: 750px; height: 800px;"/>

Apparently, GitHub will allow 1 User/Organization page, which seems to be associated with pages not attached to any particular project, and as many Project pages as one wants. Since these webpages are purely a set of notes to myself, a User page seemed more appropriate. To create a user page, I just declared the repository name to be <my username>.github.io - tubaybb321.github.io, which is now the web address of these pages. Once this repository was created, I got taken to the following page:

<img src="/GitHubShot3a.png" alt="/GitHubShot3.png" style="width: 750px; height: 800px;"/>

Clicking on the circled button copies the web address of my new page to the clipboard, which made it easier to upload the webpages I'd created in this way:



## Updating GitHub Repository
    ::::Text only
    me@Bedrock1:~/Projects/PelicanProjects/test1/output$ git remote add origin https://github.com/tubaybb321/tubaybb321.github.io.git #Only necessary when first connecting local repository to remote repository
    me@Bedrock1:~/Projects/PelicanProjects/test1/output$ git push origin master
    Username for 'https://github.com': tubaybb321
    Password for 'https://tubaybb321@github.com': 

    Counting objects: 44, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (42/42), done.
    Writing objects: 100% (44/44), 27.58 KiB | 0 bytes/s, done.
    Total 44 (delta 10), reused 0 (delta 0)
    To https://github.com/tubaybb321/tubaybb321.github.io.git
     * [new branch]      master -> master
    me@Bedrock1:~/Projects/PelicanProjects/test1/output$  
