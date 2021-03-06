# Extra 扩展

## Hash 密码

Hash 用于密码加密与验证，此服务必须安装 `kain/think-extra`，需要添加配置 `config/hashing.php`

```php
return [

    // 散列类型
    'driver' => 'argon2id',
    // Bcrypt 配置
    'bcrypt' => [
        'rounds' => env('BCRYPT_ROUNDS', 10),
    ],
    // Argon2i 与Argon2id 配置
    'argon' => [
        'memory' => 1024,
        'threads' => 2,
        'time' => 2,
    ],

];
```

- driver `bcrypt|argon|argon2id` 加密算法
- bcrypt `array` bcrypt 的配置
- argon `array` argon2i 与 argon2id 的配置

安装后服务将自动注册可通过依赖注入使用

```php
use think\extra\contract\HashInterface;

class Index extends BaseController
{
    public function index(HashInterface $hash)
    {
        $hash->create('123456');
    }
}
```

### 加密密码 

- create(string $password, array $options = [])
  - password `string` 密码
  - options `array` 加密参数

```php
use think\support\facade\Hash;

Hash::create('123456789');
```

### 验证密码

- check(string $password, string $hashPassword): bool
  - password `string` 密码
  - hashPassword `string` 散列密码

```php
use think\support\facade\Hash;

$hash = Hash::create('123456789');

// "$argon2id$v=19$m=65536,t=4,p=1$QmlpMEpNY2x3S0FMZ1phVg$XBhTEMcblOge1svlB2/5NNieCDfoT1BvJDinuyBwkKQ"

Hash::check('12345678', $hash);

// false

Hash::check('123456789', $hash);

// true
```

## Cipher 数据加密

Cipher 可以将字符串或数组进行加密解密的服务，此服务必须安装 `kain/think-extra`，需要添加配置 `app_secret` 与 `app_id` 到 `config/app.php`

```php
return [

    'app_id' => env('app.id', null),
    'app_secret' => env('app.secret', null),

];
```

- app_id `string` 应用ID
- app_secret `string` 应用密钥

安装后服务将自动注册可通过依赖注入使用

```php
use think\extra\contract\CipherInterface;

class Index extends BaseController
{
    public function index(CipherInterface $cipher)
    {
        $cipher->encrypt('123');
    }
}
```

### 加密数据内容

- encrypt($context): string
  - context `string|array` 数据
  - return `string` 密文

```php
use think\support\facade\Cipher;

Cipher::encrypt('123');

// FLgXf5EXF6eGEqphO3WVJQ==

Cipher::encrypt([
    'name' => 'kain'
]);

// IyGcnXqDT6ersFhAKdduUQ==
```

### 解密数据

- decrypt(string $ciphertext, bool $auto_conver = true)
  - ciphertext `string` 密文
  - auto_conver `bool` 数据属于数组时是否自动转换
  - return `string|array` 解密内容

```php
use think\support\facade\Cipher;

$result = Cipher::encrypt([
    'name' => 'kain'
]);

Cipher::decrypt($result);

// array:1 [▼
//   "name" => "kain"
// ]
```

## Token 令牌

Token 是 JSON Web Token 方案的功能服务，此服务必须安装 `kain/think-extra`，首先更新配置 `config/token.php`

```php
return [
    'system' => [
        'issuer' => 'system',
        'audience' => 'someone',
        'expires' => 3600,
    ],
];
```

当中 `system` `xsrf` 就是 `Token` 的 Label 标签，可以自行定义名称

- issuer `string` 发行者
- audience `string` 听众
- expires `int` 有效时间

安装后服务将自动注册可通过依赖注入使用

```php
use think\extra\contract\TokenInterface;

class Index extends BaseController
{
    public function index(TokenInterface $token)
    {
        $token->create('system', '12345678', 'a1b2');
    }
}
```

### 生成令牌

- create(string $scene, string $jti, string $ack, array $symbol = []): Plain
  - scene `string` 场景标签
  - jti `string` Token ID
  - ack `string` Token 确认码
  - symbol `array` 标识组
  - return `Lcobucci\JWT\Token\Plain`

```php
use think\support\facade\Token;

$token = Token::create('system', '12345678', 'a1b2');

dump($token->toString());

// "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhcGkua2Fpbm9ubHkuY29tIiwiYXVkIjoiY29uc29sZS5rYWlub25seS5jb20iLCJqdGkiOiIxMjM0NTY3OCIsImFjayI6ImExYjIiLCJzeW1ib2wiOltdLCJleHAiOiIxNjA2MzY3MzQyLjUxMjA2MSJ9.YTIaJU2fBWIssxCu752DAM6yUlWOzJCTJFdsdkT18-0 ◀"
```

### 获取令牌对象

- get(string $jwt): Plain
  - jwt `string` 字符串令牌
  - return `Lcobucci\JWT\Token\Plain`

