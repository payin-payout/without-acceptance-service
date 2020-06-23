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
    'ul' => 1, // является ли регистрируемый пользователь юридическим лицом
    'email' => 'sourcetest4@email.ru', // почта пользователя
    'phone' => '+7111223344', // телефон пользователя
    'username' => 'username', // имя пользователя
    'password' => 'dfg5t54ht435', // пароль
    'resident' => 1, // является ли пользователь резидентом РФ
    'timestamp' => time(), // временная метка обеспечивающая уникальность подписи
    'source_registration' => 'some_source', // источник регистрации
    'user_id' => 12345, // идентификатор пользователя от имени которого ведётся работа с API
];

uksort($post, 'strcasecmp');
$data_string = http_build_query($post, '', '&');
$hash = hash_hmac('md5', $data_string, 'some_key');
$post['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $post);
curl_setopt($curl, CURLOPT_HTTPHEADER, [
    'Accept: application/json' // чтобы получить ошибку в случае её возникновения в json
]);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/user/api-create');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";

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
