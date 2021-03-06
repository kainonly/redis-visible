# Func 功能

## Auth 登录鉴权

Auth 创建登录后将 Token 字符串存储在Cookie 中，通过主控制去引用该特性

```php
use app\system\controller\BaseController;
use think\support\traits\Auth;

class Main extends BaseController
{
    use Auth;
}
```

### 设置令牌自动刷新的总时效 

- refreshTokenExpires(): int
  - return `int` 默认 `604800`，单位< 秒 >

```php
use app\system\controller\BaseController;
use think\support\traits\Auth;

class Main extends BaseController
{
    use Auth;

    protected function refreshTokenExpires()
    {
        return 7200;
    }
}
```

### 创建登录鉴权 

- create(string $scene, array $symbol = []): array
  - scene `string` 场景标签
  - symbol `array` 标识
  - return `array`

在登录验证成功后调用

```php
use app\system\controller\BaseController;
use think\support\traits\Auth;

class Main extends BaseController
{
    use Auth;

    public function login()
    {
        // $raws = ...
        // ...
        // 登录验证成功

        return $this->create('system', [
            'user' => $raws['username'],
            'role' => explode(',', $raws['role'])
        ]);
    }
}
```

### 验证登录

- authVerify(string $scene): array
  - scene `string` 场景标签

```php
use app\system\controller\BaseController;
use think\support\traits\Auth;

class Main extends BaseController
{
    use Auth;

    public function verify()
    {
        return $this->authVerify('system');
    }
}
```

### 验证返回钩子

- authHook(array $symbol): array
  - symbol `array` 标识

```php
use app\system\controller\BaseController;
use think\support\traits\Auth;

class Main extends BaseController
{
    use Auth;

    protected function authHook(array $symbol): array
    {
        $data = AdminRedis::create()->get($symbol['user']);
        if (empty($data)) {
            return [
                'error' => 1,
                'msg' => 'freeze'
            ];
        }
        return [
            'error' => 0,
            'msg' => 'ok'
        ];
    }
}
```

### 销毁登录鉴权

- destory(string $scene): array
  - scene `string` 场景标签

```php
use app\system\controller\BaseController;
use think\support\traits\Auth;

class Main extends BaseController
{
    use Auth;

    public function logout()
    {
        return $this->destory('system');
    }
}
```

## RedisModel 缓存模型

使用 RedisModel 定义缓存模型，目的是将分散的缓存操作统一定义，例如：设定Acl访问控制表的缓存模型

```php
class Acl extends RedisModel
{
    protected $key = 'system:acl';
    private $rows = [];

    /**
     * 清除缓存
     * @return bool
     */
    public function clear()
    {
        return (bool)$this->redis->del([$this->key]);
    }

    /**
     * @param string $key 访问控制键
     * @param int $policy 控制策略
     * @return array
     * @throws \Exception
     */
    public function get(string $key, int $policy)
    {
        if (!$this->redis->exists($this->key)) {
            $this->update($key);
        } else {
            $this->rows = json_decode($this->redis->hget($this->key, $key), true);
        }

        switch ($policy) {
            case 0:
                return explode(',', $this->rows['read']);
            case 1:
                return array_merge(
                    explode(',', $this->rows['read']),
                    explode(',', $this->rows['write'])
                );
            default:
                return [];
        }
    }

    /**
     * 更新缓存
     * @param string $key 访问控制键
     * @throws \Exception
     */
    private function update(string $key)
    {
        $lists = Db::name('acl')
            ->where('status', '=', 1)
            ->field(['key', 'write', 'read'])
            ->select();

        if (empty($lists)) {
            return;
        }

        $this->redis->pipeline(function (Pipeline $pipeline) use ($key, $lists) {
            foreach ($lists as $index => $value) {
                $pipeline->hset($this->key, $value['key'], json_encode([
                    'write' => $value['write'],
                    'read' => $value['read']
                ]));
                if ($key == $value['key']) {
                    $this->rows = [
                        'write' => $value['write'],
                        'read' => $value['read']
                    ];
                }
            }
        });
    }
}
```

当对应的 `acl` 表数据发生变更时，执行 `clear()` 来清除缓存

```php
Acl::create()->clear();
```

通过缓存模型自定义的获取规则获取对应的数据，例如：查访问键 `admin` 对应的数据，如缓存不存在则生成缓存并返回数据

```php
Acl::create()->get('admin', 0);
```

如果同时要执行多个缓存模型，可以注入事务对象

```php
Redis::transaction(function (MultiExec $multiExec) {
    Someone1::create($multiExec)->factory();
    Someone2::create($multiExec)->factory();
    Someone3::create($multiExec)->factory();
});
```

### SMS 短信验证

手机短信验证码缓存类

#### 设置手机验证码缓存

- factory(string $phone, string $code, int $timeout = 120): string
  - phone `string` 手机号
  - code `string` 验证码
  - timeout `int` 超时时间，默认60秒
  - return `bool`

```php
Sms::create()->factory('12345678910', '13125');
```

#### 验证手机验证码

- check(string $phone, string $code, bool $once = false): bool
- phone `string` 手机号
- code `string` 验证码
- once `bool` 验证成功后失效，默认`false`
- return `bool`

```php
$sms = Sms::create();
$checked = $sms->check('12345678910', '11224');
dump($checked);
// false
$checked = $sms->check('12345678910', '13125');
dump($checked);
// true
$checked = $sms->check('12345678910', '13125', true);
dump($checked);
// true
$checked = $sms->check('12345678910', '13125');
dump($checked);
// false
```

#### 获取验证时间 

- time(string $phone): array
  - phone `string` 手机号
  - return `array`

```php
$sms = Sms::create();
$sms->factory('12345678910', '13125', 3600);

$data = $sms->time('12345678910');
dump($data);
// array (size=2)
//   'publish_time' => int 1548644216
//   'timeout' => int 3600
```

- publish_time `int` 指发布时间
- timeout `int` 指有效时间

### Refresh Token 缓存

Refresh Token 是用于自动刷新、验证对应 Token 的缓存模型

#### 生产 Refresh Token

- factory(string $jti, string $ack, int $expires): string
  - jti `string` JSON Web Token ID
  - ack `string` Token ID 验证码
  - expires `int` 存在时间，单位<秒>
  - return `string`

```php
$jti = Ext::uuid()->toString();
$ack = Str::random();

RefreshToken::create()->factory($jti, $ack, 86400*7);
```

#### 验证 Token 的 Token ID 有效性 

- verify(string $jti, string $ack): bool
  - jti `string` JSON Web Token ID
  - ack `string` Token ID 验证码
  - return `bool`

```php
RefreshToken::create()->verify($jti, $ack);
```

#### 清除 Token 对应的 Refresh Token

- clear(string $jti, string $ack): bool
  - jti `string` JSON Web Token ID
  - ack `string` Token ID 验证码
  - return `bool`

```php
RefreshToken::create()->clear($jti, $ack);
```