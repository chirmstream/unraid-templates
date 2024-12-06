# Seafile Guide (unRAID)

## Step 1 - System prep
In the webui navigate to Settings>Docker and enable "Preserve user defined networks"  You will have to disable docker and switch to the advanced view.
<img src="/screenshots/docker-network-settings.png?ref_type=heads">
Re-enable docker and open a new unRAID terminal and create a new docker network.  In this example I use ``seafile-net`` but you can change it if you wish.

    docker network create seafile-net

Verify that the docker network was successfully created

    docker network list

## Step 2 - Install database(s)
In the Mariadb template settings change the network type to ``seafile-net``
<img src="/screenshots/docker-network-type.png?ref_type=heads">
* I've been using Mariadb for my database and had good success.  I believe you can use your choise of SQL server database but the only one I've tested is Mariadb.  Seafile can utilize memcached, but I have not tried setting it up, as I've been happy with the performance with only Mariadb.  
* I recomend the ``linuxserver`` Mariadb container.
* The root password will be required for the seafile template later

Go ahead and remove ``MYSQL_DATABASE``, ``MYSQL_USER``, ``MYSQL_PASSWORD``, and ``REMOTE_SQL``.  Seafile will connect to Mariadb with the root password and create all necessary users and databases.

Don't forget to change your appdata path if want the database to be stored in a different location that default.
<img src="/screenshots/database-template.png?ref_type=heads">  

If you have other Mariadb containers on your server, you'll want to rename this one.  Mine is named ``seafile-mariadb``.

## Step 3 - Install seafile
Make sure to set the network type to ``seafile-net`` for this template as well.  ``SQL Host`` will be the name of your database container, in my case I called it ``seafile-mariadb``.  Here is where you will need the SQL root password.

## Step 3a - Seafile Pro setup (optional)
If you want to use seafile professional (free for up to 3 users) instead of the community edition you will need to create at [https://customer.seafile.com/](https://customer.seafile.com/).  You will be given a username and password for seafile's private docker registry.  To save the credentials in unRAID open a new terminal and type:

    docker login docker.seadrive.org

Enter your given username and password.

Edit the seafile template repository to ``docker.seadrive.org/seafileltd/seafile-pro-mc:<tag>-latest``.  Where <tag> is the latest version available.  Currently seafile 11 is the latest release, and the respective tag used would be ``11.0-latest``.  So the entire repository would be set to ``docker.seadrive.org/seafileltd/seafile-pro-mc:11.0-latest``

You can also go to https://docker.seadrive.org and browse the images to find valid tags.

## Step 4 - Configure seafile
You should now be able to see the login screen.  Go ahead and login with the admin credentials setup in the docker template
<img src="/screenshots/seafile-login.png?ref_type=heads">  

If you're going to run seafile behind a reverse proxy you will need to set ``SERVICE_URL`` and ``FILE_SERVER_ROOT`` located in the ``System Admin`` webui section.
<img src="/screenshots/seafile-admin-settings.png?ref_type=heads">

Whatever you have set here should be the same as ``SEAFILE_SERVER_HOSTNAME`` in the seafile template.  If you are using letsencrypt with a reverse proxy make sure to use ``https``.

## Notes

* ``CSRF Verification Failed`` - Starting in version 11, if you are behind a reverse proxy you;ll get this error.  You need to make a modification to your ``seahub_settings.py`` file.  If you did not change the default location that would be under ``/mnt/user/seafile/seafile/conf/seahub_settings.py``.

    nano /mnt/user/seafile/seafile/conf/seahub_settings.py

and add ``CSRF_TRUSTED_ORIGINS = ["https://seafile.example.tld"]`` to the bottom of ``seahub_settings.py``, where ``seafile.example.tld`` is your domain.  Credit to GusFit on reddit for this [fix](https://www.reddit.com/r/unRAID/comments/1f4ebke/guide_for_installing_the_latest_version_of/?share_id=KqhG1zJ_OS02tbUWIEVL8&utm_content=2&utm_medium=ios_app&utm_name=ioscss&utm_source=share&utm_term=1).
