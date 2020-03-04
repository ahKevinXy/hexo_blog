---
title: laravel5.5手册
date: 2020-01-05 06:04:53
tags:
---
### laravel 请求生命周期

* 简介
* 生命周期概述
* 聚集服务提供者

#### 简介

    
##### 生命周期概述

##### 开始
***
  
    
 public/index.php 文件是所有对laravel应用程序的请求入口点.所有的web服务器通过配置引导这个文件.文件不包含太多的代码,却是加载框架的起点
    
    
 ##### Http /控制器内核
 
***
根据进入应用程序的请求类型来将传入的请求发送到http内核或控制台内核.而这两个内核是用来作为所有请求都要通过的中心位置.app/Http/Kernel.php中的HTTP内核Http内核继承了Illuminate\Foundation\Http\Kernel 类,它定义了执行请求之前的运行的bootstrappers 数组.这个数组负责在实际处理请求之前完成这些内容:配置错误处理,配置日志记录,Http内核还定义了所有请求被应用程序处理之前必须经过的http中间件的列表.这些中间件处理http会话的读写,确定应用程序是否处于维护模式,验证csrf令牌


##### 服务提供器

***

最重要的内核引导操作之一是加载应用程序的服务提供器.应用程序的所有服务提供器都在config/app.php配置文件的providers数组配置.首先所有服务器都会调用register方法,接着由boot方法负责调用所有被注册提供器.服务提供器负责引导所有框架的各种组件,如数据库,队列,验证和路由组件,也就是说,框架提供的每个功能都由它们来引导并配置
    
##### 分配请求

***
 一旦引导了应用程序且注册所有服务提供者,Request 请求就会被转发给路由器来进行调度.路由器将请求发送到路由或控制器或任何运行与路由的特定组件
    
 
 
 #### 服务器容器解析
 
 ##### 简介 

***
   laravel 服务容器是用于管理类的依赖和执行依赖注入的工具.依赖注入这个花俏名词实质是指:类的依赖项通过构造函数,或者某些情况下通过setter 方法注入到类中
    
  
```php
    
    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
    
        protected $users;
        
        
        public function __construct($UserRepository $users)
        {
            $this->users = $users;
        }
        
        public function show($id)
        {
            $user = $this->users->find($id);
            
            return view('user.profile',['user'=>$user]);
        }
    }


```

***
   在这个例子中,控制器UserController 需要从数据源中获取users,因此,我们要注入可以获取users的服务器,在这种情况下UserRepository 可能是使用Eloquent 从数据库中获取user信息,因为Respository 是通过UserRepository注入的,所以我们可以轻易的将其切换为另一个实现,这种注入方式便利之处体现我们为应用编写测试时,我们还可以轻松的模拟或创建UserRepository 的虚拟实现
   
   ##### 绑定
   
   ##### 绑定基础
  *** 
   因为几乎所有服务容器都是在服务提供器中注册绑定的,
   
        所有类没有依赖任何接口,就没有必要将绑定到容器中,容器不需要指定如何构建这些对象,因为它可以使用反射自动解析这些对象
        
        
  ##### 简单绑定
  ***
   在服务提供器中,可以通过$this->app 属性服务容器,我们可以通过bind方法注册绑定,传递我们想要注册的类或接口名称再返回类的实体Closure
  ```php

    $this->app->bind("HelpSpot\API",function($app){
        return new HelpSpot\API($app->make("HttpClient"));
    });
```
  
  ##### 绑定一个单例
  ***
   singleton 方法将类或接口绑定到只能解析一次的容器中.绑定的单例被解析后,相同的对象实例会在随后的调用中返回到容器中
  ```php

    $this->app->singleton("HelpSpot\API",function($app){
    
        return new HelpSpot\API($app->make('HttpClient'));
        });
    
```

##### 绑定实例

***
可以使用instance 方法将现有对象实例绑定到容器中,给定的实例会始终在随后的调用中返回到容器中

```php
    $api = new HelpSpot/API(new Client());
    
    $this->instance('HelpSpot',$api);

```



##### 绑定初始值
***
当有一个类不仅需要接受一个注入类,还需要注入一个基本值,你可以使用上下文绑定来轻松注入你的类需要任何值:

```php

    $this->app->where('App\Http\Controller\UserController')
        ->needs('$variableName')
        ->give($value);
        
```


#### 服务提供者


##### 简介

