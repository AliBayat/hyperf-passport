# richard/hyperf-passport

hyperf �� hyperf-passport �����֧�ֶԶ����û����е�¼��Ȩ֧��Oauth2.0��������Ȩģʽ��Ŀǰ������Ȩģʽ����ȫ���á�
������ο��� laravel �� passport �����ƣ�ʹ���������� laravel �� passport ��ࡣ

> �κ��������QQ���ʣ�444626008

## ��װǰ��׼�� - before install

PHP>=7.3
��װ������
```bash
#��Ȩ������
$ composer require 96qbhy/hyperf-auth
$ php bin/hyperf.php vendor:publish 96qbhy/hyperf-auth
#����������
$ composer require hyperf-ext/encryption
$ php bin/hyperf.php vendor:publish hyperf-ext/encryption
$ composer require hyperf-ext/hashing
$ php bin/hyperf.php vendor:publish hyperf-ext/hashing
#ģ���������ͼ
$ composer require hyperf/view-engine
$ composer require hyperf/view
#hyperf��session
$ composer require hyperf/session
$ php bin/hyperf.php vendor:publish hyperf/session
```
ʹ�� php bin/hyperf.php gen:key ������������Կ,����KEYֵ���Ƶ��ļ� config/autoload/encryption.php�е�env('AES_KEY', 'place_to_hold_key')

�༭�ļ�config/autoload/view.php������ͼĬ������:
```
<?php
declare(strict_types=1);

use Hyperf\View\Mode;
use Hyperf\View\Engine\BladeEngine;

return [
    // ʹ�õ���Ⱦ����
    'engine' => BladeEngine::class,
    // ����д��Ĭ��Ϊ Task ģʽ���Ƽ�ʹ�� Task ģʽ
    'mode' => Mode::TASK,
    'config' => [
        // �������ļ��в����������д���
        'view_path' => BASE_PATH . '/storage/view/',
        'cache_path' => BASE_PATH . '/runtime/view/',
    ],
];
?>
```
���ļ� config/autoload/middlewares.php�����ȫ���м�� 
```
<?php
return [
    // ����� http ��ӦĬ�ϵ� server name��������Ҫ������ server ��ʹ�� Session����Ҫ��Ӧ������ȫ���м��
    'http' => [
        \Hyperf\Session\Middleware\SessionMiddleware::class,
    ],
];
?>
```


## ��װ - install


```bash
$ composer require richard/hyperf-passport
php bin/hyperf.php vendor:publish richard/hyperf-passport
```

## ���� - configuration


�༭�ļ� config/autoload/auth.php

���ļ�������Provicer��Guard

use Richard\HyperfPassport\PassportUserProvider;

use Richard\HyperfPassport\Guard\TokenGuard;

��guards������д

'passport' => [
    'driver' => TokenGuard::class,
    'provider' => 'users',
]

����users���涨����Ӧ��provider

'users' => [
    'driver' => PassportUserProvider::class, 
	
    'model' => App\Model\User::class,
]

����Ϊauth.php�ļ�����

```
<?php

declare(strict_types=1);

use HPlus\Admin\Model\Admin\Administrator;
use Qbhy\HyperfAuth\Provider\EloquentProvider;
use Qbhy\SimpleJwt\Encoders\Base64UrlSafeEncoder;
use Qbhy\SimpleJwt\EncryptAdapters\PasswordHashEncrypter;
use Richard\HyperfPassport\PassportUserProvider;
use Richard\HyperfPassport\Guard\TokenGuard;

return [
    'default' => [
        'guard' => 'jwt',
        'provider' => 'admin',
    ],
    'guards' => [// �����߿�������������Լ��� guard ��guard Qbhy\HyperfAuth\AuthGuard �ӿ�
        'jwt' => [
            'driver' => Qbhy\HyperfAuth\Guard\JwtGuard::class,
            'provider' => 'admin',
            'secret' => env('JWT_SECRET', 'hyperf.plus'),
            'ttl' => 60 * 60, // ��λ��
            'default' => PasswordHashEncrypter::class,
            'encoder' => new Base64UrlSafeEncoder(),
            'cache' => function () {
                return make(Qbhy\HyperfAuth\HyperfRedisCache::class);
            },
        ],
        'session' => [
            'driver' => Qbhy\HyperfAuth\Guard\SessionGuard::class,
            'provider' => 'users',
        ],
        'passport' => [
            'driver' => TokenGuard::class,
            'provider' => 'users',
        ],
    ],
    'providers' => [
        'admin' => [
            'driver' => EloquentProvider::class, // user provider ��Ҫʵ�� Qbhy\HyperfAuth\UserProvider �ӿ�
            'model' => Administrator::class, //  ��Ҫʵ�� Qbhy\HyperfAuth\Authenticatable �ӿ�
        ],
        'users' => [
            'driver' => PassportUserProvider::class, // user provider ��Ҫʵ�� Qbhy\HyperfAuth\UserProvider �ӿ�
            'model' => App\Model\User::class, //  ��Ҫʵ�� Qbhy\HyperfAuth\Authenticatable �ӿ�
        ],
        'merchants' => [
            'driver' => PassportUserProvider::class, // user provider ��Ҫʵ�� Qbhy\HyperfAuth\UserProvider �ӿ�
            'model' => App\Model\Merchant::class, //  ��Ҫʵ�� Qbhy\HyperfAuth\Authenticatable �ӿ�
        ],
    ],
];
?>
```
ִ��Ǩ��

