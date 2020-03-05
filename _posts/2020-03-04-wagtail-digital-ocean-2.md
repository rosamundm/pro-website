---
layout: post
title: "Deploying my Wagtail blog to Digital Ocean, pt. 2"
date: 2020-03-04 17:40:20 +0100
---
The first part of this post is [here](https://rosamundmather.com/2020/01/29/wagtail-digital-ocean-1.html).

It's true what they say: when you reach a goal in programming, that sense of achievement is real. It is something I'd underestimated. Coming from a "creative" background as a writer, creating work you're proud of is quite a different feeling. Maybe it's because you know it's come from *you*, whereas when you get stuff right in programming, you figured something out; something that millions of people have also had to learn and get their heads around. Writing is lonely, but programming, no matter how much some may insist otherwise, is community-based.

I was elated when I got my site online. My joy was short-lived — more on that later — but I'm going to publish this post anyway.
<br>
<br>
![502 bad gateway](https://1.bp.blogspot.com/-m_oHBp4ymaw/Xl_sQYPrfuI/AAAAAAAAH8k/PsO-pn3ETi4S1UVbGbi3JzmGgHVdWc5lwCNcBGAsYHQ/s1600/502%2Bbad%2Bgateway.png)
<br>
<br>

<h3>4. Testing — and not</h3>

This is a topic too broad to cover, so I'm just going to say the entirety of <em><a href="[https://www.obeythetestinggoat.com/pages/book.html#toc](https://www.obeythetestinggoat.com/pages/book.html#toc)">Obey the Testing Goat</a></em> by Harry Percival is available online for free, for your reference. It walks you through testing Django apps with Selenium.

Confession: I went back on my word and didn't test this as extensively as I wanted to, as I was kind of in a hurry. Luckily, most stuff worked when the site went live (read on to find out what didn't).
I did have a brush with Geckodriver and I hated it. It's recommended in the book as the webdriver communicating between Firefox and Selenium, as it was a complete pain to install... however, if you are determined to use Firefox as your browser, it's kinda the only way. The first time I tried to add Geckodriver as an executable, I ended up breaking my bash console. (By the way, if this ever happens to you, the solution is to open <code>.bash_profile</code>, delete everything, and make sure the path is <code>export PATH=/bin:/usr/bin:/usr/local/bin</code>).

Here is what I ended up doing, [with](https://selenium.dev/documentation/en/webdriver/driver_requirements/#adding-executables-to-your-path) [these](https://medium.com/dropout-analytics/selenium-and-geckodriver-on-mac-b411dbfe61bc) [guides](https://medium.com/@sonaldwivedi/downloading-and-setting-up-geckodriver-87873e25207c) helping me.

Go [here](https://github.com/mozilla/geckodriver/releases) and pick your Geckodriver version to download to your local drive; if you're using a DO server it'll most likely be <code>geckodriver-v0.26.0-linux64.tar.gz</code> (I spent aaaaages wondering why it wasn't working, when it turned out I'd been on autopilot and downloaded the MacOS one. Doh!) Unzip the folder and there should be a <code>geckodriver</code> executable file within it.

Go to `/usr/local/share` on your Ubuntu server and create a directory called <code>geckodriver2</code> (so that you don't confuse it with the local one). Connect to FileZilla and move the <code>geckodriver</code> file to <code>geckodriver2</code>.

It's possible you'll get the following stack trace error:
<code>selenium.common.exceptions.WebDriverException: Message: 'geckodriver' executable may have wrong permissions.</code> This probably means you'll need to change the `PATH` in the `.bash_profile` file (Google will help you find it). After doing so, run `source .bash_profile` so the changes are applied, then test that it worked by running <code>echo $PATH</code>.

I've played around with Selenium within the context of parsing HTML, but I certainly have more to learn when it comes to testing. Also, the Selenium "language" is called [Selenese](https://www.softwaretestingmentor.com/what-is-selenese/), which sounds nice!

That's all I have to say about testing for now!


<h3>5. Security</h3>

When everything is ready, don't open the golden gates (port 80) to internet traffic yet!!!
First, run <code>python manage.py check --deploy</code>. This will give you an automated list of security issues you need to rectify before you deploy.

Django offers its own <a href="[https://docs.djangoproject.com/en/3.0/howto/deployment/checklist/](https://docs.djangoproject.com/en/3.0/howto/deployment/checklist/)">deployment checklist</a>, or you can refer to this more <a href="[https://dev.to/coderasha/django-web-security-checklist-before-deployment-secure-your-django-app-4jb8](https://dev.to/coderasha/django-web-security-checklist-before-deployment-secure-your-django-app-4jb8)">beginner-friendly one</a>, which spells out some of the security warnings you will see when you run the command above. Among other warnings, I solved <code>W004</code>, <code>W006</code>, <code>W007</code>,<code>W008</code> thanks to this guide. I also found that some issues with static and styling could be solved by clearing the browser cache.

There's also the perennial question of how to deal with secret info in Django projects, such as database credentials and the `SECRET_KEY`. I made sure I had all info common to development and production in <code>[base.py](https://github.com/rosamundm/rosederwelt-blog/blob/master/mysite/mysite/settings/base.py)</code>, while private info was in `dev.py`, which in turn was listed in the `.gitignore`. The "public version" with concealed info, which runs in production, is therefore in [`production.py`](https://github.com/rosamundm/rosederwelt-blog/blob/master/mysite/mysite/settings/production.py).

Again, virtual environments are a whole 'nother topic, so I'll just say I used the `environ` library. Docs [here](https://django-environ.readthedocs.io/en/latest/).

<h3>6. Deployment</h3>

Now that your security settings are hopefully watertight, make sure you set <code>DEBUG</code> to <code>False</code> in your `production.py` file! This is like the code version of having lipstick on your teeth, only significantly more dangerous in terms of privacy and security.

Since I had been working on a VM but had done the bulk of the actual site creation on my local machine, I had to generate another SSH key so I could commit my project to GitHub. I ended up creating a new repo for the stuff I'd been doing on the VM, as I wanted to to keep it separate from the work I'd done which was mainly "design"-related.

After setting the DNS records for my desired domain on Digital Ocean, the next step was getting an SSL certificate — I used [Certbot](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx), which is free (be sure to select the right system!). Then, as the DO documentation says, I did `sudo ufw delete allow 8000` then `sudo ufw allow 'Nginx Full'` to close the development server port and open port 80.

And then the website was online at [rosederwelt.com](https://rosederwelt.com), and it was good. For a while there, I didn't think that day would come!

<h3>7. Work in progress</h3>

Alas, I sabotaged myself when I tried fixing something. When editing a page in the Wagtail admin and previewing it, I got a 400 error. I started to fix this by going in to the admin panel, clicking on clicking on *Settings*, then *Sites*:
<br>
<br>
![Wagtail admin panel](https://1.bp.blogspot.com/-KVMrfRyjOCY/Xl_W3at7MSI/AAAAAAAAH8E/dhypEhUGih0Nm7Iot-Rc5zmdalENQi7JwCNcBGAsYHQ/s1600/wagtail1.png)
<br>
<br>
![Click on Pages, then Sites](https://1.bp.blogspot.com/-SL0wCMm3v7M/Xl_W3am9VrI/AAAAAAAAH8M/-96QV9nOFzoTAkv7kzZ6UkQhE-azm3IUwCNcBGAsYHQ/s1600/wagtail2.png)
<br>
<br>
From there, click on *localhost* with port *80*. Change the number of the port. Also, I don't think it's mandatory, but I also just changed the port name for clarity. Save.
<br>
<br>
![Click on the name of the site. Change the port number and site name.](https://1.bp.blogspot.com/-SZ2M6gMoRoQ/Xl_W3Tcv_GI/AAAAAAAAH8I/u1czBdJF09or0wxJbsqbi4-zUD18Q1FSACNcBGAsYHQ/s1600/wagtail3.png)
<br>
<br>
But then none of this mattered anyway, because soon after that, I ended up accidentally deleting half of my virtual environment, and as a result, nothing would run. Unfortunately, simply deleting what was left of the environment and creating a new one didn't have the intended effect; it messed up the paths. Now my Django and Gunicorn installations are out of whack, which I'm still trying to figure out the best way to fix.

As I keep coming back to dealing with this, the thing to remember is that I did it all on my own. Most people build products and apps in teams where different people are responsible for various parts of the process. Not everyone has the patience to stick through the frustration of doing a whole project by themselves. But I did, even when I was convinced I sucked, even when it felt like the odds were against me and none of this was going to lead me anywhere I wanted, professionally or personally.

And I want *more* of it.
