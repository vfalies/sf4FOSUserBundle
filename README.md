FOSUserBundle is certainly the most popular bundle for Symfony to manage users.

With the last major version of Symfony (4), the organization of the code of the framework is a little different. The official documentation isn't clear about the process to install this bundle.

Below, you will found the necessary steps to use FOSUserBundle. You can get all sources of this installation on repository [sf4FOSUserBundle](https://github.com/vfalies/sf4FOSUserBundle) on github.

## Installation

Installation is a process in 7 steps:

1. Symfony 4 skeleton & required components installation
2. Download FOSUserBundle using composer
3. Create your User class
4. Configure your application's security.yml
5. Configure the FOSUserBundle
6. Import FOSUserBundle routing
7. Update your database schema

## Step 1: Symfony 4 skeleton & required components installation

### Symfony 4 skeleton

The first step is the installation of the symfony skeleton to start from a clean base.

```shell
composer create-project symfony/skeleton demo
```

!!! All next steps must be executed in the new created directory (aka. demo)

### Doctrine

Object-Relational-Mapper to manage database

```shell
composer require doctrine
```

At this step you must, as suggested by installer, modifying your `.env` configuration file and eventually `config/packages/doctrine.yaml`.
In `.env`, the string to change is like: 

```
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
```

You must indicate, the user, password, ip and name of the database.

### Annotations

Docblock Annotations Parser used with Doctrine

```shell
composer require annotations
```

### SwiftMailerBundle

Swiftmailer, free feature-rich PHP mailer in symfony bundle version

```shell
composer require swiftmailer-bundle
```

### Twig

Twig, the flexible, fast, and secure template language for PHP

```shell
composer require twig
```

### Optional : Web Server

Symfony WebServerBundle

```shell
composer require server --dev
```

## Step 2: Download FOSUserBundle using composer

```shell
composer require friendsofsymfony/user-bundle "~2.0"
```

At the end of the installation you will have the following error message :

!! The child node "db_driver" at path "fos_user" must be configured.

Don't panic, it's normal, you must now, create your User class and configured FOSUserBundle.

## Step 3: Create your User class

Create `src/Entity/User.php` as custom user class who extend the FOSUserBundle `BaseUser` class.

```php
<?php
// src/Entity/User.php

namespace App\Entity;

use FOS\UserBundle\Model\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="fos_user")
 */
class User extends BaseUser
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    public function __construct()
    {
        parent::__construct();
        // your own logic
    }
}
```

### Step 4: Configure your application's security.yml

Modify `config/packages/security.yaml` to setup FOSUserBundle security

```yaml
security:
    encoders:
        FOS\UserBundle\Model\UserInterface: bcrypt

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: ROLE_ADMIN

    # https://symfony.com/doc/current/security.html#where-do-users-come-from-user-providers
    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                csrf_token_generator: security.csrf.token_manager

            logout:       true
            anonymous:    true

    # Easy way to control access for large sections of your site
    # Note: Only the *first* access control that matches will be used
    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
```

### Step 5: Configure the FOSUserBundle

Create a new file `config/packages/fos_user.yaml` for the configuration of FOSUserBundle

```yaml
fos_user:
    db_driver: orm # other valid values are 'mongodb' and 'couchdb'
    firewall_name: main
    user_class: App\Entity\User
    from_email:
        address: "vincent@vfac.fr"
        sender_name: "vincent@vfac.fr"
```

Update `config/packages/framework.yaml` to add templating configuration

```yaml
framework:
    templating:
        engines: ['twig', 'php']
```

### Step 6: Import FOSUserBundle routing

Create `config/routes/fos_user.yaml`

```yaml
fos_user:
    resource: "@FOSUserBundle/Resources/config/routing/all.xml"
```

### Step 7: Update your database schema

If not already done, you must create your database

```shell
php bin/console doctrine:database:create
```

Update the schema with the informations from your User class entity

```shell
php bin/console doctrine:schema:update --force
```

At this point, all is installed and configured to use FOSUserBundle in Symfony 4.
Run the following command to check if all is ok

```shell
composer update
```

If you don't have any error message, you can test !

You can run the web server to test your application 

```shell
php bin/console server:start
```