php bin/hyperf.php migrate

php bin/hyperf.php migrate:status

��װpassport

php bin/hyperf.php passport:install  --force --length=4096

php bin/hyperf.php passport:purge

�㻹���Ը���providers�����������Ԫ������client

php bin/hyperf.php passport:client --password --name="your client name"


�������������ļ�����ִ�� php bin/hyperf.php db:seed --path=seeders/user_table_seeder.php

����seeders/user_table_seeder.phpΪ����ļ�·��

ע������ļ�������ĸ�ʽΪ \HyperfExt\Hashing\Hash::make('your password'),����ᵼ��passport����У��ʧ��


���û�ģ��������\Richard\HyperfPassport\HasApiTokens��\Richard\HyperfPassport\Auth\AuthenticatableTrait�Լ�\Qbhy\HyperfAuth\AuthAbility

�û���¼Ĭ������֤email���ϣ����֤�����ֶο�����ģ�������findForPassport������Ȼ���д�Լ��Ĵ����߼�

�û�����Ĭ�ϴ洢�ֶ���password���ϣ����֤�����ֶο�����ģ�������getAuthPassword������Ȼ�󷵻��Լ��������ֶ�

����Ϊģ���ļ�User.php����

```
<?php

declare (strict_types=1);

namespace App\Model;

use Hyperf\DbConnection\Db;
use Qbhy\HyperfAuth\AuthAbility;
use Qbhy\HyperfAuth\Authenticatable;
use Richard\HyperfPassport\Auth\AuthenticatableTrait;
use Richard\HyperfPassport\HasApiTokens;

/**
 */
class User extends Model implements Authenticatable {

    use HasApiTokens;
    use AuthenticatableTrait;
    use AuthAbility;

    public $timestamps = false;

    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'member';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [];

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [];


    /**
     * �޸���֤ʱ��Ĭ��username�ֶ�
     */
    public function findForPassport($username) {
        if (strpos($username, '@') !== false) {
            return $this->findByEmailForPassport($username);
        } else {
            if ((is_numeric($username)) && (strlen($username) == 11)) {
                return $this->findByMobileForPassport($username);
            } else {
                return $this->findByUsernameForPassport($username);
            }
        }
    }

    protected function findByEmailForPassport($username) {
        return $this->where('email', $username)->first();
    }

    protected function findByMobileForPassport($username) {
        return $this->where('mobile', $username)->first();
    }

    protected function findByUsernameForPassport($username) {
        return $this->where('member_name', $username)->first();
    }

    /**
     * Get the password for the user.
     *
     * @return string
     */
    public function getAuthPassword() {
        return $this->member_pass;
    }

}
?>
```


## ʹ�� - usage


> ������α���룬�����ο���
���ļ�config/autoload/exceptions.php�����ȫ���쳣������
```
<?php

declare(strict_types=1);
/**
 * This file is part of Hyperf.
 *
 * @link     https://www.hyperf.io
 * @document https://hyperf.wiki
 * @contact  group@hyperf.io
 * @license  https://github.com/hyperf/hyperf/blob/master/LICENSE
 */
return [
    'handler' => [
        'http' => [
            \HPlus\Admin\Exception\Handler\AppExceptionHandler::class,
            Hyperf\HttpServer\Exception\Handler\HttpExceptionHandler::class,
            App\Exception\Handler\AppExceptionHandler::class,
            \Richard\HyperfPassport\PassportExceptionHandler::class,
        ],
    ],
];
?>
```
##### �ӿ�����

- ��¼�Ի�ȡ�Ự���ƺ�ˢ������

##### �����ַ
- ` /oauth/token `
  
##### ����ʽ
- POST 

##### ����

