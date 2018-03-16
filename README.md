# php-centrifugo - version 1

PHP client for [Centrifugo](https://github.com/centrifugal/centrifugo) real-time messaging server

## Features

* Support publishing messages via Redis engine API listener (publish, broadcast, unsubscribe, disconnect methods only)
* Support transport chain (Redis + HTTP) as failover. If Redis down (or method not supported by Redis transport) client try send message via HTTP transport
* Support batch requests

### Attention!
> The library is based on the php extension [PhpRedis](https://github.com/phpredis/phpredis)


## Install

> composer require maksimusyan/php-centrifugo
```php
{
  "require": {
    "maksimusyan/php-centrifugo": "1.0.5"
  }
}
```
or the developer version directly from github:
```php
{
  "repositories": [
    {
      "type": "vcs",
      "url": "git://github.com/maksimusyan/php-centrifugo.git"
    }
  ],
  "require": {
    "maksimusyan/php-centrifugo": "dev-master"
  }
}
```

## Quick Examples

### Create Centrifugo client

```php
<?php

use Centrifugo\Centrifugo;

$endpoint = 'http://example.com/api/';
$secret = 'SECRET_API_KEY';

// From Redis transport
$centrifugo = new Centrifugo($endpoint, $secret, [
    'redis' => [
        'host'         => 'localhost',
        // additional params
        'port'         => 6379,
        'db'           => 0,
        'timeout'      => 0.0,
        'shardsNumber' => 0,
        'auth' => 'YOUR_REDIS_AUTH_TOKEN',
    ]
]);

// OR from CURL transport
$centrifugo = new Centrifugo($endpoint, $secret, [
    'http' => [
        // Curl options
        CURLOPT_TIMEOUT => 5,
    ],
]);

```

### Send request to Centrifugo

```php
<?php

use Centrifugo\Centrifugo;
use Centrifugo\Exceptions\CentrifugoException;

$userId = 1;
$channel = '#chan_1';
$messageData = ['message' => 'Hello, world!'];

try {
    //Send message into channel.
    $response = $centrifugo->publish($channel, $messageData);
    
    //Very similar to publish but allows to send the same data into many channels.
    $response = $centrifugo->broadcast($channels, $messageData);
    
    //Unsubscribe user from channel.
    $response = $centrifugo->unsubscribe($channel, $userId);
    
    //Disconnect user by user ID.
    $response = $centrifugo->disconnect($userId);
    
    //Get channel presence information (all clients currently subscribed on this channel).
    $response = $centrifugo->presence($channel);
    
    //Get channel history information (list of last messages sent into channel).
    $response = $centrifugo->history($channel);
    
    //Get channels information (list of currently active channels).
    $response = $centrifugo->channels();
    
    //Get stats information about running server nodes.
    $response = $centrifugo->stats();
    
    //Get information about single Centrifugo node.
    $response = $centrifugo->node('http://node1.example.com/api/');
} catch (CentrifugoException $e) {
    // invalid response
}
```

### Send batch request

```php
<?php

use Centrifugo\Centrifugo;
use Centrifugo\Exceptions\CentrifugoException;

$userId = '1'; //must be a string
$channel = '#chan_1';
$messageData = ['message' => 'Hello, world!'];

try {
    $requests[] = $centrifugo->request('publish', ['channel' => $channel, 'data' => $messageData]);
    $requests[] = $centrifugo->request('broadcast', ['channel' => $channel, 'data' => $messageData]);
    $requests[] = $centrifugo->request('unsubscribe', ['channel' => $channel, 'user' => $userId]);
    $requests[] = $centrifugo->request('disconnect', ['user' => $userId]);
    
    $batchResponse = $centrifugo->sendBatchRequest($requests);
    
    foreach ($batchResponse as $response) {
        if ($response->isError()) {
            // get error info
            $error = $response->getError();
        } else {
            // get response data as array
            $responseData = $response->getDecodedBody();
        }
    }
} catch (CentrifugoException $e) {
    // invalid response
}
```
