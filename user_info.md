# Получение информации о пользователе

Необходимо отправить POST - запрос на адрес
https://lk.payin-payout.net/user/get_user_info_via_dev_key указав следующие параметры:

|Параметр|Значение|Описание|
|---|---|---|
|user_id   | Целое число, например 1111   |Идентификатор пользователя который осуществляет работу с API   |
|timestamp   | Целое число, время Unix в секундах | [Текущее время Unix](calculate-hash.md#Метка-текущего-времени-в-параметрах) в секундах в UTC |
|email   | Строка | email пользователя (либо можно указать phone) |
|phone   | Строка |телефон пользователя (либо можно указать email)  |
|hash   | Строка  |[Хэш безопасности](calculate-hash.md)   |

### Пример запроса реализованный на PHP

```php
$paramPost = [
    'user_id' => 11111,
    'email' => 'search_for@email.com',
    'timestamp' => time(),
];
uksort($paramPost, 'strcasecmp');
$data_string = http_build_query($paramPost, '', '&');
$hash = hash_hmac('md5', $data_string, 'key');
$paramPost['hash'] = $hash;
$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_HTTPHEADER, [
    'Accept: application/json' // чтобы получить ошибку в случае её возникновения в json
]);
curl_setopt($curl, CURLOPT_POSTFIELDS, $paramPost);
curl_setopt($curl, CURLOPT_URL, 'https://lk.payin-payout.net/user/get_user_info_via_dev_key');
$out = curl_exec($curl);
echo "\nout:\n$out";
$error = curl_error($curl);
echo "\nerror:\n$error";
```

В случае корректного запроса, если пользователь найден, возвращается ответ:

```json
{
  "status": true,
  "result": {
    "inner_id": 2222,
    "email": "search_for@email.com",
    "phone": "71112223344"
  },
  "tracker": " gid_5f2bb49d51dfb5.80455021"
}
```

Если пользователь с указанным email или phone не был найден:

```json
{
  "status": true,
  "result": [],
  "tracker": " gid_5f2bb51fc59396.10172298"
}
```

### Описание данных в ответе

|Параметр|Значение|Описание|
|---|---|---|
|status   | логическое значения   |определяет успешность операции  |
|result   |массив с данными пользователя | пользователь (если таковой найден)  |
|inner_id   |Идентификатор   |Идентификатор найденного пользователя  |
|email   |email   |email найденного пользователя  |
|phone   |phone   |phone найденного пользователя  |

