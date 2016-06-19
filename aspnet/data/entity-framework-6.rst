Getting Started with ASP.NET Core and Entity Framework 6
===========================================================

开始ASP.NET Core和Entity Framework 6
=========================================================


By `Paweł Grudzień`_ and `Damien Pontifex`_


作者： `Paweł Grudzień`_ 和 `Damien Pontifex`_

翻译： `魏美娟(初见) <http://github.com/ChujianA>`_


This article will show you how to use Entity Framework 6 inside an ASP.NET Core application.

本章将展示怎样在ASP.NET Core应用中使用Entity Framework 6

.. contents:: Sections:
  :local:
  :depth: 1
    
Prerequisites
-------------

前提
------
    
Before you start, make sure that you compile against full .NET Framework in your project.json as Entity Framework 6 does not support .NET Core. If you need cross platform features you will need to upgrade to `Entity Framework Core`_.

在你开始之前，在你的project.json 中要确保你的编译器支持全部.NET Framework。因为Entity Framework 6不支持.NET Core。如果你需要跨平台的功能，你将需要升级到 `Entity Framework Core`_ 。

In your project.json file specify a single target for the full .NET Framework:

在你的project.json文件中为全部 .NET Framework指定一条目标:


.. code-block:: json
    
    "frameworks": {
        "net46": {}
    }
    
Setup connection strings and dependency injection
-------------------------------------------------

设置连接字符串和依赖注入
-------------------------------------------------


The simplest change is to explicitly get your connection string and setup dependency injection of your ``DbContext`` instance. 

最简单的改变是在你的 ``DbContext`` 实例中明确地获取连接字符串，并设置依赖注入

In your ``DbContext`` subclass, ensure you have a constructor which takes the connection string as so:

在你的 ``DbContext`` 子类中，请确保你有获取连接字符串的构造函数，像这样：

.. code-block:: c#
    :linenos:
    
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(string nameOrConnectionString) : base(nameOrConnectionString)
        {
        }
    }

In the ``Startup`` class within ``ConfigureServices`` add factory method of your context with it's connection string. Context should be resolved once per scope to ensure performance and ensure reliable operation of Entity Framework. 

在 ``Startup`` 类里的 ``ConfigureServices`` 方法中添加带有连接字符串的context的工厂方法。Context应该在每个生命周期内解析一次以确保性能和Entity Framework的可操作性。

.. code-block:: c#
    :linenos:
    
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddScoped((_) => new ApplicationDbContext(Configuration["Data:DefaultConnection:ConnectionString"]));
        
        // Configure remaining services
    }

Migrate configuration from config to code
-----------------------------------------

将配置从config迁移到代码中
-----------------------------------------

Entity Framework 6 allows configuration to be specified in xml (in web.config or app.config) or through code. As of ASP.NET Core, all configuration is code-based.

Entity Framework 6允许在xml(如：web.config或者app.config)中或者通过代码指定配置。自从ASP.NET Core，所有的配置都是基于代码的。

Code-based configuration is achieved by creating a subclass of ``System.Data.Entity.Config.DbConfiguration`` and applying ``System.Data.Entity.DbConfigurationTypeAttribute`` to your ``DbContext`` subclass.

基于代码的配置是通过创建 ``System.Data.Entity.Config.DbConfiguration`` 的一个子类和将 ``System.Data.Entity.DbConfigurationTypeAttribute`` 应用到 ``DbContext`` 的子类中。 

Our config file typically looked like this:

我们的配置文件通常看起来像这样的：

.. code-block:: xml
    :linenos:
    
    <entityFramework>
        <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
            <parameters>
                <parameter value="mssqllocaldb" />
            </parameters>
        </defaultConnectionFactory>
        <providers>
            <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
        </providers>
    </entityFramework>

The ``defaultConnectionFactory`` element sets the factory for connections. If this attribute is not set then the default value is ``SqlConnectionProvider``. If, on the other hand, value is provided, the given class will be used to create ``DbConnection`` with its ``CreateConnection`` method. If the given factory has no default constructor then you must add parameters that are used to construct the object.

 ``defaultConnectionFactory`` 元素设置了链接工厂，如果这个属性没有被设置，那么默认值为``SqlConnectionProvider`` 。另一方面，如果设置值，给定的类将通过它的 ``CreateConnection`` 方法创建 ``DbConnection`` 。如果给定的工厂没有默认的构造函数，你必须添加用于构造对象的参数。

.. code-block:: c#
    :linenos:

    [DbConfigurationType(typeof(CodeConfig))] // point to the class that inherit from DbConfiguration
    public class ApplicationDbContext : DbContext
    {
        [...]
    }
    
    public class CodeConfig : DbConfiguration
    {
        public CodeConfig()
        {
            SetProviderServices("System.Data.SqlClient",
                System.Data.Entity.SqlServer.SqlProviderServices.Instance);
        }
    }
    
SQL Server, SQL Server Express and LocalDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SQL Server, SQL Server Express and LocalDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the default and so no explicit configuration is needed. The above ``CodeConfig`` class can be used to explicitly set the provider services and the appropriate connection string should be passed to the ``DbContext`` constructor as shown `above <#setup-connection-strings-and-dependency-injection>`_.

这是默认的，所以没有明确配置是必要的。以上 ``CodeConfig`` 类被用来显示设置提供程序服务和传递给 ``DbContext`` 构造函数适当的连接字符串。如 `above <#setup-connection-strings-and-dependency-injection>`_ 。

Summary
-------
概要
-------

Entity Framework 6 is an object relational mapping (ORM) library, that is capable of mapping your classes to database entities with little effort. These features made it very popular so migrating large portions of code may be undesirable for many projects. This article shows how to avoid migration to focus on other new features of ASP.NET.

Entity Framework 6是一个对象关系映射（ORM）类库，它有效的将类和数据库实体进行映射。这些特性使得它是非常受欢迎的，所以迁移大部分代码对于很多项目来说是不可取的。本文将介绍如何避免迁移，把重点放在ASP.NET的其他新的功能上。

Additional Resources
--------------------

额外资源
--------------------

- `Entity Framework - Code-Based Configuration <https://msdn.microsoft.com/en-us/data/jj680699.aspx>`_

- `Entity Framework - 基于代码的配置 <https://msdn.microsoft.com/en-us/data/jj680699.aspx>`_
