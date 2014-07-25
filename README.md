# ydcmd

Консольный клиент Linux/FreeBSD для работы с облачным хранилищем [Яндекс.Диск](https://disk.yandex.ru/) посредством [REST API](http://api.yandex.ru/disk/api/concepts/about.xml).

## Подготовка к работе

Для работы клиента необходимо получить отладочный OAuth токен. Для его получения, [зарегистрируйте приложение на Яндексе](https://oauth.yandex.ru/client/new):

* `Название` - `ydcmd` (может быть любым)
* `Права` - `Яндекс.Диск REST API`
* `Клиент для разработки` - установить флажок

После регистрации приложения скопируйте `id приложения` и перейдите по ссылке:

* `https://oauth.yandex.ru/authorize?response_type=token&client_id=<id_приложения>`

После разрешения доступа сервис перенаправит вас по ссылке вида:

* `https://oauth.yandex.ru/verification_code?dev=True#access_token=<токен>`

Значение "токен" и есть требуемое. Подробнее можно ознакомиться по ссылке [получение отладочного токена вручную](http://api.yandex.ru/oauth/doc/dg/tasks/get-oauth-token.xml).

## Работа

Вывод краткой справки в консоли можно получить запуском скрипта без параметров или с командой `help`. Общий формат вызова:

```
ydcmd [команда] [опции] [аргументы]
```

**Команды**:

* `help` - получение краткой справки по командам и опциям приложения;
* `ls` - получение списка файлов и директорий;
* `rm` - удаление файла или директории;
* `cp` - копирование файла или директории;
* `mv` - перемещение файла или директории;
* `put` - загрузка файла или директории в хранилище;
* `get` - получение файла или директории из хранилища;
* `mkdir` - создание директории;
* `stat` - получение метаинформации об объекте;
* `du` - оценка места, занимаемого файлами в хранилище;
* `clean` - очистка файлов и директорий.

**Опции**:

* `--timeout=<N>` - таймаут в секундах на установку сетевого соединения;
* `--retries=<N>` - количество попыток вызова метода api перед возвратом кода ошибки;
* `--delay=<N>` - таймаут между попытками вызова метода api в секундах;
* `--limit=<N>` - количество элементов, возвращаемое одним вызовом метода получения списка файлов и директорий;
* `--token=<S>` - oauth токен (в целях безопасности рекомендуется указывать в конфигурационном файле);
* `--quiet` - подавление вывода об ошибках, результат успеха операции определяется по коду возврата;
* `--verbose` - вывод расширенной информации;
* `--debug` - вывод отладочной информации;
* `--chunk=<N>` - размер блока данных в КБ для операций ввода/вывода;
* `--ca-file=<S>` - имя файла с сертификатами доверенных центров сертификации (при пустом значении проверка валидности сертификата не производится);
* `--ciphers=<S>` - набор алгоритмов шифрования.

### Получение списка файлов и директорий

```
ydcmd ls [опции] [disk:/объект]
```

**Опции**:

* `--human` - вывод размера файла в человеко-читаемом виде;
* `--short` - вывод списка файлов и директорий без дополнительной информации (одно имя в одну строку);
* `--long` - вывод расширенного списка (время создания, время модификации, размер, имя файла).

Если целевой объект не указан, то будет использоваться корневая директория хранилища.

### Удаление файла или директории

```
ydcmd rm disk:/объект
```

**Опции**:

* `--poll=<N>` - время в секундах между опросом состояния при выполнении асинхронной операции;
* `--async` - выполнение команды без ожидания завершения (`poll`) операции.

Файлы удаляются без возможности восстановления. Директории удаляются рекурсивно (включая вложенные файлы и директории).

### Копирование файла или директории

```
ydcmd cp disk:/объект1 disk:/объект2
```

**Опции**:

* `--poll=<N>` - время в секундах между опросом состояния при выполнении асинхронных операций;
* `--async` - выполнение команды без ожидания завершения (`poll`) операции.

В случае совпадения имен, директории и файлы будут перезаписаны. Директории копируются рекурсивно (включая вложенные файлы и директории).

### Перемещение файла или директории

```
ydcmd mv disk:/объект1 disk:/объект2
```

**Опции**:

* `--poll=<N>` - время в секундах между опросом состояния при выполнении асинхронных операций;
* `--async` - выполнение команды без ожидания завершения (`poll`) операции.

В случае совпадения имени, директории и файлы будут перезаписаны.

### Загрузка файла в хранилище

```
ydcmd put <файл> [disk:/объект]
```

**Опции**:

* `--rsync` - синхронизация дерева файлов и директорий в хранилище с локальным деревом;
* `--encrypt` - шифрование файлов при помощи `--encrypt-cmd` перед операцией загрузки в хранилище;
* `--encrypt-cmd` - команда получающая на `stdin` содержимое локального (нешифрованного) файла и отдающее в `stdout` его шифрованную версию;
* `--temp-dir` - директория для хранения временных зашифрованных файлов.

Если целевой объект не указан, то для загрузки файла будет использоваться корневая директория хранилища. Если целевой объект указывает на директорию (заканчивается на `/`), то к имени директории будет добавлено имя исходного файла. Если целевой объект существует, то он будет перезаписан без запроса подтверждения. Символические ссылки игнорируются.

### Получение файла из хранилища

```
ydcmd get <disk:/объект> [файл]
```

**Опции**:

* `--rsync` - синхронизация локального дерева файлов и директорий с деревом в хранилище;
* `--decrypt` - расшифровка файлов при помощи `--decrypt-cmd` после операции получения из хранилища;
* `--decrypt-cmd` - команда получающая на `stdin` содержимое шифрованного файла из хранилища и отдающее в `stdout` его дешифрованную версию;
* `--temp-dir` - директория для хранения временных зашифрованных файлов.

Если не указано имя целевого файла, будет использовано имя файла в хранилище. Если целевой объект существует, то он будет перезаписан без запроса подтверждения.

### Создание директории

```
ydcmd mkdir disk:/путь
```

### Получение метаинформации об объекте

```
ydcmd stat [disk:/объект]
```

Если целевой объект не указан, то будет использоваться корневая директория хранилища.

### Оценка занимаемого места

```
ydcmd du [disk:/объект]
```

**Опции**:

* `--depth=<N>` - отображать размеры директорий до уровня N;
* `--long` - отображать размеры в байтах вместо человеко-читаемого вида.

Если целевой объект не указан, то будет использоваться корневая директория хранилища.

### Очистка файлов и директорий

```
ydcmd clean <опции> [disk:/объект]
```

**Опции**:

* `--dry` - не выполнять удаление, а вывести список объектов для удаления;
* `--type=<S>` - тип объектов для удаления (`file` - файлы, `dir` - директории, `all` - все);
* `--keep=<S>` - критерий выборки объектов, которые требуется сохранить:
  * Для выбора даты **до** которой требуется удалить данные, можно использовать строку даты в формате ISO (например, `2014-02-12T12:19:05+04:00`);
  * Для выбора относительного времени, можно использовать число и размерность (например, `7d`, `4w`, `1m`, `1y`);
  * Для выбора количества копий, можно использовать число без размерности (например, `31`).

Если целевой объект не указан, то будет использоваться корневая директория хранилища. Сортировка и фильтрация объектов производится по дате модификации (не по дате создания).

## Конфигурация

Для удобства работы рекомендуется создать конфигурационный файл с именем `~/.ydcmd.cfg` и установить на него права `0600` или `0400`. Формат файла:

```
[ydcmd]
# комментарий
<option> = <value>
```

Например:

```
token   = 1234567890
verbose = yes
ca-file = /etc/ssl/certs/ca-certificates.crt
```

## Код выхода

При работе в автоматическом режиме (по cron) может быть полезно получить результат выполнения команды:

* `0` - успешное завершение;
* `1` - общая ошибка приложения;
* `4xx` - код состояния HTTP-4xx (ошибка клиента);
* `5xx` - код состояния HTTP-5xx (ошибка сервера).
