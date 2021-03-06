# Redis 简单使用

## 简单操作
 比如显示给定用户的配置文件。
```php
  $user = Redis::get('user:profile:'.$id);
  
  Redis::set('user:profile:'.$id, "ok");
 ```
 你可以在 `Redis` 上调用任何的 `Redis` 命令。`Swoft` 使用魔术方法将传递命令给 `Redis`服务器，因此只需传递 Redis 命令所需的参数即可：
 
 ```php
  Redis::set('name', 'Taylor');
  
  $values = Redis::lrange('names', 5, 10);
 ```
所有方法和 操作 `phpredis` 原生一样 本质是通过魔术方法代理访问的，让操作变得更简单

当然普通的字符串设置不能满足我们的日常开发，你也可以使用 数组的方式
```php
  $setData = [
        'goods' => [
            'goods_id'   => 1,
            'goods_name' => 'iPhone xx'
         ]
  ];
  Redis::set($key, $setData);
```
通过 **get** 方法 调用，底层会自动反序列化，保证了存进去，取出来的数据**一致性**。

## 通过 @Inject注入连接池方式使用`Redis`

通过`Inject`注入属性使用和`1.0`方式一样
```php
<?php declare(strict_types=1);


namespace App\Http\Controller;

use Swoft\Redis\Pool;
use Swoft\Bean\Annotation\Mapping\Inject;
use Swoft\Http\Server\Annotation\Mapping\Controller;
use Swoft\Http\Server\Annotation\Mapping\RequestMapping;

/**
 *
 * @Controller("Redis")
 */
class RedisController
{
    /**
     * eg 1
     *
     * @Inject("redis.inc.pool")
     *
     * @var Pool
     */
    private $redisInc;
    
    /**
     * eg 2
     *
     * @Inject()
     *
     * @var Pool
     */
    private $redis;
    
    /**
     * @RequestMapping(route="find")
     *
     * @return array
     *
     * @throws Throwable
     */
    public function find(): array
    {
        $this->redis->set('user', ["name"=>"gimi", "age"=>"18"]);

        return $this->redis->get('user');
    }
}  
```

你可以在中 `Inject` 指定使用的那一个`redis`连接池，如果在`Inject`中没有指定连接池，将默认会用 `@var` 指定的类型注入。

## 指定连接池获取连接
当然 所有的连接都是在连接池中分配的 如果你想使用自己定义的连接可以使用 

```php
$poolName  = 'redis-clusters-pool'
$redis     =  Redis::connection($poolName);
$redis->get("a");
```
 获取的连接了之后和 操作 和`phpredis` 原生使用方式扩展一致 如何创建连接池 参考 `redis 设置`章节
 ，默认是在 `redis.pool`连接池中获取的
 
 ## 缓存命中率
 如果你想看一个 `key` 获取的命中情况，可以在你的`notice`日志级别中获取，格式大致是 `
 redis.hit/req....` 这样的前缀，有助于你分析缓存命中率。
 
 ## 事件监听
 
 ### before 事件
 在`Redis` 执行之前底层会抛出 `RedisEvent::BEFORE_COMMAND` 事件
```php

<?php declare(strict_types=1);


namespace App\Listener;

use Swoft\Event\Annotation\Mapping\Listener;
use Swoft\Event\EventHandlerInterface;
use Swoft\Event\EventInterface;
use Swoft\Redis\RedisEvent;

/**
 * Class FinishListener
 *
 * @since 2.0
 *
 * @Listener(event=RedisEvent::BEFORE_COMMAND)
 */
class RedisBeforeListener implements EventHandlerInterface
{
    /**
     * @param EventInterface $event
     */
    public function handle(EventInterface $event): void
    {
        [$method, $parameters] = $event->getParams();
    }
}

```
 ### after 事件
 
在 `Redis` 命令之后会抛出 `RedisEvent::AFTER_COMMAND` 事件 你可以监听命令的执行情况，如果在执行命令中出现异常 不会抛出 `RedisEvent::AFTER_COMMAND` 事件
```php
<?php declare(strict_types=1);


namespace App\Listener;

use Swoft\Event\Annotation\Mapping\Listener;
use Swoft\Event\EventHandlerInterface;
use Swoft\Event\EventInterface;
use Swoft\Redis\RedisEvent;

/**
 * Class FinishListener
 *
 * @since 2.0
 *
 * @Listener(event=RedisEvent::AFTER_COMMAND)
 */
class RedisAfterListener implements EventHandlerInterface
{
    /**
     * @param EventInterface $event
     */
    public function handle(EventInterface $event): void
    {
        [$method, $parameters, $result] = $event->getParams();
    }
}

```