|������|��ѡ|����|˵��|
|:----    |:---|:----- |-----   |
username |��  |string |�û���/����/�ֻ���  |
password |��  |string |�û�����  |
grant_type |��  |string |��Ȩ���� һ����password  |
client_id |��  |string |����˷����client_id  |
client_secret |��  |string |����˷����client_id��Ӧ����Կ  |

##### ��Ӧ��Ϣ 

``` 
{
    "token_type": "Bearer",
    "expires_in": 31536000,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiIyIiwianRpIjoiMjFhZWY3ODJkNjhhMjE3YmQzYTg1YTFlMTY3MWYwMjBmMzIxOGIwMTNlZDA4ZmNlMjIzYTFiNGFkODM1ZGY2MDE3YThjODg0YjU3ZDVhMDUiLCJpYXQiOjE2Mjg1NjkzMzQsIm5iZiI6MTYyODU2OTMzNCwiZXhwIjoxNjYwMTA1MzM0LCJzdWIiOiIxIiwic2NvcGVzIjpbXX0.Z2PCWbJeL4NsDtJqLyER-Gg0Kf-DV-66mV91bVLryXJtl3YBhZcPsALQZt5WVTa4gC2s-GBOvMBnREZeJ4RfBaBKoBmEF1_6Cj1I8u0zdazF3mGh-V5JpiDJw73XmdQQ61lqiOp0Lxx34H7ZSkhGYhn9QqSkHmqtrRT_8NYY-bJ1GOE4i0EO-ckFqrHhtvWrVooZ5eN3SkxG3bEf24LLuvCj9EhtKPdF818dYjjWxiA2pl_3OAakQDOHTVh1MhvFW1TTnjBGf0aG2_7gcZWyFzJydx59U_8knOvIS5upTB9aP13q2dXGTGf9Q-1EvVDLNyxB_ppDCkCogc7daTkgs_aqvoC1EsC5pqPDZQDCO5RbVcJ881GGNdjtrmud7qapc9HO7e6JZSKg_cx72IB8jziKwWiqTMJMtDrlaWJ4gkGMO5MeEnberIp6J6Yut0iWR6CUWVBTDPymeOpdbZqpwINhcFh4qq_YSNh8IE9tW9-HbYi6NrAX1I1KSaDWgHI9m56nkY2afT8le0IbEJ5AjwcWBATuQbJfj3S2jfyIembJoq9egKeGMrG9KADM151phFR1h6vItJMlGbjwMx6Pry6E5fvQUIxfi3N5-k-ptfKEsb2x-ENffzg_W8aeEEbObt4kV7OczxKGGdGqk3WY7_suASyEkN-7oEiUd77EJps",
    "refresh_token": "def50200961f6b024f098f4ef9416c07515a6449f5feb7e3db6e2e19f4bb260f2758e9bc71b81e49587a96f83658c05f8b93243a6cd5342f1a6a7eee8582e3dedab8915ce24f41875077cd5e22926a53ea0b4675eeb86f3322285848cbac96086eedb0782d6d99a8f9bbe39bcdf1c3215ae127e0a40a9536bdb3496e36f03026015ebf88c81c1a860c6c15a8a48edc7bc8c4a150948b1cfad76c29b01e403711a25f6a6969aaec0777ef6919a7fc707ea63c780e744ceb593f8d7cfd8aef7af59769f1ba5be7b6479c45cdd1c15d3827dd6ba4d0193bead299840c4fea66356a56e2ca407add2d904b1a97a4f0977ad4fcc256cc8f805d3e1fe0379e77478c32d2c22b3f3b31ac289645873cae6de46fa50523238826942846746b0ee4270e6dffcd79994b14a939ada51af7afcc86047f5350b178f0a1d18ba4a3c72b5327dd366a4224252571e1a238fd11748703dbd439f620809b6706fd0d485c29b5c04feb"
}
```
##### �ӿ�����

- ʹ��ˢ�����ƻ���µĻỰ���ƺ�ˢ������

##### �����ַ
- ` /oauth/token/refresh `
  
##### ����ʽ
- POST 

##### ����

|������|��ѡ|����|˵��|
|:----    |:---|:----- |-----   |
grant_type |��  |string |��Ȩ���� refresh_token  |
refresh_token |��  |string |��¼��õ�ˢ������  |
client_id |��  |string |����˷����client_id  |
client_secret |��  |string |����˷����client_id��Ӧ����Կ  |

##### ��Ӧ��Ϣ 

