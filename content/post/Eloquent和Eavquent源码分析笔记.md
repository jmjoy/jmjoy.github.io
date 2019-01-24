+++
showonlyimage = false
date = "2017-01-27T15:39:41+08:00"
title = "Eloquent和Eavquent源码分析笔记"
image = ""
draft = false
categories = ["其它"]
+++

Eloquent/Model

ActiveRecord

魔术方法 `__get`和`__set`

``` php
    public function __get($key)
    {
        return $this->getAttribute($key);
    }

    public function __set($key, $value)
    {
        $this->setAttribute($key, $value);
    }
    
```

魔术方法 `__call`和`__callStatic`，查询器

``` php
    public function __call($method, $parameters)
    {
        if (in_array($method, ['increment', 'decrement'])) {
            return $this->$method(...$parameters);
        }

        return $this->newQuery()->$method(...$parameters);
    }

    public static function __callStatic($method, $parameters)
    {
        return (new static)->$method(...$parameters);
    }
```

Trait Evaquent

复写了魔术方法`__call`

``` php
    public function __call($method, $parameters)
    {
        // 每次调用查询都会先boot
        $this->bootEavquentIfNotBooted();

        if ($this->isAttributeRelation($method)) {
            return call_user_func_array($this->attributeRelations[$method], $parameters);
        }

        return parent::__call($method, $parameters);
    }
```
