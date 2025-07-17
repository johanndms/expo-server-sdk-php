# expo-server-sdk-php

Server-side library for working with Expo using PHP - Forked from ctwillie and added PHP 8 nullable requirements.

If you have any problems with the code in this repository, feel free to [open an issue](https://github.com/johanndms/expo-server-sdk-php/issues) or make a PR!

<details open="open">
<summary>Table of Contents</summary>

-   [expo-server-sdk-php](#expo-server-sdk-php)
    -   [Testing](#testing)
    -   [Installation](#installation)
    -   [Use Cases](#use-cases)
    -   [Composing a message](#composing-a-message)
    -   [Sending a push notification](#sending-a-push-notification)
    -   [Channel subscriptions](#channel-subscriptions)
    -   [Expo responses](#expo-responses)
    -   [Handling unregistered devices](#handling-unregistered-devices)
    -   [Retrieving push receipts](#retrieving-push-receipts)
    -   [Changelog](#changelog)
    -   [Contributing](#contributing)
    -   [License](#license)
    -   [Credits](#credits)

</details>

## Testing

You can run the test suite via composer:

```bash
composer test
```

## Installation

You can install the package via composer:

```bash
composer require johanndms/expo-server-sdk-php
```

## Use Cases

This package was written with two main use cases in mind.

1. Sending push notification messages to one or more recipients, then you're done! The most obvious use case.
2. And channel subscriptions, used to subscribe one or more tokens to a channel, then send push notifications to all tokens subscribed to that channel. Subscriptions are persisted until a token unsubscribes from a channel. Maybe unsubscribing upon the end users request.

Keep this in mind as you decide which is the best use case for your back end.

## Composing a message

Compose a push notification message to send using options from the [Expo docs](https://docs.expo.dev/push-notifications/sending-notifications/#message-request-format).

```php
use ExpoSDK\ExpoMessage;

/**
 * Create messages fluently and/or pass attributes to the constructor
 */
$message = (new ExpoMessage([
    'title' => 'initial title',
    'body' => 'initial body',
]))
    ->setTitle('This title overrides initial title')
    ->setBody('This notification body overrides initial body')
    ->setData(['id' => 1])
    ->setChannelId('default')
    ->setBadge(0)
    ->playSound();
```

## Sending a push notification

Compose a message then send to one or more recipients.

```php
use ExpoSDK\Expo;
use ExpoSDK\ExpoMessage;

/**
 * Composed messages, see above
 * Can be an array of arrays, ExpoMessage instances will be made internally
 */
$messages = [
    [
        'title' => 'Test notification',
        'to' => 'ExponentPushToken[xxxx-xxxx-xxxx]',
    ],
    new ExpoMessage([
        'title' => 'Notification for default recipients',
        'body' => 'Because "to" property is not defined',
    ]),
];

/**
 * These recipients are used when ExpoMessage does not have "to" set
 */
$defaultRecipients = [
    'ExponentPushToken[xxxx-xxxx-xxxx]',
    'ExponentPushToken[yyyy-yyyy-yyyy]'
];

(new Expo)->send($messages)->to($defaultRecipients)->push();
```

## Channel subscriptions

Subscribe tokens to a channel, then push notification messages to that channel. Subscriptions are persisted internally in a local file so you don't have to worry about this yourself. Unsubscribe the token from the channel at any time to stop messages to that recipient.

> :warning: **If you are are running multiple app servers**: Be very careful here! Channel subscriptions are stored in an internal local file. Subscriptions will not be shared across multiple servers.

```php
/**
 * Specify the file driver to persist subscriptions internally.
 */
use ExpoSDK\Expo;

$expo = Expo::driver('file');

// composed message, see above
$message;

$recipients = [
    'ExponentPushToken[xxxx-xxxx-xxxx]',
    'ExponentPushToken[yyyy-yyyy-yyyy]'
];

// name your channel anything you'd like
$channel = 'news-letter';
// the channel will be created automatically if it doesn't already exist
$expo->subscribe($channel, $recipients);

$expo->send($message)->toChannel($channel)->push();

// you can unsubscribe one or more recipients from a channel.
$expo->unsubscribe($channel, $recipients);
```

## Expo responses

Get the data returned from successful responses from the Expo server.

```php

$response = $expo->send($message)->to($recipients)->push();

$data = $response->getData();
```

## Handling unregistered devices

Expo provides a macro for handling tokens that have DeviceNotRegistered error in the Expo response. You can register a callback before sending your messages to handle these unregistered tokens.

You only need to register the handler once as it will be applied to all Expo instances.

```php
use ExpoSDK\Expo;

Expo::addDevicesNotRegisteredHandler(function ($tokens) {
    // this callback is called once and receives an array of unregistered tokens
});

$expo1 = new Expo();
$expo1->send(...)->push(); // will call your callback

$expo2 = new Expo();
$expo2->send(...)->push(); // will also call your callback
```

## Retrieving push receipts

Retrieve the push receipts using the ticket ids from the Expo server.

```php
$ticketIds = [
    'xxxx-xxxx-xxxx-xxxx',
    'yyyy-yyyy-yyyy-yyyy'
];

$response = $expo->getReceipts($ticketIds);
$data = $response->getData();
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](.github/CONTRIBUTING.md) for details.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

## Credits

-   [Cedric Twillie](https://github.com/ctwillie)
-   [All Contributors](../../contributors)

