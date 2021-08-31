---
layout: post
title:  "Recap: HTB Traceback"
date:   2021-08-30
categories: htb recap writeup
---

**NOTE:** This is a writeup of a box I have previously done (thus the "Recap"). Therefore, this post will be straightforward and to the point on how this box was solved. For a more realistic view of my process, try a non-recap writeup!



This was one of my first boxes, done when I first tried HTB as a platform. Traceback definitely has a special spot in my heart. :)

Getting straight to business, after the obligatory ping to see that everything is working, we launch an nmap scan while we go check out port 80 for some recon.

![nmap scan](/assets/Traceback/nmap.png)

Port 80 is open. If you go check out the webpage you'll find something a little less than usual.

![Webpage on port 80](/assets/Traceback/webpage.png)

This is the idea behind this box. A very straightforward beginer idea, to give them a backdoor to just jump into. Now we just have to see how we get into it. A look into the HTML source of the page has this little comment. 

![HTML source of webpage](/assets/Traceback/comment.png)

A bit suspicious. A quick trip to google with this phrase brings up a [GitHub repository](https://github.com/TheBinitGhimire/Web-Shells) with a list of some webshells. The list is actually quite short, so there is no need to pull up any sort of fuzzer. A quick run down the list reveals that this uses smevk.php.

![Webshell login page](/assets/Traceback/smevk_login.png)

We've found our webshell (with a login page, how ironic). On the webshell GitHub page, we follow the link trail to [the smevk repository](https://github.com/TheBinitGhimire/Web-Shells/blob/master/PHP/smevk.php) to pull the default login credentials (admin/admin) and get our way in.

![Webshell home page](/assets/Traceback/smevk_home.png)

Not the simplest nor prettiest webshell, but it's a foothold. Let's get a "real" shell while we're here. While netcat is installed on the system, I couldn't seem to get a shell off of it (many versions don't have the -c or -e flags for obvious reasons). But since we have read/write anyway, we'll just get ourselves a login with an SSH key. According to the top of the interface, we're the `webadmin` user, so we'll just head over to his home page and toss our key into `authorized_keys`

![New authorized_keys](/assets/Traceback/authorized_keys.png)

![SSH session established](/assets/Traceback/webadmin.png)

Okay great. A quick `ls` tells us that there is no flag here, so we'll need to move laterally. Additionally, there is a note here. What could that tool to practice lua be?

So we run through some of the quick essentials and our user has permission to run a program called "luvit" as sysadmin. Let's just use sudo to run it, and it's... a lua intepreter. The tool to practice lua.

![lua interpreter](/assets/Traceback/sudo_l.png)

Okay great, now all we need to do is use Lua's `os.execute()` to give us a shell, and probably the flag aswell.

![User flag](/assets/Traceback/userflag.png)

First things first, we set up shop by putting our ssh key into this user's authorized keys, for some persistence (it's an easy box, I don't need to fly under any radars). Not going to supply a screenshot for this because it's nothing special.

Now, we get root. The astute may have noticed that the motd is changed on the box as well, and we know that those are scripts that are run on login. `uname -a` reveals that we are on an Ubuntu machine, and we know that modern Ubuntu machines store the motd in `/etc/update-motd.d/`

![etc/update-motd.d](/assets/Traceback/updatemotd.png)

There we are. This should be run as root when we log in, so we get to do whatever we want as root here! Let's just grab a shell. Note there is some weirdness, the motd will often rewrite itself, meaning you'll lose your edits. Given that all of these scripts execute, I decided to edit `10-help-text` as it seems the least likely to be updated (and overwritten) by the system.

![Our new motd](/assets/Traceback/newmotd.png)

Awesome. Now we just log in before it decides to roll back our changes.

![Root flag get!](/assets/Traceback/rootflag.png)

And there's our flag! That's the traceback box rooted.

**Thoughts:**

All in all, I thought this was a neat beginner box. It was one of my first, and served to introduce to me how powerful webshells were. It also implicitly teaches that privilege escalation is often a result of misconfiguration, as opposed to some strange 0-day or other vulnerability. After all, the entire reason I got root on this box was because of a permissions misconfiguration on the motd. The box was fun, but not very difficult (it's beginner rated, what did we expect). Thanks for reading and have a good one `:)`
