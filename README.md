# Authenticator

This is a dbo authenticator based on the Zend Framework.

## Installation
It's recommended that you use [Composer](https://getcomposer.org/) to install

```bash
$ composer require neilmillard/authenticator
```

## Usage

For use with Slim 3 (for instance)

```php
// database mysqli connection
$container['database'] = function ($c) {
    $settings = $c['settings']['database'];
    $connection = new \PDO($c['dsn'],$settings['username'],$settings['password']);
    return $connection;
};

// authentication
$container['authenticator'] = function ($c) {
    $settings = $c['settings']['authenticator'];
    $connection = $c['database'];
    $adapter = new \App\Authentication\Adapter\Db\EloAdapter(
        $connection,
        $settings['tablename'],
        $settings['usernamefield'],
        $settings['credentialfield']
    );
    $authenticator = new \App\Authentication\Authenticator($adapter);
    return $authenticator;
};
```

middleware is provided.

```php
$container['Authenticator\Middleware'] = function ($c) {
    return new \App\Authentication\Middleware($c['authenticator'],$c['router'],$c['view']);
};

//authenticator to populate twig view
$app->add('Authenticator\Middleware:addViewData');
```

in the routes add in the middleware

```php
$app->get('/profile', 'App\Action\ProfileAction:dispatch')
    ->setName('profile')
    ->add('Authenticator\Middleware:auth');
```

login

```php
$result = $this->authenticator->authenticate($username, $password);
if ($result->isValid()){
    // Success
} else {
    // Authentication error
    $messages = $result->getMessages();
    $error=(string) $messages[0];
}
```

and to logout

```php
$this->authenticator->clearIdentity();
```