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

uksort($paramPost, 'strcasecmp');
$data_string = http_build_query($paramPost, '', '&');
$hash = hash_hmac('md5', $data_string, 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_HTTPHEADER, [
    'Accept: application/json' // чтобы получить ошибку в случае её возникновения в json
]);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/user/api-get-balance');
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
    "840": "0.02000000",
    "999": "0.00000000",
    "980": "0.00000000",
    "978": "29.00000000",
    "398": "0.00000000",
    "643": "8726.47000000",
    "826": "0.00000000"
  },
  "tracker": " gid_5ed772db10bc57.06308154"
}
```

### Описание данных в ответе

|Параметр|Значение|Описание|
|---|---|---|
|status   | логическое значения   |определяет успешность операции  |
|result   |число  |баланс рублёвого счёта |
|tracker   |строка   |служебная информация   |

