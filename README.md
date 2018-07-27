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
