# Component 组件

## Redis 缓存

Redis 缓存使用 [Predis](https://github.com/nrk/predis) 做为依赖，还需要安装 `kain/think-redis`

```shell
composer require kain/think-redis
```

安装后服务将自动注册，然后需要更新配置 `config/database.php`，例如：

```php
return [

    'redis' => [
        'default' => [
            // 服务器地址
            'host' => Env::get('redis.host', '127.0.0.1'),
            // 密码
            'password' => Env::get('redis.password', null),
            // 端口
            'port' => Env::get('redis.port', 6379),
            // 数据库号
            'database' => Env::get('redis.db', 0),
        ]
    ],
    
];
```

- scheme `string` 连接协议，支持 `tcp` `unix` `http`
- host `string` 目标服务器的IP或主机名
- port `int` 目标服务器的TCP / IP端口
- path `string` 使用 `unix socket` 的文件路径
- database `int` 逻辑数据库
- password `string` 身份验证口令
- async `boolean` 指定是否以非阻塞方式建立与服务器的连接
- persistent `boolean` 指定在脚本结束其生命周期时是否应保持基础连接资源处于打开状态
- timeout `float` 用于连接到Redis服务器的超时
- read_write_timeout `float` 在对基础网络资源执行读取或写入操作时使用的超时
- alias `string` 通过别名来标识连接
- weight `integer` 集群权重
- iterable_multibulk `boolean` 当设置为true时，Predis将从Redis返回multibulk作为迭代器实例而不是简单的PHP数组
- throw_errors `boolean` 设置为true时，Redis生成的服务器错误将转换为PHP异常

### 测试写入一个缓存 

- client(string $name = 'default')
  - name `string` 配置标识
  - return `Predis\Client`

```php
use think\support\facade\Redis;

Redis::client()->set('name', 'abc')
```

使用 `pipeline` 批量执行一万条写入

```php
use think\support\facade\Redis;

Redis::client()->pipeline(function (Pipeline $pipeline) {
    for ($i = 0; $i < 10000; $i++) {
        $pipeline->set('test:' . $i, $i);
    }
});
```

面向缓存使用事务处理

```php
use think\support\facade\Redis;

// success
Redis::client()->transaction(function (MultiExec $multiExec) {
    $multiExec->set('name:a', 'a');
    $multiExec->set('name:b', 'b');
});

// failed
Redis::client()->transaction(function (MultiExec $multiExec) {
    $multiExec->set('name:a', 'a');
    // mock exception
    throw new Exception('error');
    $multiExec->set('name:b', 'b');
});
```

## AMQP 消息队列

AMQP 消息队列操作类使用 [kain/simplify-amqp](https://github.com/kainonly/simplify-amqp) 做为依赖，首先使用 `composer` 安装操作服务

```shell
composer require kain/think-amqp
```

安装后服务将自动注册，然后需要更新配置 `config/queue.php`，例如：

```php
return [

    'rabbitmq' => [
        'default' => [
            // 服务器地址
            'hostname' => Env::get('rabbitmq.host', 'localhost'),
            // 端口号
            'port' => Env::get('rabbitmq.port', 5672),
            // 虚拟域
            'virualhost' => Env::get('rabbitmq.virualhost', '/'),
            // 用户名
            'username' => Env::get('rabbitmq.username', 'guest'),
            // 密码
            'password' => Env::get('rabbitmq.password', 'guest'),
        ]
    ]

];
```

### AMQP 客户端 

- client(string $name = 'default')
  - name `string` 配置标识
  - return `simplify\amqp\AMQPClient`

### 创建默认信道

- channel(Closure $closure, string $name = 'default', array $options = [])
  - closure `Closure` 信道处理
  - name `string` 配置标识
  - options `array` 操作配置
    - transaction `boolean` 开启事务，默认 `false`
    - channel_id `string` 定义信道ID，默认 `null`
    - reply_code `int` 回复码，默认 `0`
    - reply_text `string` 回复文本，默认 `''`
    - method_sig `array` 默认 `[0,0]`

```php
use think\support\facade\AMQP;
use simplify\amqp\AMQPManager;

AMQP::channel(function (AMQPManager $manager) {
    // Declare
    $manager->queue('test')
        ->setDeclare([
            'durable' => true
        ]);

    // Or delete
    $manager->queue('test')
        ->delete();
});
```

### 创建包含事务的信道

- channeltx(Closure $closure, string $name = 'default', array $options = [])

```php
use think\support\facade\AMQP;
use simplify\amqp\AMQPManager;

AMQP::channeltx(function (AMQPManager $manager) {
    $manager->publish(
        AMQPManager::message(
            json_encode([
                "name" => "test"
            ])
        ),
        '',
        'test'
    );
    // 当返回为 false 时，将不提交发布消息
    return true;
});
```

关于 `simplify\amqp\AMQPManager` 对象完整使用可查看 [simplify-amqp](https://github.com/kainonly/simplify-amqp) 的单元测试 `tests` 目录

## ElasticSearch 全文搜索

ElasticSearch 可对数据进行全文搜索或针对数据分析查询，首先使用 `composer` 安装操作服务

```shell
composer require kain/think-elastic
```

安装后服务将自动注册，然后需要更新配置 `config/database.php`

```php
return [

    'elasticsearch' => [
        'default' => [
            // 集群连接
            'hosts' => explode(',', Env::get('elasticsearch.hosts', 'localhost:9200')),
            // 重试次数
            'retries' => 0,
            // 公共CA证书
            'SSLVerification' => null,
            // 开启日志
            'logger' => null,
            // 配置 HTTP Handler
            'handler' => null,
            // 设置连接池
            'connectionPool' => Elasticsearch\ConnectionPool\StaticNoPingConnectionPool::class,
            // 设置选择器
            'selector' => Elasticsearch\ConnectionPool\Selectors\RoundRobinSelector::class,
            // 设置序列化器
            'serializer' => Elasticsearch\Serializers\SmartSerializer::class
        ]
    ]

];
```

- hosts `array` 集群连接
- retries `int` 重试次数
- SSLVerification `string` 公共CA证书
- logger `LoggerInterface` 开启日志
- handler `mixed` 配置 HTTP Handler
- connectionPool `AbstractConnectionPool|string` 设置连接池
- selector `SelectorInterface|string` 设置选择器
- serializer `SerializerInterface|string` 设置序列化器

### 客户端

- client(string $label = 'default')
  - label `string` 配置label
  - return `Elasticsearch\Client`

```php
use think\support\facade\ES;

$response = ES::client()->index([
    'index' => 'test',
    'id' => 'test',
    'body' => [
        'value' => 1
    ]
]);

// ^ array:8 [▼
//   "_index" => "test"
//   "_type" => "_doc"
//   "_id" => "test"
//   "_version" => 1
//   "result" => "created"
//   "_shards" => array:3 [▼
//     "total" => 2
//     "successful" => 1
//     "failed" => 0
//   ]
//   "_seq_no" => 0
//   "_primary_term" => 1
// ]
```

获取文档

```php
use think\support\facade\ES;

$response = ES::client()->get([
    'index' => 'test',
    'id' => 'test'
]);

// ^ array:8 [▼
//   "_index" => "test"
//   "_type" => "_doc"
//   "_id" => "test"
//   "_version" => 1
//   "_seq_no" => 0
//   "_primary_term" => 1
//   "found" => true
//   "_source" => array:1 [▼
//     "value" => 1
//   ]
// ]
```

搜索文档

```php
use think\support\facade\ES;

$response = ES::client()->search([
    'index' => 'test',
    'body' => [
        'query' => [
            'match' => [
                'value' => 1
            ]
        ]
    ]
]);

// ^ array:4 [▼
//   "took" => 4
//   "timed_out" => false
//   "_shards" => array:4 [▼
//     "total" => 1
//     "successful" => 1
//     "skipped" => 0
//     "failed" => 0
//   ]
//   "hits" => array:3 [▼
//     "total" => array:2 [▼
//       "value" => 1
//       "relation" => "eq"
//     ]
//     "max_score" => 1.0
//     "hits" => array:1 [▼
//       0 => array:5 [▼
//         "_index" => "test"
//         "_type" => "_doc"
//         "_id" => "test"
//         "_score" => 1.0
//         "_source" => array:1 [▼
//           "value" => 1
//         ]
//       ]
//     ]
//   ]
// ]
```

删除文档

```php
use think\support\facade\ES;

$response = ES::client()->delete([
    'index' => 'test',
    'id' => 'test'
]);

// ^ array:8 [▼
//   "_index" => "test"
//   "_type" => "_doc"
//   "_id" => "test"
//   "_version" => 2
//   "result" => "deleted"
//   "_shards" => array:3 [▼
//     "total" => 2
//     "successful" => 1
//     "failed" => 0
//   ]
//   "_seq_no" => 1
//   "_primary_term" => 1
// ]
```

删除索引

```php
use think\support\facade\ES;

$response = ES::client()->indices()->delete([
    'index' => 'test',
]);

// ^ array:1 [▼
//   "acknowledged" => true
// ]
```

创建索引

```php
use think\support\facade\ES;

$response = ES::client()->indices()->create([
    'index' => 'test'
]);

// ^ array:3 [▼
//   "acknowledged" => true
//   "shards_acknowledged" => true
//   "index" => "test"
// ]
```

`think-elastic` 使用了 [elasticsearch/elasticsearch](https://packagist.org/packages/elasticsearch/elasticsearch) ，更多方法可查看 [Elasticsearch-PHP](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/index.html) 完整文档

## 阿里云相关扩展

阿里云相关扩展是针对阿里云库的统一简化，首先使用 `composer` 安装操作服务

```shell
composer require kain/think-aliyun-extra
```

安装后服务将自动注册，然后需要更新配置 `config/aliyun.php`，例如：

```php
return [

    'accessKeyId' => env('aliyun.id'),
    'accessKeySecret' => env('aliyun.secret'),
    'oss' => [
        'endpoint' => env('aliyun.oss_endpoint'),
        'extranet' => env('aliyun.oss_extranet'),
        'bucket' => env('aliyun.oss_bucket')
    ]

];
```

- accessKeyId `string` 阿里云 keyid
- accessKeySecret `string` 阿里云 key secret
- oss
  - endpoint `string` 对象存储endpoint
  - extranet `string` 对象存储外网地址
  - bucket `string` 桶名

### 获取对象存储客户端

- Oss::getClient(bool $extranet = false): OssClient

### 上传至阿里云对象存储

- Oss::put(string $name): string
  - name `string` File 请求文件
  - return `string` 对象名称

```php
use think\support\facade\Oss;

public function uploads()
{
    try {
        $saveName = Oss::put('image');
        return [
            'error' => 0,
            'data' => [
                'savename' => $saveName,
            ]
        ];
    } catch (\Exception $e) {
        return [
            'error' => 1,
            'msg' => $e->getMessage()
        ];
    }
}
```

## 华为云相关扩展

华为云相关扩展是针对华为云库的统一简化，首先使用 `composer` 安装操作服务

```shell
composer require kain/think-huaweicloud-extra
```

安装后服务将自动注册，然后需要更新配置 `config/huaweicloud.php`，例如：

```php
return [

    'accessKeyId' => env('huaweicloud.id'),
    'accessKeySecret' => env('huaweicloud.secret'),
    'obs' => [
        'endpoint' => env('huaweicloud.obs_endpoint'),
        'bucket' => env('huaweicloud.obs_bucket')
    ]

];
```

- accessKeyId `string` 华为云 keyid
- accessKeySecret `string` 华为云 key secret
- obs
  - endpoint `string` 对象存储endpoint
  - bucket `string` 桶名

### 获取对象存储客户端

- Obs::getClient(): ObsClient

### 上传至华为云对象存储

- Obs::put(string $name): string
  - name `string` File 请求文件
  - return `string` 对象名称

```php
use think\support\facade\Obs;

public function uploads()
{
    try {
        $saveName = Obs::put('image');
        return [
            'error' => 0,
            'data' => [
                'savename' => $saveName,
            ]
        ];
    } catch (\Exception $e) {
        return [
            'error' => 1,
            'msg' => $e->getMessage()
        ];
    }
}
```
