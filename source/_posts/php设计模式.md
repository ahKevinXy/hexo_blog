---
title: php设计模式
date: 2020-01-05 06:03:43
tags:
---
# PHP 设计模式

## 面向对象基础

#### 类与对象

    把一个模块看做是相关函数的一个集合.被称为类, 类由属性和方法 属性是不同类型的数据对象 方法是处理这些数据的函数
    
    
   
#### 单一职责原则 
    
       可以把类看作是有共同特征的对象的集合,特征的共同性并不是指这些对象都是相同的,而是指它们都有共同的问题
       

#### PHP 中的构造函数

    构造函数在类实例化化后自动启动
    
    
    
### OOP 概念


#### 抽象

       抽象指示了一个对象的基本特征,使它与所有其它对象区分开,从而查看者的角度提供了清晰定义的概念边界;抽象是用来处理复杂性的主要工具,一个问题越复杂,就越需要抽象来解决
       
       
 #### 抽象类 (abstract class)
    抽象类可以为项目提供一种组织机制,抽象类不能实例化,只能由具体类继承抽象类的接口及其它的所有属性值
       
       
 #### 接口(interface)
    
        接口可以包含具体常量,不能有方法或者变量 一般约定接口总以I或i开头
        
        
        
#### 封装 

    封装就是划分一个抽象的诸多元素的过程,这些元素构成该抽象的机构和行为;封装的作用就是将抽象的契约接口与其实现分离
    
    
    
#### 继承(extends)

    一个类如果扩展了=另一个类就会拥有这个类的所有属性和方法
    
    
 #### 多态(ploymorphism)
 
    多态就是指多种形态,一个名字多种实现
    






















## 理解设计模式

### 什么是设计模式

 
* 设计的问题由三个部分组成
1. what 要考虑的相关业务和功能需求
2. how 如何使用特定的设计模式
3. work 实际的实现

#### UML图
    
    统一建模语言(Unified Modeling Language) UML 是对操作.对象和用例进行图形编程的标准方式
    
    
#### 设计模式的特性

1. 设计模式并非即插即用
2. 设计模式是可维护
3. 设计模式是重构的必经之路



### 适配器模式
***
> 适配器设计模式只是将某个对象的接口适配为另一个对象所期望的接口

* 代码示例
```php
    class errorObject {
        private $_error;
        public function __construct($error)
        {
            $this->_error = $error;
        }
        
    }
    
    class logToConsole {
        private $_errorObject;
        public function __construct($errorObject)
        {
            $this->_errorObject = $errorObject;
        }
        public function write()
        {
            fwrite(STDERR,$this->_errorObject->getError();
        }
    }
```

    需要提供日志记录格式  
    
 ```php

    class logTOCsv {
        const CSV_LOCATION = 'log.csv';
        
        private $_errorObject;
        
        public function __construct($errorObject)
        {
            $this->_errorObject = $errorObject;
        }
        public function write()
        {
            $line =  $this->_errorObject->getErrorNumber();
            $line.= ',';
            $line.= $this->_errorObject->getErrorText();
            $line.= "\n";
                                               file_put_contents(self::CSV_LOCATION,$line,FILE_APPEMD);
        }
        
    }

```

>针对这个问题,我们可以采用下面两种解决方案
1. 更改现有代码库的errorObject
2. 创建一个Adapter

```php
    class logToCSVAdapter extends errorObject {
        private $_errorNumber,$_errorText;
        
        public function __construct($error)
        {
            parent::__construct($error);
            $parts  = explode(":",$this->getError());
            $this->_errorNumber = $parts[0];
            $this->_errorText = $parts[1];
            
        }
        public function getErrorNumber()
        {
            return $this->_errorNumber;
        }
        public function getErrorText()
        {
            return $this->getErrorText;
        }
    
    }

```






### 建造者模式

***

> 建造者模式定义了处理其他对象的复杂构建的对象设计


* 代码示例

```php

    class product {
    
        protected $_type = '';
        
        protected $_size = '';
        
        protected $_color = '';
        
        public function __construct($type)
        {
            $this->_type = $type;
        }
        public function setSize($size)
        {
            $this->_size = $size;
        }
        
        public function setColor($color)
        {
            $this->_color = $color;
        }
    }

```

    使用 productBuild 类设计为接受构建的product 对象所需的这些配置选项
    
    
    
    
```php
    class productBuilder {
    
        protected $_product = null;
        
        protected $_configs = array();
        
        public function __construct($configs)
        {
            $this->_product = new product();
            
            $this->_configs = $configs;
        }
        
        public function build()
        
        {
            $this->_product->setSize($this->_configs['size']);
            $this->_product->setType($this->_configs['type']);
            
            $this->_product->setColor($this->_configs['color']);
        }
    
        public function getProduct()
        {
            return $this->_product;
        }
    
    }



```


### 数据访问对象模式

***

> 数据访问对象模式模式了如何创建提供透明访问任何数据源的对象


    DAO(Data Access Object)
    
    
    
### 装饰器模式
*** 
> 如果已有对象的部分内容或功能性发生改变,但是不需要修改原始对象结构,那么使用装饰器模式最合适

* 代码示例

```php
    
    class CD {
    
        public $strackList;
        
        public function __construct()
        {
            $this->strackList = array();
        }
        public function addTrack($strack)
        {
            $this->strackListt[] = $strack;
        }
        public function getTrackList()
        {
            $output = '';
            foreach($this->trackList as $key=>$value){
                $output.= ($key+1).") {$value}.";
            }
        }
    

```


### 委托模式

*** 

>通过分配或委托至其他对象,委托设计模式能够除去核心对象中的判决和复杂的功能性



