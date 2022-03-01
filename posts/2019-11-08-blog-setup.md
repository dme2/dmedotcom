---
title: Setting up a blog with Hakyll (and nginx)
---

For my first Real Blog Post I figured that I'd just go over how I went about setting this thing up. Despite using hakyll (as apposed to something like jakyll or hugo) and semi-self-hosting this website (on digital ocean), the whole process was actually rather simple.

As its name implies, hakyll is the Haskell answer to Go's Jakyll. It's a good static site generator, but its main appeal is that it relies on pandoc for compiling and transforming files. This allows you to write in a multitude of file formats (html, markdown, latex, etc..) with the final product being the html files that make up your website. [Hakyll's website](https://jaspervdj.be/hakyll/tutorials.html) has some nice introductory tutorials for getting started.

For web-hosting I'm using a $5/month VPS from Digital Ocean. The set up process was super simple (all while being new to me - I've never done this before!). Sitting on the VPS and serving this webstie is nginx with nearly the defualt configuration (which also ended up being super simple to set up). Here's what that script looks like.

```
{
  listen 443 default_server;
  listen [::]:443 default_server;

  ssl on;
  ssl_certificate /etc/letsencrypt/live/dme2.dev/fullchain.p
  ssl_certificate_key /etc/letsencrypt/live/dme2.dev/privkey.pem;

  root /var/www/dmedotcom;
  index index.html;
  server_name dme2.dev www.dme2.dev;

  location / {
    try_files $uri $uri/ =404;
  }
}
server {
       listen 0.0.0.0:80;
       server_name dme2.dev www.dme2.dev;
       rewrite ^ https://$host$request_uri? permanent;
}
```

I tied pushing to my github repository, updating the site on my personal computer, and updating the site on my VPS with this shell script.

```
#!/bin/bash

read -r -p 'enter commit message: ' commit_message

stack exec site rebuild

git add .

git add -u

git commit -m "$commit_message"

git push origin master

echo syncing to web server

rsync -a /Users/dave/dmedotcom/_site/ root@vps_ip:/var/www/dmedotcom
```

And that's basically all there is to it!
