# LDAP Authenticate plugin
LDAP Authenticate Plugin for CakePHP 3.x and AuthComponent.
based on the excellent work of queencitycodefactory/ldap

**This branch is for CakePHP 3.6+**

## Requirements
* CakePHP 3.x (updated to CakePHP 3.6)
* php5-ldap module

## Installation

You can install this plugin into your CakePHP application using [composer](http://getcomposer.org).

The recommended way to install composer packages is:

```
composer require rubyan/ldap
```

You can also add `"rubyan/ldap" : "dev-master"` to `require` section in your application's `composer.json`.

Enable ldap in php.ini:
```
extension=php_ldap.dll
```
## Usage

In your app's `src/Application.php` in function: `bootstrap()` add:
```
$this->addPlugin('Rubyan/LDAP');
```
## Configuration:

Setup the authentication class settings

### AppController Setup:

```php
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('Flash');
        $this->loadComponent('Auth', [
            'loginAction' => [
                'controller' => 'Users',
                'action' => 'login'
            ],
            'authError' => 'Insufficient privileges to view requested resources. Please login to continue!',
            'authenticate' => [
                'Rubyan/LDAP.Ldap' => [
                    'fields' => [
                        'username' => 'username',
                        'password' => 'password'
                    ],
                    'port' => Configure::read('Ldap.port'),
                    'host' => Configure::read('Ldap.host'),
                    'domain' => Configure::read('Ldap.domain'),
                    'baseDN' => Configure::read('Ldap.baseDN'),
                    'search' => Configure::read('Ldap.search'),
                    'errors' => Configure::read('Ldap.errors'),
                    'flash' => [
                        'key' => 'ldap',
                        'element' => 'Flash/error',
                    ]
                ]
            ]
        ]);
    }
```

### Setting the Base LDAP settings

config/app.php:
```php
    /**
     * LDAP Configuration.
     *
     * Contains an array of settings to use for the LDAP configuration.
     *
     * ## Options
     *
     * - `domain` - The domain name to match against or auto complete so user isn't
     *    required to enter full email address
     * - `host` - The domain controller hostname. This can be a closure or a string.
     *    The closure allows you to modify the rules in the configuration without the
     *    need to modify the LDAP plugin. One host (string) should be returned when
     *    using closure. You can find your ldap servers with the following command:
     *      host -t srv _ldap._tcp.YOURDOMAIN.LOCAL
     * - `port` - The port to use. Default is 389 and is not required.
     * - `search` - The attribute to search against. Usually 'UserPrincipalName'
     * - `baseDN` - The base DN for directory - Closure must be used here, the plugin
     *    is expecting a closure object to be set.
     * - `attributes` - An array of the required attributes, e.g. ["mail", "sn", "cn"]. 
     *    Note that the "dn" is always returned irrespective of which attributes types are 
     *    requested.
     * - `errors` - Array of errors where key is the error and the value is the error
     *    message. Set in session to Flash.ldap for flashing
     *
     * @link http://php.net/manual/en/function.ldap-search.php - for more info on ldap search
     */
    'Ldap' => [
        'domain' => 'domain.local',
        'host' => function() {
            $hosts = [
                'host1.domain.local', 
                'host2.domain.local'
            ];
            shuffle($hosts);
            return $hosts[0];
        },
        'port' => 389,
        'search' => function($username, $domain) {
            if (strpos($username, $domain) !== false) {
                // remove the @domain from username 
                $username = str_replace('@' . $domain, '', $username);
            }
            $search = '(&(objectCategory=person)(samaccountname=' . $username. '))';
            return $search;
        },           
        'baseDN' => function($username, $domain) {
            if (strpos($username, $domain) !== false) {
                $baseDN = 'OU=Domain,DC=domain,DC=local';
            } else {
                $baseDN = 'CN=Users,DC=domain,DC=local';
            }
            return $baseDN;
        },
        'attributes' => ['samaccountname','mail', 'displayname'],
        'errors' => [
            'data 773' => 'Some error for Flash',
            'data 532' => 'Some error for Flash',
        ]
    ],

```
