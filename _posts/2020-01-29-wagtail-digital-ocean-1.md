---
layout: post
title: "Deploying my Wagtail blog to Digital Ocean, pt. 1"
date: 2020-01-29 19:35:15 +0100
---

As I mentioned in my [first post](https://rosamundmather.com/2019/10/24/hello-world.html), my main project for the past couple of months has been building a Wagtail blog. It is finished, and it runs, but deploying it has been a bit of a sticking point. Now that I'm in a place where I could theoretically do so, I don't want to, mainly because I want to test it properly and learn good habits.

Still, I've learnt a lot through this process so far and wanted to share as I go along. Also, I'd rather have at least some content on my blog than wait until absolutely everything is perfect.

I'd been an admirer of 🦈[Digital Ocean](https://www.digitalocean.com/)🦈 long before I even considered learning to code. A few years ago, in my capacity as a content writer, I went to a conference about diversity in tech, and they had some speakers from DO. I immediately loved their branding and picked up some swag on the way out. When I started to get curious about building my own web apps, I dived into their approachable, community-focused documentation. I never actually saw myself using DO's services, though, because I assumed it was reserved for heavyweight products.

However, when I was looking to host my Wagtail blog, I tried out PythonAnywhere, Heroku, and Divio in succession. These are  commonly used for hobby and personal projects, but for various reasons, they each failed to work for me. I'm not dunking on any of these services; I'd absolutely consider using them in the future, but they just didn't deliver what I needed at that point. Exasperated, I saw that DO were giving free credit to new users, so I thought I'd give them a go. I'm really happy with them so far and have really enjoyed learning about stuff that wouldn't have occurred to me before, such as.... Ubuntu.

Somewhat naively, I'd assumed I would never have to give Linux OS — which Ubuntu is distributed from — the time of day. I mean, I'm a Mac OS user so I'm basically set for development, right? <em>Wrong.</em> If you're serious about Python/Django, you will most likely have a brush with Linux at some point.

Thankfully, you don't need to obtain a whole new Linux machine, or even install an <a href="https://medium.com/@mannycodes/installing-ubuntu-18-04-on-mac-os-with-virtualbox-ac3b39678602">Ubuntu virtual machine</a>, because DO does that for you (well, after you've done a bit of setup, of course).

Please note that this article is not an comprehensive walkthrough, but a compilaton of observations and extra notes accompanying the Digital Ocean tutorials that I reference here.

<h3>1. Initial setup</h3>
Follow the instructions on <a href="https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04">Initial Server Setup with Ubuntu 18.04</a> to get the ball rolling.

After successfully SSH'ing into your Ubuntu server, you will most likely receive a message saying packages need to be updated. Run the following:<br>
<code>sudo apt upgrade</code>
<code>sudo apt update</code>

And if you get the `Permission denied (publickey)` error, run this, thus opening the config file:
<code>sudo nano /etc/ssh/sshd_config</code>

Make sure <code>PermitRootLogin</code> and <code>PasswordAuthentication</code> are both set to <code>yes</code>, then reboot the server with <code>sudo service ssh restart</code>. Log in with your username again.


<h3>2. Things get more exciting</h3>

After you've finished that, go straight onto the next tutorial: <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04">How to Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 18.04</a>.

Of course, I used Wagtail instead of the regular Django framework. I'd recommend first familiarising yourself with <a href="[https://docs.wagtail.io/en/v2.6.2/getting_started/tutorial.html](https://docs.wagtail.io/en/v2.6.2/getting_started/tutorial.html)">starting a project on Wagtail</a> — perhaps first on your own local machine —  then you can substitute the Django commands with Wagtail ones.

I must say, I got quite a kick out of seeing the Wagtail welcome screen finally pop up, dancing egg and all, after months in vain of trying to get it going!

