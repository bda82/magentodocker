# Installing Magento in container for demonstration and integration

Because is no one official Magento image exists, i try to create one from scratch.

## Start with configuration

First of all i investigate some tutorials and create Dockerfile and docker-compose configuration to combine:
1. Ubuntu image with php7.4 directly, because magento dont work with "latest" version of ubuntu php 8.
2. Mysql DB.
3. Elasticsearch.
4. Container nenwork.

Magento will work on 5000 port, mysql - on 3306, elasticsearch on 9200 and 9300.

MySql user will be: magento with password "password" and DB name will be magento_demo.

Next i run all in one:

```
docker-compose uo --build
```

## In-container work: install Magento components

To install Magento in container, i should enter it

```
docker exec -it magento-demo-web bash
```

Than i should run installation because Magento has composer package.

```
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition /workspaces/magento-demo/install
```

NOTE: in this command you should use:
```"username": "<Public key>"```
```"password": "<Private key>"```   

Those keys are in Magento official site Account section and we will use them in local installation because Magento package is proprietal and we should be secure in access to it.
This command will download all the Magento files as specified by the magento/project-community-edition project from the https://repo.magento.com/ repository. 
To obtain this keys i follow the next guide [https://devdocs.magento.com/guides/v2.4/install-gde/prereq/connect-auth.html].


Than, Magento placed all files in "install" folder and i should move them directly in our root folder (WORKDIR: /workspace/magento-demo) manually or with command:

```
mv ./install/* ./
```

## In-container work: install and initialize Magento package

We now need to actually install Magento like this

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

## In-container work: run Magento

Once that command is done, i am ready to rock and roll. Lets run the next command:

```
php -S 0.0.0.0:5000 -t ./pub/ ./phpserver/router.php
```

And than, i should navigating to ```localhost:5000```  in browser. I see empty Magento homepage.

## In-container work: installing the sample data:

Magento will empty, but i found some commands to install sample datas.
NOTE: This will also use secure magento repository and i will need auth with Publik and Private keys:

```
bin/magento sampledata:deploy
bin/magento setup:upgrade
bin/magento indexer:reindex
bin/magento cache:flush
```

## In-container work: look on admin URI

```
bin/magento info:adminuri
```

For example: 

```
magento@8e550a9e9b38:/workspaces/magento-demo$ bin/magento info:adminuri

Admin URI: /admin_1xbhov
```

Admin URL should be:

```
http://localhost:5000/admin_1xbhov
```

## In-container work: create a new admin user

```
bin/magento admin:user:create --admin-firstname=John --admin-lastname=Doe --admin-email=j.doe@example.com --admin-user=j.doe --admin-password=A0b9%t3g
```

## In-container work: disable 2FA if admin cant sign in

```
bin/magento mo:di Magento_TwoFactorAuth
```

## In-container work: if i need to remove old configuration at all

Remove previous installation:

```
rm app/etc/env.php
or 
rm /workspace/magento-demo
```

NOTE: I should set correct permissions for the whole Magento 2 installation directory by running the below command:

```
find . -type d -exec chmod 700 {} \; && find . -type f -exec chmod 600 {} \;
```

Profit...

## Install API2Cart bridge and integrate with Vincofy

1. Upload API2Cart bridge file (ex. magento_ai2cart_plugin.zip).
2. Enter into your shop hosting machine via SSH.
3. Place magento_ai2cart_plugin.zip file into temporary folder.
4. Unzip magento_ai2cart_plugin.zip into temporary folder.
5. Find Magento 2 server installation path and copy the archive content into shop folder ./app/code/<Plugin_Vendor>/<Plugin_Name>
6. Check vendor/name in plugin file composer.json in section "autoload" >> "psr-4" >> "...".
7. It should be like this: ""Api2cart\\BridgeConnector\\".
8. Set path to plugin folder like this: ./app/code/Api2cart/BridgeConnector/
9. Run command in server console from the Magento 2 server root folder: php bin/magento setup:upgrade
10. Run command php bin/magento setup:static-content:deploy -f
11. Open your Admin section of Magento 2 Web site.
12. Look on the left side menu bottom.
13. Click on API2CART BRIDGECONNECTOR.
14. Click on the blue button INSTALL CONNECTOR.
15. Copy secret code from the field YOUR STORE KEY.
16. Open the Vincofy Web site.
17. Sign in as Vendor.
18. Open My Shops menu in the left side menu.
19. Click Connect my store and select Magento option.
20. Fill all necessary fields, store name, store URL and insert the secret key (YOUR STORE KEY).
21. Click on Integrate this.
