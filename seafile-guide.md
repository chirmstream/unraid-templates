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

## Troubleshooting

* ``CSRF Verification Failed`` - Starting in version 11, if you are behind a reverse proxy you'll get this error.  You need to make a modification to your ``seahub_settings.py`` file.  If you did not change the default location that would be under ``/mnt/user/seafile/seafile/conf/seahub_settings.py``.

    nano /mnt/user/seafile/seafile/conf/seahub_settings.py

and add ``CSRF_TRUSTED_ORIGINS = ["https://seafile.example.tld"]`` to the bottom of ``seahub_settings.py``, where ``seafile.example.tld`` is your domain.  Credit to GusFit on reddit for this [fix](https://www.reddit.com/r/unRAID/comments/1f4ebke/guide_for_installing_the_latest_version_of/?share_id=KqhG1zJ_OS02tbUWIEVL8&utm_content=2&utm_medium=ios_app&utm_name=ioscss&utm_source=share&utm_term=1).


## Advanced
### Migrating from Pro to Community edition
#### Step 1 
Read the [manual](https://manual.seafile.com/11.0/deploy_pro/migrate_from_seafile_community_server/) page, see the very bottom for how to go from Pro > Community.

#### Step 2
Backup your current Seafile instance just in case.

#### Step 3
Change your docker template repository to the community version on Dockerhub.  ``seafileltd/seafile-mc``  Make sure you specify the same major version in the docker tag of your current install.  Pull the new docker image.  If Seafile automatically runs the migration script this is all you'll need to do.

#### Step 4 
To manually run the migration script open a new unraid terminal and start a bash session inside the docker container.

    docker exec -t -i seafile /bin/bash

Make sure to change ``seafile`` to match the name of your docker container.  From here you can use typicall unix commands such as ``ls`` and ``cd`` to navigate inside the docker container.  

#### Step 5
Navigate to ``/opt/seafile/seafile-server-XX.XX.XX`` where ``XX.XX.XX`` is your current server version and stop the server.

    ./seafile.sh stop
    ./seahub.sh stop

Navigate to ``/opt/seafile/seafile-server-XX.XX.XX/upgrade`` and run the ``minor-upgrade.sh`` script.

    ./minor-upgrade.sh

Navigate back to ``/opt/seafile/seafile-server-XX.XX.XX/`` and start the server.

    ./seafile.sh start
    ./seahub.sh start

Your server should now be working again.  Personally, I've had issues with this if it is the FIRST time changing versions, it won't work.  For me, everytime I'm migrating for the first time, before running the ``minor-upgrade.sh`` script I would also need to run ``pro.py setup --migrate`` as well.

    seafile/seafile-pro-server-XX.XX.XX/pro/pro.py setup --migrate
