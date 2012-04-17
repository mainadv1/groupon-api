Groupon API
===========

Groupon API построен поверх протокола HTTP с использованием принципов REST. На данный момент мы поддерживаем два формата ответа сервера - XML и JSON.

В дальнейшем, в этой документации мы описываем все запросы и ответы сервера в формате JSON. Работа с XML полностью идентична и не должна вызвать затруднений.


Доступ к API
------------

Наш API подразделяет пользователей на несколько ролей. Каждая роль имеет строго определенный уровень функциональности и доступа к информации.

**Merchant** - пользователь, являющийся нашим поставщиком услуг. С помощью Merchant API он имеет возможность получать инфорамцию о своих акциях, купонах и т.п. вне сайта [Groupon](http://groupon.ru).

Примером использования Merchant API может является интернет-магазин по продаже билетов, использующий API для проверки введеных купонов.

**Partner** - пользователь, являющийся нашим партнером. Имеет доступ к информации по текущим акциями в городах, регистрации пользователей и т.п.

Примером использования Partner API могут являться сайты-агрегаторы купонов.

На данный момент предоставляем доступ к API в индивидуальном порядке.


Авторизация
-----------

Прежде, чем воспользоваться нашим API, вам нужно авторизоваться в системе и получить необходимые для дальнейшей работы значения ``api_id`` и ``api_token``.

Для авторизации вам необходимо выполнить POST-запрос по адресу:

    http://api.groupon.ru/v3/sign_in.json

В качестве параметров запроса нужно передать данные от вашего аккаунта на сайте [Groupon](http://groupon.ru):

- ``user[email]`` - ваш адрес электронной почты
- ``user[password]`` - ваш пароль

*Пример авторизации с помощью [Curl](http://ru.wikipedia.org/wiki/CURL):*

```shell
curl -d "user[email]=ivan_ivanov@gmail.com&user[password]=secret" http://api.groupon.ru/v3/sign_in.json
```

Если вы указали корректные данные, и авторизация прошла успешно - вы получите следующий ответ:

```json
{
  "user": {
    "id": 13579,
    "api_id": 24680,
    "api_token": "fc5e038d38a57032085441e7fe7010b0"
  }
}
```

В случае, если вы указали неверные данные - ответ будет следующим:

```json
{
  "error": "Invalid credentials."
}
```

Храните полученные при авторизации данные (``api_id`` и ``api_token``) - они необходимы для выполнения последующих запросов к API.


Формирование запросов
---------------------

Каждый запрос к API должен содержать ряд обязательных параметров. В зависимости от того, за какой информацией вы обращаетесь, параметры будут варьироваться. Однако, есть три параметра, которые всегда должны присутствовать в формируемом запросе:

- ``api_id`` - номер вашего api-аккаунта в системе
- ``signature`` - уникальная подпись вашего запроса
- ``timestamp`` - текущее время вашего сервера/устройства в формате [Unix Timestamp](http://ru.wikipedia.org/wiki/UNIX-%D0%B2%D1%80%D0%B5%D0%BC%D1%8F)

Параметр ``api_id`` служит для того, чтобы идентифицировать вас в системе. Его значение вы получаете при успешной авторизации.

Параметр ``signature`` служит для проверки безопастности выполняемого запроса. О том, как создается подпись запроса, описано в разделе **"Формирование подписи"**.

Параметр ``timestamp`` указывает на точное время, когда был выполнен запрос. Его значение вы указываете самостоятельно.

*Пример правильно сформированного запроса:*

    http://api.groupon.ru/v3/cities.json?api_id=24680&signature=b159366cb50f17bab55013837c92d73a&timestamp=1332792966


Формирование подписи
--------------------

Запросы, выполняемые к нашему API, должны иметь сформированную особым образом подпись, которая передается в качестве параметра ``signature``.

При создании подписи используется три значения. Первые два - ``api_id`` и ``api_token`` - вы получаете при авторизации. Последний - ``timestamp`` - вы указываете самостоятельно (текущее время вашего сервера/устройства в формате [Unix Timestamp](http://ru.wikipedia.org/wiki/UNIX-%D0%B2%D1%80%D0%B5%D0%BC%D1%8F)).

Далее необходимо составить строку, расставив эти значения в алфавитном порядке и отделив их друг от друга с помощью символа ``_`` (нижнее подчеркивание):

    {api_id}_{api_token}_{timestamp}

*Так, например, если ваш ``api_id`` - 24680, ``api_token`` - fc5e038d38a57032085441e7fe7010b0, а ``timestamp`` - 1332792966, то строка будет выглядить следующим образом:*

    24680_fc5e038d38a57032085441e7fe7010b0_1332792966

От этой строки нужно получить MD5-хеш, который и будет являться подписью запроса.

*Таким образом, верной подписью в нашем примере будет следующий хеш, полученный от строки выше:*

    b159366cb50f17bab55013837c92d73a

Если подпись сформирована неверно - при попытке выполнения запроса с ней вы получите ошибку:

```json
{
  "error": "Incorrect signature."
}
```


Возможные ошибки
----------------

При выполнении запросов у вас могут возникнуть ошибки. Ниже приведен список с описанием каждой ошибки.

- ``Access denied.`` - У вас недостаточно прав для выполнения запроса. Например, если вы пытаетесь обратиться к Merchant API, но при этом не являетесь мерчантом.
- ``Unable to load API user.`` - Не удалось загрузить пользователя API. Следует проверить передаваемый ``api_id``.
- ``Incorrect signature.`` - Неверно сформированная подпись.
- ``Not found.`` - Ресурс, к которому пытаются обратиться, не найден.