***

 服务提供器是所有laravel 应用程序引导中心.应用程序以及laravel 的所有核心服务都是通过提供器进行引导
 在这里,引导其实指注册
 laravel config/app.php 文件中有一个providers 数组.数组中的内容是应用程序要加载的所有服务提供类.这其中有许多提供器并不会在每次请求时都被加载,只有当它们提供的服务实际需要才会加载,这种称为延迟提供器
 
 
 ##### 编写服务提供器
 ***
 所有服务提供器都继承Illuminate\Support\ServiceProvider 类大多数服务提供器都包含register 和boot 方法.在register 方法中,只需要绑定类到服务容器中/而不是尝试在register方法中注册任何事件监听器,路由或任何其它功能
 使用Artisan 命令 make:provider 命令生成一个新的提供器
 
    php artisan make:provider ServiceProvider
    
    
 > Riak Google 开源的分布式 key-value nosql 
 ```php
    namespace App\Providers;
    
    use Riak\Connction;
    use Illuminate\Support\ServiceProvider;
    
    class RiakServiceProvider extends ServiceProvider
    {
        public function register()
        {
            $this->app->singleton(Connection::class,function($app){
                return new Connection(config('riak'));
            })
        }
    }
    

```

这个服务提供器只定义了一个register 方法,并使用该方法在服务容器中定义了一个Riak\Connection 

##### 引导方法

那么,如果需要在服务提供器中注册一个视图组件,这一个在boot 方法中完成.此方法在所有其它服务提供器都注册后才能提供调用,这意味着可以访问已经被框架注册的所有服务;

```php
    namespace App\Providers;
    
    use Illuminate\Support\ServiceProvider;
    
    class ComposerServiceProvider extends ServiceProvider
    {
        view()->composer('view',function()
        {
            //
        });
    }

```

##### 引导方法依赖注入
***
可以为服务提供器的boot 方法设置类型提示.服务器会自动注入你需要的任何依赖项

```php
    use Illuminate\Contracts\Routing\ResponseFactory;
    
    public function boot(ResponseFactory $response)
    {
        $response->macro('caps',function($value)
        {
        
        }
        );
    }

```

##### 注册提供器

所有服务器都在config/app.php 配置文件中注册.该文件中有一个providers数组,用于存放服务器提供器的类名.默认情况下,这个数组列出了一系列Laravel核心服务提供器.这些服务提供器引导Laravel核心组件



##### 延迟提供器

***

如果提供器仅在服务容器中注册绑定,就可以选择延迟其注册,直到当它真正需要注册绑定.推迟加载这种提供器会提高应用程序的性能,因为它不会在每次请求时都从文件系统中加载,laravel编译并保持延迟服务提供器的所有服务的列表,以及其服务提供器类的名称,因此,只有当你在尝试解析其中一项服务时,laravle才会加载服务提供器,要延迟提供器的加载,将defer属性设置为true,并定义provider方法.providers 方法应该返回提供器注册的服务器绑定

```php
   <?php
    namespace App\Providers;
    
    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;
    
    class RiakServiceProvider extends ServiceProvider
    {
        protected $defer = true;
        
        public function register()
        {
            $this->app->singleton(Connection::class,function($app){
                return new Connection($app['config']['riak'])
            });
        }
        
        public function providers()
        {
            return [Connection::class];
        }
    }

```


#### Facades
***
##### 简介

***

Facades 为应用程序的服务容器中可用的类提供了一个静态接口.Laravel自带了很多Facades,可以访问绝大部分是Laravel的功能.Laravel Facades实际上是服务容器中底层类的静态代理,它提供了简洁而富有表现力的语法,甚至比传统的静态方法更具可测试性和扩展性. 所有的Laravel Facades 都在 Illuminate\Support\Facades 命名空间中定义.所以可以轻松使用Facade;

***
##### 何时使用Facades

Facades使用Laravel的功能提供了简单,易记的语法,而无需记住必须手动注入或配置的长长的类名.此外,由于对PHP动态方法的独特用法,使得测试起来非常容易

Facades 主要风险是会引起类作用范围的膨胀.

##### Facades 工作原理

***
在Laravel中,Facades是一个可以从容器访问对象的类.其核心的部件就是Facade类.不管是Laravel自带的Facades还是用户自定义的Facades,都继承与Illuminate\Support\Facades\Facade类 
Facade 基类使用了 __callStatic() 魔术方法将你的Facades的调用延迟,直到对象从容器中解析常量


#### Laravel 的契约(Contracts)

***

##### 简介

Laravel的契约是一组定义框架提供的核心服务接口.框架对每个契约都提供了相应的实现.


