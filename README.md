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
