# seguru-wordpress-template-all-plugins

This repo is an example of a repo configured for Hybrid deployment

This means that only the contents of the following directories are going to be deployed:
```
wp-content/plugins
wp-content/themes
```

You can add your custom plugins and themes to these directories and when deployed only these plugins and themes will be updated on your site.

This system allows for you to manage some plugin and themes via wp-admin and others via git.

However this system will also allow WordPress to auto-update even your git managed plugins if you do not disable auto-updates for them. This will only matter if you are version controlling publicly available plugins/themes. In the case of your own developed
plugins and themes this should not matter. If you do mistakenly update your git managed plugins via wp-admin then you can roll things back easily by using the redeploy feature from the panel.


## .gpconfig directory

### Deployment Type

This repo contains:
```
.gpconfig/hybrid
```
This is the token file that will determin the type of deployment, because this file exists in this repo, then the deployment type will be `hybrid`

### Deployment Scripts

You will find the following empty scripts:
```shell
/var/www/{site.domain}/releases/release-{epoch}/.gpconfig/predeploy-server.sh
/var/www/{site.domain}/releases/release-{epoch}/.gpconfig/predeploy.sh
/var/www/{site.domain}/releases/release-{epoch}/.gpconfig/postdeploy.sh
/var/www/{site.domain}/releases/release-{epoch}/.gpconfig/postdeploy-server.sh
```

These scripts will execute in the above order.

The `-server.sh` scripts are run as root, with the others running as the site system user.

The scripts are all run from the `/.gpconfig` directory.

The root scripts can be useful for running scripts across the server and microservices which need elevated privileges, be careful.

The system user scripts can be useful for running system user functionality, php, wp-cli etc

Remember to include the correct paths to htdocs and wp core files for wp-cli commands.


### Release Directory Retention

```
.gpconfig/keep.releases
```

This repo contains this file, the contents of the file can be an `integer` only, and the integer must be greater than or equal to `3` - this represents the minimum number of releases we will allow you to retain.

This `integer` is the number of releases that the system will retain in the following directory:
```
/var/www/{site.domain}/releases
```

Please see below for details of the releases and release directory naming convention.


## Plugins

Any plugins included in:
```
wp-content/plugins
```

Will be deployed to the releases directory, and then rsynced to the `htdocs/wp-content/plugins` directory with `delete` flag to overwrite any existing matching plugin.

### MU Plugins

Any MU Plugins included in:
```
wp-content/mu-plugin
```

Will be deployed to the releases directory, and then rsynced to the `htdocs/wp-content/mu-plugin` directory with `delete` flag to overwrite any existing matching mu plugin.


## Themes

Any themes included in:
```
wp-content/mu-plugin
```

Will be deployed to the releases directory, and then rsynced to the `htdocs/wp-content/mu-plugin` directory with `delete` flag to overwrite any existing matching mu plugin.


## Directory Structure Changes

### Directory Structure when Git Integration is enabled

**For Hybrid Git Repos this state is transitory and will change as soon as a deployment is triggered**

When you first enable the Git Integration Connection for a site, we run a function that modifies the Site's directory file structure.
```
root@10alpha:/var/www/test.site# ls -la
total 52
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:03 .
drwxr-xr-x+ 6 root      root      4096 Jun 29 13:02 ..
drwxr--r--  3 root      root      4096 Jun 29 13:02 .duplicacy
-rw-r--r--  1 test10350 test10350  281 Jun 29 13:02 .user.ini
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 dns
lrwxrwxrwx  1 root      root        46 Jun 29 13:03 htdocs -> /var/www/test.site/releases/release-1656507770
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:03 logs
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 modsec
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 nginx
drwxr-xr-x  4 test10350 test10350 4096 Jun 29 13:02 public
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 releases
-rw-r--r--  1 test10350 test10350  120 Jun 29 13:02 user-configs.php
-rw-r--r--  1 test10350 test10350 4189 Jun 29 13:02 wp-config.php
```

A `/var/www/{site.domain}/releases` diretory is created, and within that a release `/var/www/{site.domain}/release-{EPOCH}` subdirectory.

The site's files and directories are moved into the release directory, and then `htdocs` is symlinked to the release directory.

The site `wp-config.php` file is symlinked into the `releases` directory so that is available.
```
root@10alpha:/var/www/test.site# ls -la releases
total 20
drwxr-xr-x  5 test10350 test10350 4096 Jun 29 13:16 .
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:16 ..
drwxr-xr-x  5 test10350 test10350 4096 Jun 29 13:03 release-1656507770
lrwxrwxrwx  1 root      root        32 Jun 29 13:16 wp-config.php -> /var/www/test.site/wp-config.php
```