##### 何时使用契约
***



##### 低耦合

***
```php
namespace App\Orders;

class Repository{

    protected $cache;
    
    public function __construct(\SomePackage\Cache\Memcached $cache){
        $this->cache = $cache;
    
    }
    
    public function find($id)
    {
        if($this->cache->has($id)
        {
        
        }
    }
}

```
在这个类中,程序跟给定的缓存实现高耦合.因为我们依赖于一个扩展包的特定缓存类.一旦这个扩展包的API被更改,我们的代码就必须跟着改变
同样,如果想要将底层的缓存技术替换为另一种缓存技术,就需要再次修改Repository类,而Respository 类不应该了解过多关于谁提供了这些数据或是如何提供等等

```php
    namespace App\Order;
    
    use Illuminate\Contracts\Cache\Respository as Cache;
    
    class Respository{
        protected $cache;
        
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

```



##### 简单性
***
当所有Laravel的服务都使用简洁的接口定义,那就很容易判断给定服务提供的功能,可以将契约视为说明框架功能的简洁文档

##### 如何使用契约

*** 
如何实现一个契约
Laravel中很多类型的类都是通过服务容器解析出来的,包括控制器,事件监听器,中间件,任务队列,甚至路由闭包



#### 路由

##### 基本路由

构建最基本的路由只需要一个URI与一个闭包
```php

    Route::get('/',function(){
        return 'Hello World';
    });
```

##### 默认路由文件
***
所有的Laravel路由都在routes目录中的路由文件中定义,这些文件都由框架自动加载.routes/web.php 文件用于定义web界面的路由.这里面的路由都会被分配到web中间件组.它提供了会话状态和csrf保护等功能.定义在routes/api.php 中的路由都是无状态的,被分配了api中间组

  
    
  
##### 可用的路由方法

路由器容许注册响应任何http请求的路由

```php
    Route::get('',$callback);
    Route::post('',$callabck);
    Route::put('',$callback);
    Route::patch('',$callback);
    Route::options('',$callback);
   

```
有的时候可能需要注册一个响应对个Http请求的路由,这时可用使用match 方法,也可以使用any方法注册一个实现所有http请求的路由

```php
    Route::match(['get','post'],$callback);
    
    Route::any('',$callback);
```

##### CSRF保护

***
指向web路由文件中定义的post,put,delete 路由的任何html表单都应该包含一个csrf令牌字段

```php
    <form action="/profile" method='post'>
    {{ csrf_token()}}
    </form>

```

##### 重定向路由

***
如果要定义重定向到另一个uri的路由,可以使用Route::redirect 方法,这个方法可以快速地实现重定向,而不是需要去定义完整的路由或者控制器
```php
    Route::redirect('/','redirect',301);
    
```

##### 视图路由

*** 
如果路由只需要返回一个视图,可以使用Route::view 方法.它和redirect一样方便,不需要定义完整的路由或控制器.view方法有三个参数,其中前两个是必要参数,分别是url和视图名称,第三个选填,可以传入一个数组,数组的数据被传递到视图
```php
Route::view('/view','view/index',['data'=>[1,2]]);
```

##### 路由参数
***
###### 必填参数
当然,有时需要在路由中捕获url片段,例如获取用户的Id,可以通过定义路由参数来执行此操作

```php
    Route::get('userinfo/{id}',function($id)
    {
        return 'User'.$id;
    });
```


###### 可选参数
有时可能需要指定一个路由参数,希望这个参数是可选的,就可以在参数后面加上? 标记,但前提是要确保路由的相应的变量有默认值

```php
    Route::get('user/{name?}',function($name=null){
        return $name;
    });
```

###### 正则表达式约束

***
可以使用路由实例上的where方法约束路由参数的格式,where方法接受参数和定义参数应如何约束的正则表达式

```php
    Route::get('user/{name}',function($name){
    })->where('name','[A-Za-z]');
```

###### 全局约束

***
如果希望某个具体的路由参数都遵循同一个正则表达式的约束,就使用pattern方法在RouteServiceProvider 的boot 方法中定义这些模式
```php
    public function boot()
    {
        Route::pattern('id','[0-9]+');
    }
```

###### 命名路由
***
命名路由可以方便地为指定路由生成URl或者重定向.通过在路由定义上链式调用name 方法指定路由名称
```php
Route::get('user/profile',function()
{
    return '';
}
)->name('profile');
```
##### 路由组

***
路由组允许在大量路由之间共享路由属性

###### 中间件

***
 









 
    

```