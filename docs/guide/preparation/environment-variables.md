---
title: Environment variables
layout: doc
---

# Environment variables

Following the [12-factor app](https://12factor.net/config) methodology, and especially the [Configuration](https://12factor.net/config) section, environment variables should be used to store environment-specific configuration, such as database credentials, API keys, and other configuration values that change between environments.

## `config.php` / `core_config_data`

Magento / Adobe Commerce [allows to override any system configuration using environment variables](https://experienceleague.adobe.com/en/docs/commerce-operations/configuration-guide/deployment/examples/example-environment-variables), following the patterns below.

The slash character of config paths should be replaced with double underscores:

| Config path             | Environment variable                                     | Scope                   |
|-------------------------|----------------------------------------------------------|-------------------------|
| `dev/js/merge_files`    | `CONFIG__DEFAULT__DEV__JS__MERGE_FILES`                  | `default`               |
| `web/unsecure/base_url` | `CONFIG__STORES__MYSTORECODE__WEB__UNSECURE__BASE_URL`   | `mystorecode` store     |
| `web/secure/base_url`   | `CONFIG__WEBSITES__MYWEBSITECODE__WEB__SECURE__BASE_URL` | `mywebsitecode` website |

> [!WARNING] 
> These environment values only take precedence over values in `config.php` and the database (`core_config_data`).

> [!NOTE]
> You need to use encrypted values for sensitive configuration (ie. payment gateway credentials) environment variables.

> [!IMPORTANT]
> Magento / Adobe Commerce caches the configuration values. If you change the value of an environment variable, you need to clear the cache to see the changes.

In a regular Magento / Adobe Commerce deployment, you might need to define the following environment variables for your config, **in addition to the ones you've defined in your `env.php` file**:

| Variable name                                 | Description                                                 |
|-----------------------------------------------|-------------------------------------------------------------|
| `CONFIG__DEFAULT__WEB__UNSECURE__BASE_URL`    | Fully qualified base URL of your Magento, ending with a `/` |
| `CONFIG__DEFAULT__WEB__COOKIE__COOKIE_DOMAIN` | Cookie domain                                               |

### Elasticsearch / OpenSearch

Because Elasticsearch / OpenSearch settings are stored in the database, you need to define the following environment variables:

| Variable name                                                 | Description                                              | Default value             |
|---------------------------------------------------------------|----------------------------------------------------------|---------------------------|
| `CONFIG__DEFAULT__CATALOG__SEARCH__ENGINE`                    | Engine to use: `elasticsearch7` (legacy) or `opensearch` | `elasticsearch7`          |
| `CONFIG__DEFAULT__CATALOG__SEARCH__<ENGINE>__SERVER_HOSTNAME` | Server hostname or IP address                            | `127.0.0.1` / `localhost` |
| `CONFIG__DEFAULT__CATALOG__SEARCH__<ENGINE>__SERVER_PORT`     | Server port                                              | `9200`                    |
| `CONFIG__DEFAULT__CATALOG__SEARCH__<ENGINE>__INDEX_PREFIX`    | Indexes name prefix                                      | `magento2`                |
| `CONFIG__DEFAULT__CATALOG__SEARCH__<ENGINE>__ENABLE_AUTH`     | Enable basic authentication                              | `0`                       |
| `CONFIG__DEFAULT__CATALOG__SEARCH__<ENGINE>__USERNAME`        | Authentication username                                  |                           |
| `CONFIG__DEFAULT__CATALOG__SEARCH__<ENGINE>__PASSWORD`        | Authentication password                                  |                           |

> [!NOTE]
> Magento / Adobe Commerces uses different config paths for Elasticsearch and OpenSearch. You need to define the correct one for your environment, replacing `<ENGINE>` with `ELASTICSEARCH7` or `OPENSEARCH` in the environment variables.<br/>
> You may omit declaring some of these variables if you are using the default values.

## `env.php`

Magento / Adobe Commerce does not offer a mechanism similar to the one above for values of `env.php`.

Easy way to be able to define database connection info and such using environment variables, is taking advantage of PHP's `getenv()` function:

```php
<?php
    ...
    'db' => [
        'table_prefix' => '',
        'connection' => [
            'default' => [
                'host' => getenv('MAGENTO_DATABASE_HOST'),
                'dbname' => getenv('MAGENTO_DATABASE_NAME'),
                'username' => getenv('MAGENTO_DATABASE_USERNAME'),
                'password' => getenv('MAGENTO_DATABASE_PASSWORD'),
                'model' => 'mysql4',
                'engine' => 'innodb',
                'initStatements' => 'SET NAMES utf8;',
                'active' => '1'
            ]
        ]
    ],
    ...
```

You may also define defaults for those environment variables in your `env.php` file, using PHP's ternary operator `?:`:

```php
<?php
    ...
    'db' => [
        'connection' => [
            'default' => [
                'host' => getenv('MAGENTO_DATABASE_HOST') ?: 'localhost',
                'dbname' => getenv('MAGENTO_DATABASE_NAME') ?: 'magento',
                'username' => getenv('MAGENTO_DATABASE_USERNAME') ?: 'root',
                'password' => getenv('MAGENTO_DATABASE_PASSWORD') ?: 'root',
                ...
            ]
        ]
    ],
    ...
```

The following configuration sections should be updated to use environment variables:

* `db`
* `cache`
* `session`
* `queue`

See next page for examples.
