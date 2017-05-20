# Modify-WordPress-Commandline
This repository is used to show a working example of a script that modifies a WP database to repoint to a new host, counts the posts &amp; comments, updates the wp-config file and removes any lower level pages references from the .htaccess

```
#!/bin/bash

/Applications/MAMP/Library/bin/mysql --user=test --password=test  --table -h localhost <<!!
USE A3;

#update the WordPress directories for the SITEURL
SET @t=(SELECT table_name FROM information_schema.tables where table_schema='A3' AND table_name LIKE 'wp%_options' LIMIT 1);
SET @c=(SELECT table_name FROM information_schema.tables where table_schema='A3' AND table_name LIKE 'wp%_comments' LIMIT 1);
SET @p=(SELECT table_name FROM information_schema.tables where table_schema='A3' AND table_name LIKE 'wp%_posts' LIMIT 1);

SET @query1=CONCAT("UPDATE ", @t, " SET option_value = 'http://junk.local' WHERE option_name = 'siteurl' ");
PREPARE stmt1 FROM @query1;
EXECUTE stmt1;
DEALLOCATE PREPARE stmt1;

#update the WordPress directories for the HOME
SET @query1=CONCAT("UPDATE ", @t, " SET option_value = 'http://junk.local' WHERE option_name = 'home' ");
PREPARE stmt1 FROM @query1;
EXECUTE stmt1;
DEALLOCATE PREPARE stmt1;

#show of the current WordPress posts
SET @query1=CONCAT("SELECT DISTINCT post_title AS PostTitles FROM ", @p);
PREPARE stmt1 FROM @query1;
EXECUTE stmt1;
DEALLOCATE PREPARE stmt1;

#count the current WordPress posts
SET @query1=CONCAT("SELECT COUNT(*) AS Posts FROM ", @p);
PREPARE stmt1 FROM @query1;
EXECUTE stmt1;
DEALLOCATE PREPARE stmt1;

#count the current WordPress comments
SET @query1=CONCAT("SELECT COUNT(*) AS Comments FROM ", @c);
PREPARE stmt1 FROM @query1;
EXECUTE stmt1;
DEALLOCATE PREPARE stmt1;

SELECT @t AS WP_Name, LENGTH(@t) AS Len, IF (LENGTH(@t)>10, "WP_Name Ok", "WP_Name Failed") AS WP_Naming;

quit
!!

sed -i -e "s/.*define('DB_NAME', '.*/define('DB_NAME', 'A3');/" /Applications/MAMP/htdocs/JUNK/wp-config.php
sed -i -e "s/.*define('DB_USER', '.*/define('DB_USER', 'test');/" /Applications/MAMP/htdocs/JUNK/wp-config.php
sed -i -e "s/.*define('DB_PASSWORD', '.*/define('DB_PASSWORD', 'test');/" /Applications/MAMP/htdocs/JUNK/wp-config.php
sed -i -e "s/.*define('DB_HOST', '.*/define('DB_HOST', 'localhost');/" /Applications/MAMP/htdocs/JUNK/wp-config.php

sed -i -e "s/.*RewriteBase \/.*/RewriteBase \//" /Applications/MAMP/htdocs/JUNK/.htaccess
sed -i -e "s/.*RewriteRule . \/.*/RewriteRule . \/index.php [L]/" /Applications/MAMP/htdocs/JUNK/.htaccess

```