``` 
{
    "token_type": "Bearer",
    "expires_in": 31536000,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiIyIiwianRpIjoiZTAwYzY5ZmU1YzJmZjJhY2RlYjU5ZGU5YTY3YzIwOTNiOWNmZGJmNjFjYWIwMDg5OWNmYTI1NWJiZWVhYmNlYTJhN2JiNGFhNTUyMzg3NzEiLCJpYXQiOjE2Mjg1Njk0ODgsIm5iZiI6MTYyODU2OTQ4OCwiZXhwIjoxNjYwMTA1NDg4LCJzdWIiOiIxIiwic2NvcGVzIjpbXX0.qljYkIJUOkrMFsdxrMrieuj9uanUMo5lKqwvWBvg1cvHFvjXA0FxhTb6cnnKNKdUCFmKCIwWhCY-MleNDy5rso5NF_1EWiWTmaWgpGibVZvbuvrPSL6md8OWiMp3WBa-twO0F-YGkinRu2zycpi_3eVFp5OLL1vWTsOPNuUAHIqWc0bOjoKdGLy8z3bGY3K6iOzBDk2E9FNZG-af8ZDL-cA-0wDcsLRibexBon8rzNbpCK_rmuk8tm2u4xiEQ8xtvJysQ5d8vm19oUpXY61WkiePXbJolxpctCJAkmyZftwFIH1J9nQbLXxyeDenWqkB5Yi2L8wbgmf6y4xhNfDCDvNlionJl32COeT90zj0CUijQyUy6xrUW_kieC2OIjQ6FWLfMA_tMg2RmoS4BAyQah9cFq2B9g0K-SkVyQKpm7Tb7oTqJ3b3mMcL_SYGcwQH5QrAAFI0ngiHMdXJKW_HAcwV5qycQDRkeSqdkNExqawSeFVuM9xqstv8Q-y0i2Y4MmweBo8WHi4UL5NdALGKeK-rT_vQePcb-6l30d3PODH4UpZKwTddfNaP60m3OAqSOP4b1ZeV9shLSOotdUobDHzI3Y1Geujv30uphp1FyvPdsxKUba7o_94GOAsj7ggIAY1K5VLdjpUF0AxL611BRTWZ2NA5whwPbGOg5WGQpOc",
    "refresh_token": "def50200c3c742e60906d59ef5f9628de44af8cf2fbc77b2782c540ed0c9a98a149b2c134d6156ec38f8ea340f09aa096623e0fb7000dfb6169c140d5ed08ebe50f0daa5d186fd05937e35f1bcdd4be81cc01e6bf9d5dcf32dbeeb124e1a729acf089089f7a2ab53d94ebdacb020834b831b6b9bba56e644eb0a320ebe2ce1cbdaf5c825b195396782ab0d8d8967d68e8edc9052d276d72f4f62529182fd054cc9f150ef84ae8f2aa895a62ae109e432bc045b7d5afeb6f9d0b0a44c9de7a2a84f298354baed67728fa57e866af742b6a22f0deaa022a446060c6e339e9151ab1adf118f76e9b5738d00c9fa67c3293399f6ca2c844eaa75d08ca21e592d45f5ac53b433344590ba5c0658f1891a7b9cbe4cd917183af060858813702ca5c0c1d0d296927dc6514a595c4fa02dcde225229881ba9ff8068d8f049004c752f00e715ac4761b34113b716e53142a9449e9f6274b489a5256022f53917673d21e8de4"
}
```

����û���Ϣ

����passport֧�ֶ��������û�,clientid�ǻ���provicder���ŵ����Ի�ȡ�û���Ϣʱ��Ҫ�ṩclient

����ͷ��Ϣ

Authorization Bearer access_token(ע��Bearer���溬�пո�)

X-Client-Id client_id

����������
```
<?php

declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Contract\RequestInterface;
use Hyperf\HttpServer\Contract\ResponseInterface;
use Hyperf\HttpServer\Annotation\Controller;
use Hyperf\HttpServer\Annotation\RequestMapping;
use Hyperf\Di\Annotation\Inject;
use Hyperf\HttpServer\Annotation\Middleware;
use Hyperf\HttpServer\Annotation\Middlewares;
use Richard\HyperfPassport\PassportAuthMiddleware;
use Richard\HyperfPassport\AuthManager;

/**
 * @Controller
 */
class DemoController extends AbstractController {

    /**
     * @Inject
     * @var AuthManager
     */
    protected $auth;

    /**
     * @Middlewares({
     *    @Middleware(PassportAuthMiddleware::class)
     * })
     * @RequestMapping(path="index", methods="get,post,options")
     */
    public function index(RequestInterface $request, ResponseInterface $response) {
        $user = $this->auth->guard('passport')->user();
        //var_dump($user);
        $userId = (int) $user->getId();
        return ['user_id' => $userId];
    }


}
?>
```
