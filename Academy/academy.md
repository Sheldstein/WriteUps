# Academy

Here is a box that was relatively hard, for an easy one. I progressed through the beginning without many difficulties, not fastly, but simply enough, just by being really thorough and applying my usual method when analysing a webapp. Techniques for this first part will be reusable everywhere.
But afer that, the box felt very hard, because I fell in many (every one of them) rabbit holes. The number of users made it even worse, as the privesc wasn't linear. This was a really good thing, because it felt real. I also learned a bit about the adm group and discovered a new tool (gtfo) ! In conclusion, this box was a really valuable experience !

## Recon method

I started by launching nmap, to find out what was running on here :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/nmap.png]

We can see that the ssh and http ports are open, and two uncommon ports as well. I'm not sure what's going on here, but there's a good chance mysql is running !

First thing, I visited the webpage. It redirects me to *academy.htb*, so I needed to add `10.10.10.215  academy.htb` to my `/etc/hosts` file, and here we go :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/academy.png]

I ran a directory bruteforcer (dirbuster) to find hidden pages and directories, and then I explored the website : registering an account, logging in, home page ... After seeing *login.php*, I searched for *admin.php*, and bingo, the page existed, and it was a login page as well.
The enumeration doesn't give any secrets, except for a *config.php* page, which is blank.
During registration, I noticed something hidden, a 'roleid' input. It probably means I can register as an admin if I change the value to one :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/register.png]

Then, I tried to login to the admin page :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/admin.png]

And bingo, I stumbled upon this :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/dev_site.png]

I noticed that there are the names of two devs, **cry0l1t3** and **mrb3n**. There is also a subdomain : *dev-staging-01.academy.htb* ! I quickly added it to my hosts file and looked for it :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/secrets.png]

There was a huge error page ! At that moment, I was a bit lost, I didn't know what to look for. I finally noticed this : the app that errors out is Laravel, there is an app_key. There are also some mysql credentials.

## First exploitation to foothold

Here comes the first rabbit hole : the credentials can not be used on the mysqlx port, which was open. After spending almost an hour trying to make this work, I focused on Laravel. I looked for exploits on searchsploit :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/searchsploit-alternative.png]

There are two remote code executions. I first tried to use the one involving the error page, without success. Then, I resorted to the metasploit one :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/msfconsole.png]

The right configuration took me a few minutes :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/exploit.png]

I ran it and bingo ! I'm in ! :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/foothold.png]

## User

This part was all about enumeration. The user I had, 'www-data' had almost no rights, apart from inside the app directory, so I had to find something in `/var/www/html/academy`.
I first found an hash by searching manually in a file called Userfactory.php, which I could crack into 'secret', and then credentials to log in to mysql as root in the previously mentionned *config.php*, to find other hashes, including an uncrackable one - two other rabbit holes.
I finally noticed this :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/env.png]

I found a password ! Noticing that *cry0l1t3* had the user.txt, I tried to login with ssh with this password (he was also mentionned on the admin page).

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/cry0l1t3.png]

## Pivoting

I noticed this quickly, it's one of the first thing to do when you get a shell as a new user :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/adm.png]

Looking about it on the web, the adm group has access to `/var/log/audit`, so I spent some time trying to work with this. I fell in an other rabbit hole, because I had noticed that user *egre55* was a sudoer.
After some time, I came across this :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/audit.png]

So I tried to connect to the box as *mrb3n* over ssh, and failed. But with `su` it worked !

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/mrb3n.png]

## Root

Running this command, I found that *mrb3n* can run `composer` as root :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/sudoing.png]

I didn't know this, so I tried to use gtfo.py to know how to escalate my privileges with this:

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/gtfo.png]

I followed the instructions, and :

![nmap.png][https://github.com/Sheldstein/WriteUps/blob/main/Academy/root.png]

## Conclusion

And that's all ! I hope it was clear enough, and that you could have a glance at my methodology and my mindset while I was first approaching the box. I try to aim my writeups towards beginners, so you can ask me if you have questions. Thanks for reading !
Also thanks to mrb3n and egre55 for this wonderful box !
