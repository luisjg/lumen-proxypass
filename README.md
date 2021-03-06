# Lumen Proxy Pass
Composer package for Lumen that resolves the correct absolute URLs when behind a proxy

This package is built for version 5.0 of Lumen and above. It modifies the functionality
of the `url()`, `asset()`, and other helper methods.

To install from Composer, use the following command:

```
composer require csun-metalab/lumen-proxypass
```

## Installation

First, add the following lines to your .env file to leverage the proxy attributes:

```
PROXY_ACTIVE=true
PROXY_PATH_HEADER=HTTP_X_FORWARDED_PATH
```

You may also add the following optional lines to your .env file to leverage the ability to FORCE the URL and schema without having to pass through a load balancer or proxy:

```
# http or https: PUBLIC_SCHEMA_OVERRIDE=https
PUBLIC_SCHEMA_OVERRIDE=

# Example: PUBLIC_URL_OVERRIDE=http://some-other-domain.example.com/some-other-directory
PUBLIC_URL_OVERRIDE=
```

Next, register the service provider and the configuration file in `bootstrap/app.php` as follows:

```
$app->configure('proxypass');
$app->register(CSUNMetaLab\LumenProxyPass\Providers\ProxyPassServiceProvider::class);
```

### Configuration File

If you do not already have a `config` directory in your project root, go ahead and create it.

In order to leverage the custom configuration values from this package, copy and paste the following code into a file called `proxypass.php` within your `config` directory in Lumen:

```
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Proxy Active?
    |--------------------------------------------------------------------------
    |
    | Determines whether the rewriting of the URLs is active. Default is true.
    |
    */
    'proxy_active' => env("PROXY_ACTIVE", true),

    /*
    |--------------------------------------------------------------------------
    | Public Absolute URL Override
    |--------------------------------------------------------------------------
    |
    | Overrides the root of the path generated by various methods to be the value
    | specified here when resolving the "public" directory. Methods include
    | asset(), url(), etc.
    |
    | I.E. A value of "http://localhost/someotherpath/public" would make the
    | methods start with "http://localhost/someotherpath/public" instead
    | of the typical "http://[domain]/[application path]/public" prefix.
    |
    */
    'public_url_override' => env("PUBLIC_URL_OVERRIDE", ""),

    /*
    |--------------------------------------------------------------------------
    | Public URL Schema Override
    |--------------------------------------------------------------------------
    |
    | Overrides the schema (http|https) that will be prefixed to all URLs.
    |
    */
    'public_schema_override' => env("PUBLIC_SCHEMA_OVERRIDE", ""),

    /*
    |--------------------------------------------------------------------------
    | Proxied Path Header
    |--------------------------------------------------------------------------
    |
    | The PHP-transformed name of the header that is passing along the rewritten
    | base URL from the proxy. Defaults to "HTTP_X_FORWARDED_PATH".
    |
    */
    'proxy_path_header' => env("PROXY_PATH_HEADER", "HTTP_X_FORWARDED_PATH"),

    /*
    |--------------------------------------------------------------------------
    | Trusted Proxies
    |--------------------------------------------------------------------------
    |
    | Comma-delimited list of the hostnames/IP addresses of proxy servers that
    | are allowed to manipulate the scheme and base URL. If no trusted proxies
    | have been set then proxying is allowed implicitly since this functions
    | as a whitelist.
    |
    */
    'trusted_proxies' => env("TRUSTED_PROXIES", ""),

];

?>

```

## Environment Variables

The two environment variables you added to your .env file are the following:

### PROXY_ACTIVE

Set this to `true` to enable the proxying functionality or `false` to disable it.

### PROXY_PATH_HEADER

This is the PHP-interpreted value of the request header sent from your proxy. The
default is `HTTP_X_FORWARDED_PATH` (the computed value of `X-Forwarded-Path`)

## Trusted Proxies

This package also has the ability to allow only certain proxy servers to modify
the necessary values in order to set the proper absolute URL.

By default, all proxy servers are allowed to modify the values; if, however,
the following value is set in your .env file then you can create a whitelist of proxies:

### TRUSTED_PROXIES

This is a comma-delimited list of hostnames/IP addresses that are allowed to
perform proxying functions.

```
TRUSTED_PROXIES=192.168.1.10,www.example.com,192.168.3.12
```

The above example would allow the following three proxy servers to provide
proxying functionality:

* 192.168.1.10 (would come from the `REMOTE_ADDR` value in PHP)
* www.example.com (would come from the `X-Forwarded-Server` header from the web server)
* 192.168.3.12 (would come from the `REMOTE_ADDR` value in PHP)

## Usage Example

Let's say you have an API hosted at `http://lumen.example.com` but that is
not the location you want to show to the world. Instead, you want to show a URL of
`http://www.example.com/lumen` so you place your Lumen API behind a proxy.

However, you notice that while the front page loads properly, none of the URLs you
have written with the `url()`, `asset()`, or other helpers work with that URL and instead
continue writing `http://lumen.example.com` as their base path.

You can configure your proxy to add a request header along with your `ProxyPass` and
`ProxyPassReverse` directives in Apache (ensure you have `mod_headers` enabled and
`mod_proxy` enabled as well):

```
ProxyPass        /lumen http://lumen.example.com
ProxyPassReverse /lumen http://lumen.example.com

<Location /lumen>
RequestHeader set X-Forwarded-Path "http://www.example.com/lumen"
</Location>
```

Now all of your URLs using the `url()`, `asset()`, and other helpers will be written
correctly!
