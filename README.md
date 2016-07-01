# Firebase Token Generator - PHP

[Firebase Custom Login](https://www.firebase.com/docs/web/guide/simple-login/custom.html)
gives you complete control over user authentication by allowing you to authenticate users
with secure JSON Web Tokens (JWTs). The auth payload stored in those tokens is available
for use in your Firebase [security rules](https://www.firebase.com/docs/security/api/rule/).
This is a token generator library for PHP which allows you to easily create those JWTs.


## Dependencies

The Firebase PHP token generator library depends on [PHP-JWT](https://github.com/firebase/php-jwt).


## A Note About Security

**IMPORTANT:** Because token generation requires your Firebase Secret, you should only generate
tokens on *trusted servers*. Never embed your Firebase Secret directly into your application and
never share your Firebase Secret with a connected client.

## Installation

Using composer:

```
composer require firebase/token-generator
```

## Generating Tokens

- [Version 3.x of the Firebase SDK](#version-3x-of-the-firebase-sdk)
- [Versions 1.x and 2.x of the Firebase SDK](#versions-1x-and-2x-of-the-firebase-sdk)
  - [Token Options](#token-options)

### Version 3.x of the Firebase SDK

To create custom tokens with the Firebase server SDKs, you must have a service account. 
Follow the [server SDK setup instructions](https://firebase.google.com/docs/server/setup/)
for more information on how to initialize your server SDK with a service account.

Once you have downloaded the service account's key file, you can generate a custom token with
this snippet of PHP code:

```php
use Firebase\Token\V3\ServiceAccount;
use Firebase\Token\V3\TokenGenerator;

try {
    $serviceAccount = ServiceAccount::fromKeyFile('/path/to/key_file.json');
    // The $enableDebug parameter is optional and defaults to false
    $generator = new TokenGenerator($serviceAccount, $enableDebug = true);
    
    // The $claims parameter is option and defaults to an empty array
    $token = $generator->createCustomToken('custom_uid', $claims = ['foo' => 'bar']);
    
    echo $token;
} catch (\Exception $e) {
    echo 'Error: '.$e->getMessage();
    exit;
}
```

### Versions 1.x and 2.x of the Firebase SDK

To generate tokens, you'll need your Firebase Secret which you can find by entering your Firebase
URL into a browser and clicking the "Secrets" tab on the left-hand navigation menu.

Once you've downloaded the library and grabbed your Firebase Secret, you can generate a token with
this snippet of PHP code:

```php
use Firebase\Token\TokenException;
use Firebase\Token\TokenGenerator;

try {
    $generator = new TokenGenerator('<YOUR_FIREBASE_SECRET>');
    $token = $generator
        ->setData(array('uid' => 'exampleID'))
        ->create();
} catch (TokenException $e) {
    echo "Error: ".$e->getMessage();
}

echo $token;
```

The payload passed to `setData()` is made available for use within your
security rules via the [`auth` variable](https://www.firebase.com/docs/security/api/rule/auth.html).
This is how you pass trusted authentication details (e.g. the client's user ID)
to your Firebase security rules. The payload can contain any data of your
choosing, however it must contain a "uid" key, which must be a string of less
than 256 characters. The generated token must be less than 1024 characters in
total.


#### Token Options

Token options can be set to modify how Firebase treats the token. Available options are:

* **expires** (number or DateTime) - A timestamp (as number of seconds since the epoch) or a `DateTime`
denoting the time after which this token should no longer be valid.

* **notBefore** (number or DateTime) - A timestamp (as number of seconds since the epoch) or a `DateTime`
denoting the time before which this token should be rejected by the server.

* **admin** (boolean) - Set to `True` if you want to disable all security rules for this client.
This will provide the client with read and write access to your entire Firebase database.

* **debug** (boolean) - Set to `True` to enable debug output from your security rules. You should
generally *not* leave this set to `True` in production (as it slows down the rules implementation
and gives your users visibility into your rules), but it can be helpful for debugging.

Here is an example of how to set options:

```php
use Firebase\Token\TokenGenerator;

$generator = new TokenGenerator('<YOUR_FIREBASE_SECRET>');

// Using setOption()
$token = $generator
    ->setOption('admin', true)
    ->setOption('debug', true)
    ->setData(array('uid' => 'exampleID'))
    ->create();

// Using setOptions()
$token = $generator
    ->setOptions(array(
        'admin' => true,
        'debug' => true
    ))
    ->setData(array('uid' => 'exampleID'))
    ->create();
```

## Changelog

#### 3.0.0 - 2015-11-18
- Update PHP-JWT to >= 3.0
- Remove deprecated `Services_FirebaseTokenGenerator` and update tests
- Thanks to [@jeromegamez](https://github.com/jeromegamez) for the above contributions!

#### 2.1.0 - 2015-06-22
- Update the minimum required PHP version to >= 5.4.
- [Major package overhaul including API improvements](https://github.com/firebase/firebase-token-generator-php/pull/18)
thanks to [@jeromegamez](https://github.com/jeromegamez)!

#### 2.0.1 - 2015-04-02
- Specifying the PHP-JWT version more specifically.

#### 2.0.0 - 2014-09-15
- Additional validation to ensure tokens contain a "uid" field unless they have
the "admin" option set to `true`.

#### 1.0.0 - 2014-09-04
- Initial release
