# Выставление счёта для пополнения баланса

В случае если пользователь имеет баланс недостаточный для оплаты услуги он должен пополнить свой
баланс (проверить баланс пользователя [можно данным запросом](get-balance.md)).

Для пополнения баланса пользователя (который затем используется для автоматических списаний)
пользователь должен либо самостоятельно создать счёт, либо счёт можно создать для него
запросом на метод API.

## Запрос на создание счёта

Для создания счёта нужно отправить POST - запрос на адрес
https://lk.payin-payout.net/payment-in/create-invoice указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|topup_user_id   | Целое число, например 12345   |Идентификатор пользователя счёт которого требуется пополнить  |
|amount   | Число, например 10000   |Сумма в рублях на которую требуется пополнить баланс   |
|user_id   | Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|preference | Целое число (необязательный параметр) | [Выбранный способ оплаты](https://github.com/payin-payout/payin-api#Коды-типов-платежей), если этот параметр выбран, пользователь может миновать этап выбора способа оплаты |
|success_url | Строка (необязательный параметр) |Адрес на который будет перенаправлен пользовать в случае успешной оплаты|
|fail_url | Строка (необязательный параметр) |Адрес на который будет перенаправлен пользовать в случае если оплата не будет успешной|
|hash   | Строка  |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
<?php

$paramPost = [
    'topup_user_id' => 1111,
    'amount' => '1500',
    'user_id' => 2222,
    'timestamp' => time(),
];

$hash = hash_hmac('md5', implode($paramPost), 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HEADER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/payment-in/create-invoice');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";
```

### Пример запроса curl

```bash
curl -X POST \
  https://lk.payin-payout.net/payment-in/create-invoice \
  -H 'cache-control: no-cache' \
  -d 'topup_user_id=1111&timestamp=1558429736&amount=1500&user_id=2222&hash=7ef310b3e56e29c1b1928e4c556419eb'
```

В случае корректного запроса возвращается ответ:

```json
{
  "status": true,
  "result": {
    "pay_link": "https://lk.payin-payout.net/api/pay/64417825388265#/index"
  },
  "": " gid_5ce396ae761767.57601823"
}
```

### Описание данных в ответе

|Параметр|Значение|Описание|
|---|---|---|
|status   | логическое значения   |определяет успешность операции  |
|pay_link   |текстовая строка   |ссылка по которой нужно перейти пользователю для оплаты  |
|tracker   |строка   |служебная информация   |

