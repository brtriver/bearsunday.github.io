---
layout: docs-en
title: Database
category: Manual
permalink: /manuals/1.0/en/database.html
---

# Databse

`Aura.Sql`、`Doctrine DBAL`, `CakeDB` modules are available for database connections.

# Aura.Sql

[Aura.Sql](https://github.com/auraphp/Aura.Sql) is an Aura database library that extends from `PDO` .

## Installation

Install `Ray.AuraSqlModule` via composer.

{% highlight bash %}
composer require ray/aura-sql-module
{% endhighlight %}

Installing `AuraSqlModule` in your application module`src/Module/AppModule.php`.

{% highlight php %}
<?php
use BEAR\AppMeta\AppMeta;
use BEAR\Package\PackageModule;
use Ray\AuraSqlModule\AuraSqlModule; // add this line
use Ray\Di\AbstractModule;

class AppModule extends AbstractModule
{
    protected function configure()
    {
        $this->install(new PackageModule));
        // Add the below install method call and contents
        $this->install(
            new AuraSqlModule(
                'mysql:host=localhost;dbname=test', 
                'username', 
                'password'
            )
        );
    }
}
{% endhighlight %}

Now the `DI` bindings are ready. The db object will be injected via a constructor or the `AuraSqlInject` setter trait.

{% highlight php %}
<?php

use Aura\Sql\ExtendedPdoInterface;

class Index
{
    public function __construct(ExtendedPdoInterface $pdo)
    {
        return $this->pdo; // \Aura\Sql\ExtendedPdo
    }
}
{% endhighlight %}


{% highlight php %}
<?php
use Ray\AuraSqlModule\AuraSqlInject;

class Index
{
    use AuraSqlInject; 
 
    public function onGet()
    {
        return $this->pdo; // \Aura\Sql\ExtendedPdo
    }
}
{% endhighlight %}

`Ray.AuraSqlModule` contains [Aura.SqlQuery](https://github.com/auraphp/Aura.SqlQuery) to help you build sql queries.


## Replication

Installing `AuraSqlReplicationModule` using a `connection locator` for master/slave connections.

{% highlight php %}
<?php
use Ray\Di\AbstractModule;
use Ray\AuraSqlModule\AuraSqlModule;
use Ray\AuraSqlModule\Annotation\AuraSqlConfig;
use Aura\Sql\ConnectionLocator;

class AppModule extends AbstractModule
{
    protected function configure()
    {
        $locator = new ConnectionLocator;
        $locator->setWrite('master', new Connection('mysql:host=localhost;dbname=master', 'id', 'pass'));
        $locator->setRead('slave1',  new Connection('mysql:host=localhost;dbname=slave1', 'id', 'pass'));
        $locator->setRead('slave2',  new Connection('mysql:host=localhost;dbname=slave2', 'id', 'pass'));
        $this->install(new AuraSqlReplicationModule($locator));
    }
}

{% endhighlight %}

You will now have a slave db connection when using HTTP GET, or a master db connection in other HTTP methods.

{% highlight php %}
<?php

use Aura\Sql\ExtendedPdoInterface;
use BEAR\Resource\ResourceObject;
use PDO;

class User extends ResourceObject
{
    public $pdo;

    public function __construct(ExtendedPdoInterface $pdo)
    {
        $this->pdo = $pdo;
    }

    public function onGet()
    {
         $this->pdo; // slave db
    }

    public function onPost($todo)
    {
         $this->pdo; // master db
    }
}
{% endhighlight %}

`$this->pdo` is overwritten if the method is annotated with`@ReadOnlyConnection` or`@WriteConnection`. The master / slave db connection corresponds to the annotation.

{% highlight php %}
<?php
use Ray\AuraSqlModule\Annotation\ReadOnlyConnection;  // important
use Ray\AuraSqlModule\Annotation\WriteConnection;     // important

class User
{
    public $pdo; // override when @ReadOnlyConnection or @WriteConnection annotated method called
    
    public function onPost($todo)
    {
         $this->read();
    }

    /**
     * @ReadOnlyConnection
     */
    public function read()
    {
         $this->pdo; // slave db
    }

    /**
     * @WriteConnection
     */
    public function write()
    {
         $this->pdo; // master db
    }
}
{% endhighlight %}

## Transactions

Using the `@Transactional` annotation wraps methods with a transaction.

{% highlight php %}
<?php
use Ray\AuraSqlModule\Annotation\Transactional;

// ....
    /**
     * @Transactional
     */
    public function write()
    {
         // \Ray\AuraSqlModule\Exception\RollbackException thrown if it failed.
    }
{% endhighlight %}

# Doctrine DBAL

[Doctrine DBAL](http://www.doctrine-project.org/projects/dbal.html) is also abstraction layer for database.

Install `Ray.DbalModule` with composer.

{% highlight bash %}
composer require ray/dbal-module
{% endhighlight %}

Install `DbalModule` in application module.

{% highlight php %}
<?php
use Ray\DbalModule\DbalModule;
use Ray\Di\AbstractModule;

class AppModule extends AbstractModule
{
    protected function configure()
    {
        $this->install(new DbalModule('driver=pdo_sqlite&memory=true');
    }
}
{% endhighlight %}

New DI bindings are now ready and `$this->db` can be injected with the `DbalInject` trait.

{% highlight php %}
<?php
use Ray\DbalModule\DbalInject;

class Index
{
    use DbalInject;
 
    public function onGet()
    {
        return $this->db; // \Doctrine\DBAL\Driver\Connection
    }
}
{% endhighlight %}

[MasterSlaveConnection](http://www.doctrine-project.org/api/dbal/2.0/class-Doctrine.DBAL.Connections.MasterSlaveConnection.html) is provided for master/slave connections.

# CakeDb

**CakeDb** is the database access module for the CakePHP3 Database library. This module is provided by [@lorenzo](https://github.com/lorenzo) ( original author of CakeDb).

Installing `Ray.CakeDbModule` with composer.

{% highlight bash %}
composer require ray/cake-database-module ~1.0

{% endhighlight %}

Then see more details at [Ray.CakeDbModule](https://github.com/ray-di/Ray.CakeDbModule) and [CakePHP3 Database Access & ORM](http://book.cakephp.org/3.0/en/orm.html).



