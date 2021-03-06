# Подключение к базе данных в кластере {{ PG }}

К хостам кластера {{ mpg-short-name }} можно подключиться:

{% include [cluster-connect-note](../../_includes/mdb/cluster-connect-note.md) %}

{% note info %}

Если публичный доступ в вашем кластере настроен только для некоторых хостов, автоматическая смена мастера может привести к тому, что вы не сможете подключиться к мастеру из интернета.

{% endnote %}


## Автоматический выбор хоста-мастера {#automatic-master-host-selection}

### С библиотекой libpq {#using-libpq}

Чтобы гарантированно подключиться к хосту-мастеру, укажите FQDN всех хостов кластера в аргументе `host` и передайте параметр `target_session_attrs=read-write`, как в сделано [в большинстве примеров](#connection-string) ниже. Этот параметр поддерживается библиотекой `libpq` начиная с [версии 10](https://www.postgresql.org/docs/10/static/libpq-connect.html).

Чтобы обновить версию библиотеки, которую использует утилита `psql`:
* Для дистрибутивов Linux на основе Debian — установите пакет `postgresql-client-10` или новее (например, через [apt-репозиторий](https://www.postgresql.org/download/linux/ubuntu/)).
* Для ОС, использующих RPM-пакеты, дистрибутив {{ PG }} доступен в [yum-репозитории](https://yum.postgresql.org/).

### С драйвером, поддерживающим только один хост {#with-a-driver-that-supports-only-one-host}

Если ваш драйвер для подключения к базе данных не позволяет передавать несколько хостов в строке подключения (например, [pg в Node.js](https://www.npmjs.com/package/pg)), используйте для подключения к кластеру [особый FQDN](#fqdn-master), который указывает на хост-мастер.


## Настройка SSL-сертификата {#configuring-an-ssl-certificate}

{{ PG }}-хосты с публичным доступом поддерживают только соединения с SSL-сертификатом. Подготовить сертификат можно так:


```bash
mkdir ~/.postgresql && \
wget "https://{{ s3-storage-host }}{{ pem-path }}" -O ~/.postgresql/root.crt && \
chmod 0600 ~/.postgresql/root.crt
```


## Примеры строк подключения {#connection-string}

{% include [conn-strings-environment](../../_includes/mdb/mdb-conn-strings-env.md) %}

Вы можете подключаться к {{ PG }}-хостам в публичном доступе только с использованием SSL-сертификата. Перед подключением к таким хостам [подготовьте сертификат](#configuring-an-ssl-certificate).

В этих примерах предполагается, что SSL-сертификат `root.crt` расположен в директории `/home/<домашняя директория>/.postgresql/`. 

Подключение без использования SSL-сертификата поддерживается только для хостов, находящихся не в публичном доступе. В этом случае трафик внутри виртуальной сети при подключении к БД шифроваться не будет.

Подключиться к кластеру возможно как с использованием обычных FQDN хостов (можно передавать список из нескольких таких FQDN, разделенных запятой), так и [особых FQDN](#special-fqdns).

{% include [see-fqdn-in-console](../../_includes/mdb/see-fqdn-in-console.md) %}

{% include [mpg-connection-strings](../../_includes/mdb/mpg-conn-strings.md) %}

При успешном подключении к кластеру и выполнении тестового запроса будет выведена версия {{ PG }}.


## Особые FQDN {#special-fqdns}

Наравне с обычными FQDN, которые можно запросить со [списком хостов в кластере](hosts.md#list), {{ mpg-name }} предоставляет несколько особых FQDN, которые также можно использовать при подключении к кластеру.

### Текущий мастер {#fqdn-master}

FQDN вида `c-<идентификатор кластера>.rw.{{ dns-zone }}` всегда указывает на текущий хост-мастер в кластере. Идентификатор кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters). 

При подключении к этому FQDN разрешено выполнять операции чтения и записи.

Пример подключения к хосту-мастеру для кластера с идентификатором `c9qash3nb1v9ulc8j9nm`:

```bash
psql "host=c-c9qash3nb1v9ulc8j9nm.rw.{{ dns-zone }} \
      port=6432 \
      sslmode=verify-full \
      dbname=<имя БД> \
      user=<имя пользователя> \
      target_session_attrs=read-write"
```

### Наименее отстающая реплика {#fqdn-replica}

FQDN вида `c-<идентификатор кластера>.ro.{{ dns-zone }}` указывает на наименее отстающую от мастера [реплику](../concepts/replication.md). Идентификатор кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters). 

**Особенности:**
- При подключении к этому FQDN разрешено выполнять только операции чтения.
- Если в кластере нет активных реплик, то подключиться к этому FQDN невозможно: соответствующая CNAME-запись в DNS будет указывать «в никуда» (`null`).

Пример подключения к наименее отстающей реплике для кластера с идентификатором `c9qash3nb1v9ulc8j9nm`:

```bash
psql "host=c-c9qash3nb1v9ulc8j9nm.ro.{{ dns-zone }} \
      port=6432 \
      sslmode=verify-full \
      dbname=<имя БД> \
      user=<имя пользователя> \
      target_session_attrs=any"
```