You may have noticed the `.user.ini` file is now located in `/var/www/{site.domain}/.user.ini`, that is because it is now symlinked into the current release:
```
root@10alpha:/var/www/test.site# ls -la htdocs/.user.ini
lrwxrwxrwx 1 root root 28 Jun 29 13:03 htdocs/.user.ini -> /var/www/test.site/.user.ini
```
or
```
root@10alpha:/var/www/test.site# ls -la /var/www/test.site/releases/release-1656507770/.user.ini
lrwxrwxrwx 1 root root 28 Jun 29 13:03 /var/www/test.site/releases/release-1656507770/.user.ini -> /var/www/test.site/.user.ini
```
(If you are using OLS stack, this is also the case for the `.htaccess` file)

You will have also noticed the `/var/www/{site.domain}/public` directory. This directory is now for all public files, ie your uploads directory is symlinked here.
Files in this directory are not version controlled and they are not immutable.
```
root@10alpha:/var/www/test.site# ls -la htdocs/wp-content/uploads
lrwxrwxrwx 1 root root 25 Jun 29 13:03 htdocs/wp-content/uploads -> /var/www/test.site/public
```

At this point:
- The site's directory structure has been adapted for Full Git deployments, but the all files in all directories are the original files, your Git repo has not yet been deployed.
- The structure is also not immutable, all the files are owned by the system user, and file modifications are still allowed.

```
root@10alpha:/var/www/test.site# ls -la /var/www/test.site/releases/release-1656507770
total 220
drwxr-xr-x  5 test10350 test10350  4096 Jun 29 13:03 .
drwxr-xr-x  7 test10350 test10350  4096 Jun 29 13:28 ..
lrwxrwxrwx  1 root      root         28 Jun 29 13:03 .user.ini -> /var/www/test.site/.user.ini
-rw-r--r--  1 test10350 test10350   405 Jun 29 13:02 index.php
-rw-r--r--  1 test10350 test10350 19915 Jun 29 13:02 license.txt
-rw-r--r--  1 test10350 test10350  7401 Jun 29 13:02 readme.html
-rw-r--r--  1 test10350 test10350  7165 Jun 29 13:02 wp-activate.php
drwxr-xr-x  9 test10350 test10350  4096 Jun 29 13:02 wp-admin
-rw-r--r--  1 test10350 test10350   351 Jun 29 13:02 wp-blog-header.php
-rw-r--r--  1 test10350 test10350  2338 Jun 29 13:02 wp-comments-post.php
-rw-r--r--  1 test10350 test10350  3001 Jun 29 13:02 wp-config-sample.php
drwxr-xr-x  6 test10350 test10350  4096 Jun 29 13:03 wp-content
-rw-r--r--  1 test10350 test10350  3943 Jun 29 13:02 wp-cron.php
drwxr-xr-x 26 test10350 test10350 12288 Jun 29 13:02 wp-includes
-rw-r--r--  1 test10350 test10350  2494 Jun 29 13:02 wp-links-opml.php
-rw-r--r--  1 test10350 test10350  3973 Jun 29 13:02 wp-load.php
-rw-r--r--  1 test10350 test10350 48498 Jun 29 13:02 wp-login.php
-rw-r--r--  1 test10350 test10350  8577 Jun 29 13:02 wp-mail.php
-rw-r--r--  1 test10350 test10350 23706 Jun 29 13:02 wp-settings.php
-rw-r--r--  1 test10350 test10350 32051 Jun 29 13:02 wp-signup.php
-rw-r--r--  1 test10350 test10350  4748 Jun 29 13:02 wp-trackback.php
-rw-r--r--  1 test10350 test10350  3236 Jun 29 13:02 xmlrpc.php
```

### Directory Structure when Git Hybrid Repo is deployed

Once you click Deploy, or if you have push to deploy enabled and push an update, a deploy will be triggered on your server.

As this is a hybrid git repo, the scripts will detect the partial repo, and reconfigure the site directory structure.

The wp-core files will be placed back into `htdocs` and this directly will no longer be a symlink to a releases directory.
```
root@10alpha:/var/www/test.site# ls -la
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:31 .
drwxr-xr-x+ 6 root      root      4096 Jun 29 13:02 ..
drwxr--r--  3 root      root      4096 Jun 29 13:02 .duplicacy
-rw-r--r--  1 test10350 test10350  281 Jun 29 13:02 .user.ini
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 dns
lrwxrwxrwx  1 test10350 test10350   46 Jun 29 13:31 htdocs
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:31 logs
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 modsec
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 nginx
drwxr-xr-x  4 test10350 test10350 4096 Jun 29 13:02 public
drwxr-xr-x  8 test10350 test10350 4096 Jun 29 13:31 releases
-rw-r--r--  1 test10350 test10350  120 Jun 29 13:02 user-configs.php
-rw-r--r--  1 test10350 test10350 4321 Jun 29 13:31 wp-config.php
```

