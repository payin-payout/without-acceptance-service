# Создание автоматически оплачиваемого счёта

Для создания автоматически оплачиваемого счёта нужно отправить POST - запрос на адрес
https://lk.payin-payout.net/service-of-services/create-invoice указав следующие параметры:

### Данные передаваемые при создании платежа

|Параметр|Значение|Описание|
|---|---|---|
|service_id   | Целое число, например 50 |Идентификатор сервиса, должен быть получен у менджмента системы Payin-payout  |
|user_id   |Целое число, например 1111   |Идентификатор пользователя от имени которого осуществляется работа с API   |
|payer_id   |Целое число, например 12345   |Идентификатор пользователя с которого должны быть списаны средства   |
|amount   |Число, например 5000   |Сумма платежа в рублях   |
|external_id   |Строка, например invoice_12345   |Идентификатор счёта со стороны магазина, позднее он будет передан в колбеке информирующем о результате обработки счёта   |
|description   |Строка, например "Оплата тарифа" |Описание счёта   |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|hash   |Строка   |[Хэш безопасности](calculate-hash.md) |

### Пример запроса реализованный на PHP

```php
<?php
$paramPost = [
    'service_id' => 40,
    'user_id' => 1111,
    'payer_id' => 2222,
    'amount' => 1000,
    'external_id' => 'invoice_12345',
    'description' => 'description',
    'timestamp' => time(),
];
$hash = hash_hmac('md5', implode($paramPost), 'secret_key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HEADER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/service-of-services/create-invoice');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";

```

### Пример запроса curl

```bash
curl -X POST \
  https://lk.payin-payout.net/service-of-services/create-invoice \
  -H 'cache-control: no-cache' \
  -d 'service_id=40&timestamp=1558429736&user_id=1111&payer_id=2222&amount=1000&external_id=invoice_123452&description=description&hash=9ce2806caa31dba92b1da5443e3993bd'

```

В случае корректного запроса возвращается ответ:

```json
{
    "status": true,
    "result": true,
    "tracker": " gid_5ce3b25a0bcbd4.32457530"
}

```

### Описание данных в ответе

|Параметр|Значение|Описание|
|---|---|---|
|status   | логическое значения   |определяет успешность запроса  |
|result   | логическое значения  |определяет успешность операции  |
|tracker   |строка   |служебная информация   |

В случае некорректного запроса ответ может выглядеть следующим образом:

```json
{
    "status": false,
    "user_msg": "invoice with external_id invoice_123452 already exists",
    "tracker": " gid_5ce3b46610e687.20411654"
}
```

В данном случае переданный идентификатор со стороны магазина уже был использован.

 
## Дальнейшие действия

После создания платежа он поступает в обработку и производится [отправка уведомления](callback-handling.md) на
ранее переданный менеджменту системы Payin-payout адрес. 