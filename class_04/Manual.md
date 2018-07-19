#DNS-сервер BIND 9 на Cent OS 7

### 1) Устанавливаем BIND:

```bash
sudo yum install -y bind bind-utils
```

### 2) Открываем файл /etc/named.conf

```bash
sudo vi /etc/named.conf
```

### 3) Вносим изменения в открытый файл /etc/named.conf
 В открывшемся файле в параметры listen-on и listen-on-v6 добавляем IPv4 и IPv6 адреса текущей машины в сети (узнаём их через команду ifconfig, если ещё не знаем), изменённые строчки будут выглядеть так:

```
listen-on port 53 { 127.0.0.1; 10.219.180.13; };
listen-on-v6 port 53 { ::1;  fe80::3d:f5ff:fe4f:563b; };
```

Параметру recursion присваиваем значение no (требуется для кэширующих серверов, для авторитативных не нужен), а параметру allow_query (от каких клиентов принимать запросы) - значение any:

```
allow-query { any; };
recursion no;
```

В конец файла добавляем зону, за которую наш dns-сервер отвечает. Имя зоны - fintechtestzone, тип авторитативного сервера - master (slave использовать не будем), имя файла (который будет лежать в директории /var/named) - fintechtestzone.zone (в принципе, можно выбрать любое другое).  Получаем следующее:

```
zone "fintechtestzone" IN {
    type master;
    file "fintechtestzone.zone";
    allow-transfer { none; };
    allow-update { none; };
};
```

### 4) Выходим из редактора vi с сохранением изменений
Нажимаем Shift + ':',  следом пишем 'wq' и нажимаем Enter

### 5) Создаём файл зоны
Имя создаваемого файла - 'fintechtestzone.zone' (или другое, использованное в зоне в  /etc/named.conf)

```bash
sudo touch /var/named/fintechtestzone.zone
```

### 6) Открываем созданный файл зоны

```bash
sudo vi /var/named/fintechtestzone.zone
```

### 7) Вписываем необходимые записи в файл зоны

```
$TTL 1d
@ IN  SOA ns.fintechtestzone zakhse.yandex.ru (
  2018071901 ;Serial
  3h         ;Refresh
  1h         ;Retry
  1w         ;Expire
  1d         ;TTL
)


  IN NS     s-6.fintech-admin.m1.tinkoff.cloud.
  IN TXT    "This is not the domain you are looking for"
  IN A      127.0.0.1
```

Тут важно не забыть поставить пробелы перед строками после SOA-записи (т.е. перед записями NS, TXT, A), иначе файл будет невалидным и DNS-сервер не запустится.
### 8) Выходим из редактора vi с сохранением изменений

### 9) Запускаем DNS-сервер:

```bash
sudo systemctl start named
```

### 10) Добавляем в автозапуск:

```bash
sudo systemctl enable named
```

### P.S.:
Если сервис не будет запускаться из-за каких-то ошибок, очень помогает выполнение следующей команды:

```bash
journalctl -xe
```

Лог очень хороший. И про неправильные права доступа расскажет, и про неверный синтаксис, например, пропущенную точку с запятой. Как можно уже догадаться, автору этой инструкции пришлось не один раз выполнять эту команду =)
