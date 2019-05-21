# Регистрация нового пользователя в системе

## Запрос на регистрацию пользователя

Для регистрации нового пользователя нужно отправить POST - запрос на адрес
https://lk.payin-payout.net/user/api-create указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|ul   | 1 либо 0   |Является ли регистрируемый пользователь юридическим лицом, 1 если является, 0 если нет   |
|email   |Строка, например email@email.ru   |Почта регистрируемого пользователя, на почту будет отправлен код подтверждения   |
|phone   |Строка, например +79111111111   |Телефон регистрируемого пользователя, будет отправлено смс с кодом подтверждения   |
|username   |Строка, например username   |Имя пользователя   |
|password   |Строка, например, password   |Пароль пользователя   |
|resident   |1 либо 0 |Является ли регистрируемый пользователь резидентом РФ, 1 если является, 0 если нет   |
|user_id   |Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|hash   |Строка   |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
<?php

$post = [
    'ul' => 1,
    'email' => 'email@email.ru',
    'phone' => '+79111111111',
    'username' => 'username',
    'password' => 'password',
    'resident' => 1,
    'user_id' => 1111,
    'timestamp' => time(),
];
$hash = hash_hmac('md5', implode($post), 'secret_key');
$post['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $post);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/user/api-create');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";
```

### Пример запроса curl

```bash
curl -X POST \
  https://lk.payin-payout.net/user/api-create \
  -H 'cache-control: no-cache' \
  -d 'ul=1&timestamp=1558429736&email=email_test%40email.ru&phone=%2B79239999999&username=test_test&password=password&resident=1&hash=54509a826d079c4fddfd8456a753f007&user_id=1111'
```

В случае корректного запроса возвращается ответ:

```json
{
    "status": true,
    "result": 22222,
    "tracker": " gid_5ce24b50061366.52192801"
}
```

status информирует о успешности запроса 

result содержит идентификатор созданного пользователя

tracker содержит служебную информацию

В случае некорректного запроса ответ может выглядеть следующим образом:

```json
{
    "status": false,
    "user_msg": "EXIST_IN_SYSTEM",
    "tracker": " gid_5ce2541cd31318.27324826"
}
```

В данном случае пользователь с переданными данными уже существует в системе.

## Подтверждение email и телефона пользователя 

После регистрации пользователя неободимо подтвердить его почту и телефон с помощью кодов
которые были ему отправлены.

Это делается запросом на url 
https://lk.payin-payout.net/user/api-confirm-phone-and-email указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|phone_code   | Целое число, например 3333   |Код подтверждения который был отправлен в смс  |
|email_code   | Целое число, например 4444   |Код подтверждения который был отправлен в email   |
|registered_user_id   |Целое число, например 12345   |Идентификатор ранее зарегистрированного пользователя   |
|user_id   |Целое число, 1111   |Идентификатор пользователя который осуществляет работу с API   |
|hash   |Строка   |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
<?php

$paramPost = [
    'phone_code' => '8789',
    'email_code' => '5995',
    'registered_user_id' => '12345',
    'user_id' => 1111,
    'timestamp' => time(),
];
$hash = hash_hmac('md5', implode($paramPost), 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HEADER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);

curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/user/api-confirm-phone-and-email');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";
```

### Пример запроса curl

```bash
curl -X POST \
  https://lk.payin-payout.net/user/api-confirm-phone-and-email \
  -H 'cache-control: no-cache' \
  -d 'phone_code=3032&timestamp=1558429736&email_code=3189&registered_user_id=12345&user_id=1111&hash=7215adfa085ac058f2051a4035963ee2'
```

В случае корректного запроса возвращается ответ:

```json
{
    "status": true,
    "result": true,
    "tracker": " gid_5ce24b50061366.52192801"
}
```