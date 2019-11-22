# Регистрация нового пользователя в системе

## Запрос на регистрацию пользователя

Для регистрации нового пользователя нужно отправить POST - запрос на адрес
https://lk.payin-payout.net/user/api-create указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|ul   | 1 либо 0   |Является ли регистрируемый пользователь юридическим лицом, 1 если является, 0 если нет   |
|email   |Строка, например email@email.ru   |Почта регистрируемого пользователя   |
|phone   |Строка, например +79111111111   |Телефон регистрируемого пользователя   |
|username   |Строка, например username   |Имя пользователя   |
|password   |Строка, например, password   |Пароль пользователя   |
|resident   |1 либо 0 |Является ли регистрируемый пользователь резидентом РФ, 1 если является, 0 если нет   |
|user_id   |Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|source_registration   |Опционально, строка   |Опциональный источник регистрации|
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
    'source_registration' => 'some_source',
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
  -d 'ul=1&timestamp=1558429736&email=email_test%40email.ru&phone=%2B79239999999&username=test_test&password=password&resident=1&hash=54509a826d079c4fddfd8456a753f007&user_id=1111&source_registration=some_source'
```

В случае корректного запроса возвращается ответ:

```json
{
  "status": true,
  "result": {
    "id": 22222,
    "tips_url": "https://lk.payin-payout.net/tips/pay-to-user?userId=64389500694851",
    "card_url": "https://lk.payin-payout.net/card/64389500694851.jpeg",
    "invite_url": "https://lk.payin-payout.net/invite/viOqD0"
  },
  "tracker": " gid_5dd75fe7c400e8.58224878"
}

```

status информирует о успешности запроса 

id - содержит идентификатор созданного пользователя

tips_url - ссылка для оплаты чаевых в пользу созданного пользователя

card_url - ссылка на визитку пользователя

invite_url - ссылка для регистрации по приглашению созданного пользователя

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
