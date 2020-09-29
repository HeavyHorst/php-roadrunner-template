# php-roadrunner-template
OpenFaaS php roadrunner HTTP Template

This repository contains a PHP http-template for OpenFaaS that is based on https://github.com/spiral/roadrunner.

## Usage

```
faas template pull https://github.com/HeavyHorst/php-roadrunner-template
faas-cli new --lang php7-roadrunner function_name
```

You will find in the newly created directory my-function:

- worker.php : entrypoint
- php-extension.sh : is for installing PHP extensions if needed
- composer.json : is for dependency management

## Extra Extensions
You can refer to the PHP Docker image documentation for additional instructions on the installation and configuration of extensions

## Example
Edit the function_name/worker.php file to return some HTML:

```php
<?php
// worker.php
ini_set('display_errors', 'stderr');
include "vendor/autoload.php";

$relay = new Spiral\Goridge\StreamRelay(STDIN, STDOUT);
$psr7 = new Spiral\RoadRunner\PSR7Client(new Spiral\RoadRunner\Worker($relay));

while ($req = $psr7->acceptRequest()) {
    try {
	$resp = new \Zend\Diactoros\Response();
	$parameters = $req->getQueryParams();
        $resp->getBody()->write("Yeah, it's working!");
        $psr7->respond($resp);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}

```

## Deploy

`faas-cli up -f function_name.yml`