And all the WP core files remain belonging to the system user in `htdocs`
```
root@10alpha:/var/www/test.site# ls -la htdocs
total 220
drwxr-xr-x   5 test91975 test91975  4096 Jun 29 17:48 .
drwxr-xr-x+ 10 test91975 test91975  4096 Jun 29 17:49 ..
lrwxrwxrwx   1 root      root         28 Jun 29 17:48 .user.ini -> /var/www/test.site/.user.ini
-rw-r--r--   1 test91975 test91975   405 Jun 29 17:48 index.php
-rw-r--r--   1 test91975 test91975 19915 Jun 29 17:48 license.txt
-rw-r--r--   1 test91975 test91975  7401 Jun 29 17:48 readme.html
-rw-r--r--   1 test91975 test91975  7165 Jun 29 17:48 wp-activate.php
drwxr-xr-x   9 test91975 test91975  4096 Jun 29 17:48 wp-admin
-rw-r--r--   1 test91975 test91975   351 Jun 29 17:48 wp-blog-header.php
-rw-r--r--   1 test91975 test91975  2338 Jun 29 17:48 wp-comments-post.php
-rw-r--r--   1 test91975 test91975  3001 Jun 29 17:48 wp-config-sample.php
drwxr-xr-x   6 test91975 test91975  4096 Jun 29 17:48 wp-content
-rw-r--r--   1 test91975 test91975  3943 Jun 29 17:48 wp-cron.php
drwxr-xr-x  26 test91975 test91975 12288 Jun 29 17:48 wp-includes
-rw-r--r--   1 test91975 test91975  2494 Jun 29 17:48 wp-links-opml.php
-rw-r--r--   1 test91975 test91975  3973 Jun 29 17:48 wp-load.php
-rw-r--r--   1 test91975 test91975 48498 Jun 29 17:48 wp-login.php
-rw-r--r--   1 test91975 test91975  8577 Jun 29 17:48 wp-mail.php
-rw-r--r--   1 test91975 test91975 23706 Jun 29 17:48 wp-settings.php
-rw-r--r--   1 test91975 test91975 32051 Jun 29 17:48 wp-signup.php
-rw-r--r--   1 test91975 test91975  4748 Jun 29 17:48 wp-trackback.php
-rw-r--r--   1 test91975 test91975  3236 Jun 29 17:48 xmlrpc.php
```
The deploy will have deployed the hybrid repo to a releases subdirectory
```
root@10alpha:/var/www/test.site# ls -la releases/release-1656525881
total 24
drwxr-xr-x 5 root      root      4096 Jun 29 18:04 .
drwxr-xr-x 4 test91975 test91975 4096 Jun 29 18:04 ..
drwxr-xr-x 8 root      root      4096 Jun 29 18:04 .git
drwxr-xr-x 2 root      root      4096 Jun 29 18:04 .gpconfig
-rw-r--r-- 1 root      root        36 Jun 29 18:04 README.md
drwxr-xr-x 4 root      root      4096 Jun 29 18:04 wp-content
```
But the contents of the release directory have been copied to the `htdocs/wp-content` directory using rsync, and are no longer active on the site.
```
root@10alpha:/var/www/test.site# ls -la htdocs/wp-content
total 28
drwxr-xr-x 6 test91975 test91975 4096 Jun 29 18:04 .
drwxr-xr-x 5 test91975 test91975 4096 Jun 29 18:04 ..
-rw-r--r-- 1 test91975 test91975   28 Jun 29 17:48 index.php
drwxr-xr-x 2 test91975 test91975 4096 Jun 29 17:48 mu-plugins
drwxr-xr-x 5 test91975 test91975 4096 Jun 29 18:05 plugins
drwxr-xr-x 5 test91975 test91975 4096 Jun 29 18:04 themes
drwxr-xr-x 2 test91975 test91975 4096 Jun 29 17:48 upgrade
lrwxrwxrwx 1 root      root        25 Jun 29 18:04 uploads -> /var/www/test.site/public
```

Please note, even with Hybrid git deployments the uploads directory is still a symlink to the newly created site public directory. This makes it smoother if you wish to swap between different deployment types at a later date.

If you disable the git integration for your site, then the directory and file structure will be returned to GridPane normal using the latest files from the current release.









