---
layout: post
status: publish
published: true
title: Gitlab removal
excerpt_separator: ===
categories:
- 其他技术
tags:
- Gitlab
---

#Gitlab removal

Hi guys, long time no see. these day i have much work and much things to learn so i didnot write any thing in the blog, sorry about that. its really busy in my probject. u know people always have their own reason not to do something lol. one day i saw my server which i bought in aliyun is almost time over, so i think about whether i should change the region of the server to hongkong. finally i make a decision to move my blog, gitlab and some other things to hongkong server.

at the begion of the preparation, i try to buy a hongkong server for a month to test the network and server's perform. actually it is really good. so im start to move blog, gitlab and svn etc. to my new server.

===

#Prolbem

1.the first problem i have is which folder or data i have to move.

i move the whole git folder to new server include gitlab, gitlab-satellites, gitlab-shell, https-ca, repositories.

2.the second problem i have is mysql data.

i have lots of data in gitlab and the gitlab save the data in mysql, so i have to move them as the same.

actually its easy to move mysql. i just install mysql in new server and dump the data of mysql then import them into new server.

ok, all of the data is already be move to new server.

3.the hardest things i have is install ruby, lib of mysql and some of the dependent stuff.

its really hard for me to install lib of mysql for ruby because i dont know which source of apt-get i should use. i installed
5.5.47 mysql version but it tells me the lib of mysql is 5.5.37 and it is the newest version in source of apt-get which the installed is failed. i change some of them but still cant be install. i even download the mysql of source and make them and try to pick the lib to install but failed. finally i choose the nature source of aliyun to install the lib of mysql, its success. i think its my bad, because i dont believe the nature source of aliyun so i change it.

4.now i finished setup the lib of all i need. and then i have to set the config to let the gitlab run.

i set the gitlab config, gitlab-shell config and the nginx vhost, its much easy because i set them before in the old server.

ok i should check all of the setting by using: sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production

all of thing is ok now, so lets run the gitlab.
===

