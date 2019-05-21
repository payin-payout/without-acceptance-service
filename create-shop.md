# Создание онлайн магазина

Для того чтобы воспользоваться сервисом безакцептных платежей необходимо иметь магазин в системе
Payin-payout и сообщить его идентификатор менеджменту для подключения к сервису.

Если такой магазин отсутствует, его необходимо создать.

## Запрос на создание магазина

Для создания счёта нужно отправить POST - запрос на адрес
https://lk.payin-payout.net/shop/api-create указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|title   | Строка, название магазина   |Название магазина  |
|address   |Строка, адрес магазина   |Юридический адрес магазина   |
|user_id   |Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|hash   |Строка   |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
<?php

$paramPost = [
    'title' => 'title1',
    'address' => 'address2',
    'user_id' => 7026,
    'timestamp' => time(),
];

$hash = hash_hmac('md5', implode($paramPost), 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_VERBOSE, true);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HEADER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/shop/api-create');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";

```

### Пример запроса curl

```bash
curl -X POST \
  https://lk.payin-payout.net/shop/api-create \
  -H 'cache-control: no-cache' \
  -d 'title=7026&timestamp=1558429736&address=%20some%20address&user_id=7026&hash=99fa556d7082890be0da33144249e3f8'
```

В случае корректного запроса возвращается ответ:

```json
{
    "status": true,
    "result": 2507,
    "tracker": " gid_5ce3acc17e5290.35823848"
}
```

### Описание данных в ответе

|Параметр|Значение|Описание|
|---|---|---|
|status   | логическое значения   |определяет успешность операции  |
|result   |целое число  |идентификатор созданного магазина  |
|tracker   |строка   |служебная информация   |

 