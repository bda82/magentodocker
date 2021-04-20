# Installing Magento in our container

First order of business is to create the Composer project:

```
docker exec -it magento-demo-web bash
```


```
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition /workspaces/magento-demo/install
```

NOTE: in this command you should use:
```"username": "<Public key>"```
```"password": "<Private key>"```   

This command will download all the Magento files as specified by the magento/project-community-edition project from the https://repo.magento.com/ repository. 
There are a few gotchas though:
1. First, Magento is not openly available to download just like that. As such, Composer will ask for authentication in order to do so. 
   Follow this guide [https://devdocs.magento.com/guides/v2.4/install-gde/prereq/connect-auth.html] to obtain the authentication keys from the Magento Marketplace. 
   When Composer asks for a username, type in the public key; when it asks for password, type in the private key.
2. Second, you’ll notice that I specified /workspaces/magento-demo/install at the end of that command. 
   This is where all the files will be downloaded. 
   I’ve chosen this (an install directory inside our current one) because composer create-project will refuse to download the files in a directory that’s not empty. 
   Ours isn’t, because we’ve got our Dockerfile in it. 
   But that’s nothing to worry about, once Composer finishes downloading everything, we’ll just copy the files over to their rightful location at /workspaces/magento-demo. 
   You can do so with some Linux sorcery like this:

   ```
   (shopt -s dotglob; mv -v /workspaces/magento-demo/install/workspaces/magento-demo/install/* .)
   ```

This Composer operation will take a good while, but when it’s done, make sure to move all the contents of /workspaces/magento-demo/install into /workspaces/magento-demo.

We now need to actually install Magento:
```
bin/magento setup:install \
  --base-url=http://localhost:5000 \
  --db-host=mysql \
  --db-name=magento_demo \
  --db-user=magento \
  --db-password=password \
  --admin-firstname=admin \
  --admin-lastname=admin \
  --admin-email=admin@admin.com \
  --admin-user=admin \
  --admin-password=admin123 \
  --language=en_US \
  --currency=USD \
  --timezone=America/New_York \
  --use-rewrites=1 \
  --elasticsearch-host=elasticsearch \
  --elasticsearch-port=9200
```

Once that command is done, we’re ready. We have a working Magento. Try it out by running this:

```
php -S 0.0.0.0:5000 -t ./pub/ ./phpserver/router.php
```

And navigating to ```localhost:5000```  in your browser. You should see your empty Magento homepage.

Optional: Installing the sample data:

If you’re planning some custom extension, or to just play with Magento to get to know it better, you may want to add some sample data. Luckily, the Magento devs have graciously provided such a thing in the form of a Composer package. If you want, you can install it with this recipe:

```
bin/magento sampledata:deploy
bin/magento setup:upgrade
bin/magento indexer:reindex
bin/magento cache:flush
```

```bin/magento sampledata:deploy```  will also ask you for your Magento Makerplace keys so have them ready.

Remove previous installation:

```
rm app/etc/env.php
```

You should set correct permissions for the whole Magento 2 installation directory by running the below command:

```
find . -type d -exec chmod 700 {} \; && find . -type f -exec chmod 600 {} \;
```