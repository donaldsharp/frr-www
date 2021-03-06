# FRRouting Web Server Preparation and Deployment

This web site uses a pretty simple web server and (mostly) relies on a single
static web page, pulling in CSS, icons/logos, and other HTML as required.

The main components of the directory tree consist of 
    www       - this is the root of the web site, everything is served from here
    etc       - configuration files for the web server
    community - markdown files for simply formatted included content
    templates - header and footer templates for generated files
    logos     - the FRRouting logos
    build     - bash script that checks installation and generates
                community and user-guide content

## Rendered Site Structure (GIT/www)

The frrouting.org site is completely rendered by files under GIT/www with
static files.  GIT/www has the following structure...
```
www/index.html		- main landing page
www/community		- markdown from GIT/community converted to html with
			  format-community script
www/static		- holding pen
   css			- site css styles
   img			- images for the site
   contributors		- contributor logos (see section below)
   icons		- site call-to-action icons (see section below)
www/test-results	- PDF files generated by test system (mwinter)
www/user-guide		- stylized html created with format-manual script
```
## Updating the Site

The web site is fully static, so if the site is running, you only need to update
the appropriate files and they will get picked up (i.e. `git pull` works).  For
the most part the files are part of the git tree under GIT/www.  There are two
instances where we need to format files along the way.

### User Guide

The FRR build system creates an HTML version of the user guide; however, that
version still needs to be stylized for the FRR website.  The `format-manual`
script does this stylization.  The most practical way to make this work is to
clone this repository onto a system where you are building FRR and then run
the script.
```
	format-manual <path-to-frr-build-tree>
```
The script verifies that the build tree looks right and that you've built
the HTML user guide (with `make html`).  Then it copies the manual into this
git repository stylizing it along the way.  You are still responsible for the 
`git commit` :-).

### Community Documents

These are all written in markdown because it's both easy and correctly formated
 when viewed in github.  For web presentation, they converted to HTML with
 `format-community`.  Put any community markdown documents in  GIT/community, 
git-pull on the web server, and run the script.  At this point, they will be 
stylized with the results in GIT/www/community where they are rendered when 
viewing the website.

## Logos and Icons

### Contributors Logos

To make the insertion of logos straightforward, the logos are placed on a white
background that is 225 pixels wide and 130 pixels tall with a resolution of 72
pixels/inch with appropriate margins all around.

GIT/templates/contributor-background.jpg is a good starting point for a new logo.

### Icons

All of the FRR call-to-action icons are 137 pixels wide and 102 pixes tall with
a transparent background.  The original icons were designed by <https://unomena.com>Unomena.

## Preparing a New Site Server

In the off chance that you need to set up a new site server...

NOTE - intructions for an Ubuntu server and assume that you have sudo access.

### Install Necessary Packages

```
$ sudo apt-get install git
$ sudo apt-get install nginx
$ sudo apt-get install nodejs-legacy
$ sudo apt-get install npm
$ sudo npm install markdown-to-html
```

### Clone this repository

```
$ cd /var/www <- create this directory if it does not exist yet
$ sudo git clone git@github.com:FRRouting/frr-www.git
```

### File Ownership

All files in this git tree should be belong to group `frr-www` to facilitate
administration of the site.

```
$ sudo addgroup frr-web
$ sudo addgroup ${USER} frr-web
$ sudo chgrp -R /var/www/frr-web
$ sudo chmod g+s /var/www/frr-web
$ sudo chmod g+s /var/www/frr-web/www/community
$ sudo chmod g+s /var/www/frr-web/www/user-guide
```

## Install configuration files

**NOTE**: Installation of SSL certs requires gpg password.

```
$ cd /var/www/frr-www
$ sudo cp ngnix/frr.conf /etc/ngnix/sites-available/frr.conf
$ sudo ln -s /etc/ngnix/sites-available/frr.conf /etc/ngnix/sites-enabled/frr.conf
$ sudo mkdir /etc/nginx/ssl
$ sudo cp nginx/frrouting.ssl.tar.gpg /etc/nginx/ssl
$ cd /etc/nginx/ssl
$ sudo gpg frrouting.ssl.tar.gpg
$ sudo tar xf frrouting.ssl.tar
$ sudo chmod u=rw *
```

## (Re)Start the Web Server

```
$ sudo service nginx restart
```

