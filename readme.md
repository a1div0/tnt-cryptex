# Extended cryptography library for Tarantool
## Intro
This library contain:
- lua-openssl (zhaozg)
- jwt-module

## External dependencies
- OpenSSL >= 1.1

## Мотивация
1) В версии Tarantool 2.10 нет доступа к функциям работы с RSA-алгоритмом, ключами PKCS8 и т.п.
2) Такой функционал есть в библиотеке OpenSSL. Это стандартная библиотека.
   Однако в разных версиях ОС версия этой библиотеки отличается. Так например в
   CentOS 7 или RetHat 7 используется версия OpenSSL 1.0.2k. В CentOS 8,
   Ubuntu 20 используется версия OpenSSL 1.1.
3) Я попробовал работу и с версией 1.0 и с версией 1.1. Версия 1.0 менее стабильна
   и на некоторых тестах падает, причём в зависимости от ОС - падает по-разному.
   Версия 1.1 работает стабильно на всех тестах.

## Install
```shell
git clone --recurse https://github.com/a1div0/tnt-cryptex.git cryptex
cd cryptex
make rock
```

## JWT
### function jwt.encode(data, key, alg)
Создаёт JWT-токен на основе входных данных.
В случае использования ассиметричных алгоритмов шифрования, в качестве ключа
следует использовать приватный ключ в формате PEM (PKCS8)

#### Параметры
**data (table)** - Полезная нагрузка (данные) в виде таблицы. Например имя пользователя и перечень его прав.

**key (string)** - Ключ, используемый для подписи данного токена

**alg (string)** - Необязательный. Алгоритм формирования подписи, по умолчанию = HS256

#### Результат
(string) JWT-токен

### function jwt.decode(jwt_token, key, verify_prepare_func, verify)
Декодирует JWT-токен и проверяет подпись. В случае использования ассиметричных алгоритмов
шифрования, в качестве ключа следует использовать публичный ключ в формате PEM (PKCS8)

#### Параметры
**jwt_token (string)** - JWT-токен

**key (string)** - Ключ, используемый для подписи данного токена (JSON Web Key)

**verify_prepare_func (function)** - Необязательный.
Функция, вызываемая перед проверкой, параметр token_items содержит декодированные данные, params содержит ключ
Пример реализации:
```lua
local function verify_prepare_func(token_items, params)
    if token_items.header.alg == "RS256" then
        params.key = '-----BEGIN PUBLIC KEY-----\n' .. params.key .. '\n-----END PUBLIC KEY-----'
    end
end
```

**verify (bool)** - Необязательный. Параметр для отладочной среды. Если передать false - подпись проверяться не будет. По умолчанию = true.

#### Результат
(table) Верифицированное тело токена

## Examples
```lua
local cryptex = require('cryptex')
local openssl = cryptex.openssl -- Docs: https://zhaozg.github.io/lua-openssl/index.html
local jwt = cryptex.jwt
local keys = cryptex.generate_rsa_pem_keys()
```
