# WordPress on GAE

## Bedrock initial setup

  Create an empty working directory and initialize a new roots/bedrock project in it.

    mkdir my-project
    cd my-project
    composer create-project roots/bedrock .

  Add recommended `.editorconfig` setup.

    # editorconfig.org

    root = true

    [*]
    indent_style = space
    indent_size = 2
    end_of_line = lf
    charset = utf-8
    trim_trailing_whitespace = true
    insert_final_newline = true

    [*.php]
    indent_size = 4

## Install plugins & drop-ins

  Define how to setup WP drop-ins.

    composer require koodimonni/composer-dropin-installer

  Add to `composer.json` key `extra`:

    "dropin-paths": {
      "web/app": [
        "package:wpackagist-plugin/memcached-redux:object-cache.php",
        "package:wpackagist-plugin/batcache:advanced-cache.php",
        "type:wordpress-dropin"
      ]
    }
  
  Require the basic modules.

    composer require frc/wp-harness \
      wpackagist-plugin/google-app-engine \
      wpackagist-plugin/batcache \
      wpackagist-plugin/memcached-redux \
      wpackagist-plugin/auth0

    composer require --dev google/appengine-php-sdk

  Ignore the two drop-ins in version control.

      echo '/advanced-cache.php' >>web/app/.gitignore
      echo '/object-cache.php' >>web/app/.gitignore

## Local database and user setup

  Create local MySQL database, eg.

    mysql -uroot <<EOD
    > create database wp;
    > create user 'wp' identified by 'wp';
    > grant all on wp.* to 'wp';
    > EOD

  Setup `.env` for local development:

    DB_NAME=wp
    DB_USER=wp
    DB_PASSWORD=wp
    DB_HOST=127.0.0.1
    WP_ENV=development
    WP_HOME=http://localhost:8080
    WP_SITEURL=${WP_HOME}/wp

  Initialize the database.

    vendor/bin/wp core install \
      --url=localhost --title=wp \
      --admin_user=admin \
      --admin_email=first.last@example.com

    Admin password: *****
    Success: WordPress installed successfully.

  Verify the installation with eg. `vendor/bin/wp db check`,
  list the plugins with `vendor/bin/wp plugin list` to verify
  all tooling and paths are in place. The output should look like this.

    +--------------------------+----------+--------+---------+
    | name                     | status   | update | version |
    +--------------------------+----------+--------+---------+
    | batcache                 | inactive | none   | 1.2     |
    | google-app-engine        | inactive | none   | 1.6     |
    | auth0                    | inactive | none   | 3.6.2   |
    | bedrock-autoloader       | must-use | none   | 1.0.0   |
    | disallow-indexing        | must-use | none   | 1.0.0   |
    | register-theme-directory | must-use | none   | 1.0.0   |
    | advanced-cache.php       | dropin   | none   |         |
    | object-cache.php         | dropin   | none   |         |
    +--------------------------+----------+--------+---------+

  Next step is to setup actual HTTP (development) server and
  continue plugin setup on the browser.

## Prepare the GAE development setup

  Install Google Cloud SDK. Download it from https://cloud.google.com/sdk/ and follow the setup instructions, or install it with `brew cask`.
  
    brew cask install google-cloud-sdk

  Create `app.yaml` that defines the application routes etc.

    runtime: php55
    api_version: 1

    handlers:
    # Wordpress core as-is
    - url: /wp/(.*\.(htm|html|css|js|ico|jpg|jpeg|png|gif|woff|ttf|otf|eot|svg))
      static_files: web/wp/\1
      upload: web/wp/.*\.(htm|html|css|js|ico|jpg|jpeg|png|gif|woff|ttf|otf|eot|svg)$
      application_readable: true

    # User content: themes, plugins, uploads
    - url: /app/(.*\.(htm|html|css|js|ico|jpg|jpeg|png|gif|woff|ttf|otf|eot|svg))
      static_files: web/app/\1
      upload: web/app/.*\.(htm|html|css|js|ico|jpg|jpeg|png|gif|woff|ttf|otf|eot|svg)$
      application_readable: true

    # Entry URL for wp-admin
    - url: /wp/wp-admin/
      script: web/wp/wp-admin/index.php
      secure: always

    # All other wp-admin components
    - url: /wp/wp-admin/(.+)
      script: web/wp/wp-admin/\1
      secure: always

    # Handle wp-login separately, requires https
    - url: /wp/wp-login.php
      script: web/wp/wp-login.php
      secure: always

    # Handle wp-cron separatelyu, only admin (see cron.yaml) can call
    - url: /wp/wp-cron.php
      script: web/wp/wp-cron.php
      login: admin

    # Handle all other WP core entrypoints
    - url: /wp/wp-(.+).php
      script: web/wp/wp-\1.php

    - url: /wp/xmlrpc.php
      script: web/wp/xmlrpc.php

    # Other URLs are handled by the main index.php (ie. bedrock),
    # which then loads the config, WP core etc.
    - url: /(.+)?/?
      script: web/index.php

Create `cron.yaml` eg.

    cron:
    - description: "wordpress cron tasks"
      url: /wp/wp-cron.php
      schedule: every 15 minutes

Create `php.ini` eg.

    google_app_engine.enable_functions = "php_sapi_name, gc_enabled"
    allow_url_include = "1"
    upload_max_filesize = "8M"
    google_app_engine.disable_readonly_filesystem = "1"
    ;google_app_engine.allow_include_gs_buckets = "1"

Start the development server.

    dev_appserver.py app.yaml