```php
use think\support\facade\Token;

$jwt = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhcGkua2Fpbm9ubHkuY29tIiwiYXVkIjoiY29uc29sZS5rYWlub25seS5jb20iLCJqdGkiOiIxMjM0NTY3OCIsImFjayI6ImExYjIiLCJzeW1ib2wiOltdLCJleHAiOiIxNjA2MzY3MzQyLjUxMjA2MSJ9.YTIaJU2fBWIssxCu752DAM6yUlWOzJCTJFdsdkT18-0';

$token = Token::get($jwt);

dump($token);

// Lcobucci\JWT\Token\Plain {#78 ▼
//   -headers: Lcobucci\JWT\Token\DataSet {#87 ▼
//     -data: array:2 [▶]
//     -encoded: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9"
//   }
//   -claims: Lcobucci\JWT\Token\DataSet {#88 ▼
//     -data: array:6 [▶]
//     -encoded: "eyJpc3MiOiJhcGkua2Fpbm9ubHkuY29tIiwiYXVkIjoiY29uc29sZS5rYWlub25seS5jb20iLCJqdGkiOiIxMjM0NTY3OCIsImFjayI6ImExYjIiLCJzeW1ib2wiOltdLCJleHAiOiIxNjA2MzY3MzQyLjUxMjA2 ▶"
//   }
//   -signature: Lcobucci\JWT\Token\Signature {#90 ▼
//     -hash: b"a2\x1A%Mƒ\x05b,│\x10«´Øâ\x00╬▓RUÄ╠Éô$WlvD§¾Ý"
//     -encoded: "YTIaJU2fBWIssxCu752DAM6yUlWOzJCTJFdsdkT18-0"
//   }
// }
```

### 验证令牌有效性

- verify(string $scene, string $jwt): stdClass
  - scene `string` 场景标签
  - jwt `string` 字符串令牌
  - return `stdClass`
    - expired `bool` 是否过期
    - token `Lcobucci\JWT\Token\Plain` 令牌对象

```php
use think\support\facade\Token;

$jwt = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhcGkua2Fpbm9ubHkuY29tIiwiYXVkIjoiY29uc29sZS5rYWlub25seS5jb20iLCJqdGkiOiIxMjM0NTY3OCIsImFjayI6ImExYjIiLCJzeW1ib2wiOltdLCJleHAiOiIxNjA2MzY3MzQyLjUxMjA2MSJ9.YTIaJU2fBWIssxCu752DAM6yUlWOzJCTJFdsdkT18-0';
$result = Token::verify('system', $jwt);

dump($result);
//{#94 ▼
//  +"expired": false
//  +"token": Lcobucci\JWT\Token\Plain {#89 ▼
//    -headers: Lcobucci\JWT\Token\DataSet {#90 ▼
//      -data: array:2 [▶]
//      -encoded: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9"
//    }
//    -claims: Lcobucci\JWT\Token\DataSet {#91 ▼
//      -data: array:6 [▶]
//      -encoded: "eyJpc3MiOiJhcGkua2Fpbm9ubHkuY29tIiwiYXVkIjoiY29uc29sZS5rYWlub25seS5jb20iLCJqdGkiOiIxMjM0NTY3OCIsImFjayI6ImExYjIiLCJzeW1ib2wiOltdLCJleHAiOiIxNjA2MzY3MzQyLjUxMjA2 ▶"
//    }
//    -signature: Lcobucci\JWT\Token\Signature {#93 ▼
//      -hash: b"a2\x1A%Mƒ\x05b,│\x10«´Øâ\x00╬▓RUÄ╠Éô$WlvD§¾Ý"
//      -encoded: "YTIaJU2fBWIssxCu752DAM6yUlWOzJCTJFdsdkT18-0"
//    }
//  }
//}
```

## Utils 工具集

Utils 常用工具集合，此服务必须安装 `kain/think-extra`， 安装后服务将自动注册可通过依赖注入使用

```php
use think\extra\contract\UtilsInterface;

class Index extends BaseController
{
    public function index(UtilsInterface $utils)
    {
        return $utils
            ->jump('提交成功', 'index/index')
            ->success();
    }
}
```

### 跳转回调工具

- jump(string $msg, string $url = '', string $type = 'html'): Jump
  - msg `string` 跳转信息
  - url `string` 回调Url
  - type `int` 返回类型 `html` 或`json`

```php
use think\support\facade\Utils;

class Index extends BaseController
{
    public function index()
    {
        return Utils::jump('提交成功', 'index/index')
            ->success();
    }
}
```

## Helper 助手

Helper 助手函数扩展，此服务必须安装 `kain/think-extra`

### 生成 uuid v4

- uuid()
  - return `UuidInterface`

```php
$uuid = uuid();

dump($uuid);

// Uuid {#50 ▼
//   #codec: StringCodec {#53 ▼
//     -builder: DefaultUuidBuilder {#52 ▼
//       -converter: DegradedNumberConverter {#51}
//     }
//   }
//   #fields: array:6 [▼
//     "time_low" => "a2bcf1d5"
//     "time_mid" => "2be3"
//     "time_hi_and_version" => "4dc6"
//     "clock_seq_hi_and_reserved" => "8c"
//     "clock_seq_low" => "d4"
//     "node" => "937835a18a8b"
//   ]
//   #converter: DegradedNumberConverter {#51}
// }

dump($uuid->toString());

// "a2bcf1d5-2be3-4dc6-8cd4-937835a18a8b"
```

### Stringy字符串操作工具

- stringy($str = '', $encoding = null)
  - str `string`
  - encoding `string`
  - return `Stringy\Stringy`，更多操作可参考 [danielstjules/Stringy](https://github.com/danielstjules/Stringy)

```php
stringy('abc');
```