![wagtail start page](https://1.bp.blogspot.com/-kQxR45caCnw/XheONmsrhTI/AAAAAAAAH5c/HXC6FEVcltY4c-ymIHySQCzMu-46jr-IwCNcBGAsYHQ/s640/wagtail_start.png)

Now it's time to meet Gunicorn, the WSGI application server. 👋🏻🦄

You'll first need to add your app name to <code>base.py</code> in order to get it running. Then, go to the directory where <code>wsgi.py</code> is stored, otherwise you will keep getting worker errors. Basically, as some commenters on the DO tutorial have observed, you will probably need to add an extra directory to the end of the path given in the tutorial — i.e. the directory where the <code>wsgi</code> location is specified). For example, if your project directory is called `mynewblog` and your app directory is `mysite`, you will need to enter the path <code>/home/username/mynewblog/mysite/mysite</code>.

When you get to the part where you're testing Gunicorn, make sure you're in <code>/home/username/mynewblog/mysite</code>, and from there, run <code>gunicorn --bind 0.0.0.0:8000 mysite.wsgi</code> to start the server and see if the Wagtail site pops up in your browser.

It was plain sailing for me up until the point where I had to run the <code>curl</code> command mentioned in the tutorial — <code>curl --unix-socket /run/gunicorn.sock localhost</code> — throwing me the `7` and `56` errors respectively.

Once again, when configuring <code>gunicorn.service</code>, be sure to specify the right directory. For example, if your project directory is `mynewblog` and your app directory is `mysite`, you will need to make the path <code>/home/username/mynewblog/mysite/mysite</code>
<code><directory-holding-wsgi-file>.wsgi:application</code> then <code>exit</code> and restart the SSH connection like so:<br>
<code>sudo systemctl start gunicorn.socket</code>
<code>sudo systemctl enable gunicorn.socket</code>
<code>curl --unix-socket /run/gunicorn.sock localhost</code>

<h3>3. Getting your files onto the DO server</h3>
For some reason I was under the impression that once I came to the end of the last tutorial, I'd be looking at my new site.

Downloading [FileZilla](https://filezilla-project.org/) was slightly scary, not only because I think the website looks a bit "🤨" and the name evokes a Limewire-era file-sharing site, but also because I had my reservations about giving my private SSH key to a third-party app. Still, there were lots of trusted reviews all over the internet, so I went for it.

At first I couldn't log in with my (non-root) username. You have to specify the port on FileZilla, and I had assumed it to be either `80` or `8000`. Only when I tried `22` did it work; indeed, this is the SFTP (secure file transfer protocol) port.

![alt: "screenshot of FileZilla homepage"](https://1.bp.blogspot.com/-OVUzWisasHQ/XjG7SjzxeVI/AAAAAAAAH6U/p0xHaWdTZKYjBdqsfOfEsUWv0lrA5_q9wCNcBGAsYHQ/s640/filezilla.png)

I then tested this by running <code>$ netstat -tulpn</code>, which allows you to see the ports you're connected to.

[This](https://dev.to/coderasha/deploy-your-django-application-to-digital-ocean-using-nginx-complete-tutorial-c1l) is a good article to look at if you get stuck!

<h3>More stuff I learnt</h3>

If you get this error when trying to run the development server:

<code>django.db.utils.OperationalError: could not connect to server: Connection refused
	Is the server running on host "<000>.<000>.<000>.<000>" and accepting
	TCP/IP connections on port 5432?</code>

Go to <code>/etc/postgresql/<version_number>/main</code> and run <code>sudo nano postgresql.conf</code>, then add port `5432` (it should be around line 64 of the file).

If that doesn't work, run <code>service postgresql status</code> — hopefully the port will be active and everything will look okay. You can also run <code>netstat -na</code> to show your list of current, active internet connections, and if that doesn't work, try <code>ufw allow 5432/tcp</code> to open the port directly.

All this being said, in one case, it took me a while to realise that the problem was not to do with ports and hosts, but in fact to do with my database. I had set an environment variable for it, based on [dj-database-url](https://github.com/jacobian/dj-database-url), but for some reason it didn't take. (I'll be talking more about environment variables in another post).

Another recurring error was the static files not loading. I'd had this problem before; for some reason, I'd kept the <code>static</code> folder within my app directory, not the project directory — this makes a lot of sense, as it holds the static files (including styling files) for the entire project. Although it was annoying, as always, I learnt something new about directory paths. Check the static path in `/etc/nginx/sites-available`, then open the file with the same name as your project. You can also `mv "static" "/home/username/mynewblog"` to move your static directory into the right place.

Here's some [more info](https://docs.djangoproject.com/en/2.2/howto/static-files/) about static files.

<h3>Stay tuned!</h3>

I'm currently learning about test-driven development (TDD), which is one of the "good habits" I mentioned, so the next part of this series will be about tests, security, and with a bit of hard work and luck, deployment...
