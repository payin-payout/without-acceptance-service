# Получение баланса пользователя

Для того чтобы проверить достаточно ли средств на счету имеет пользователь для оплаты возможно
запросить его баланс.  

## Запрос на проверку баланса

Для получения баланса нужно отправить POST - запрос на адрес
https://lk.payin-payout.net/user/api-get-balance указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|user_id   |Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|balance_user_id   |Целое число, например 2222   |Идентификатор пользователя баланс которого запрашивается  |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|hash   |Строка   |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
<?php

$paramPost = [
    'user_id' => 1111,
    'balance_user_id' => 2222,
    'timestamp' => time(),
];

$hash = hash_hmac('md5', implode($paramPost), 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/user/api-get-balance');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";

```

### Пример запроса curl

```bash
curl -X POST \
  https://lk.payin-payout.net/user/api-get-balance \
  -H 'cache-control: no-cache' \
  -d 'timestamp=1558429736balance_user_id=1111&user_id=2222&hash=99fa556d7082890be0da33144249e3f8'
```

В случае корректного запроса возвращается ответ:

```json
{
  "status": true,
  "result": 12425.56,
  "tracker": " gid_5ce4b4bfb671a2.64203729"
}
```

### Описание данных в ответе

|Параметр|Значение|Описание|
|---|---|---|
|status   | логическое значения   |определяет успешность операции  |
|result   |число  |баланс рублёвого счёта |
|tracker   |строка   |служебная информация   |

 