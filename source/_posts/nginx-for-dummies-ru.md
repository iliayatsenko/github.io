---
title: NGINX для чайников
date: 2022-05-30
tags:
    - nginx
categories:
    - RU
---

Чаще всего когда мне надо сделать так, чтоб какой-то URL вызывал какой-то PHP код, приходится иметь дело с конфигом nginx. 

<!-- more -->

И каждый раз это превращается в долгие страдания и заканчивается ~~громкими матами~~ полным отчаянием.

Даже однажды разобравшись в том как это делается, всё равно успеваешь всё забыть до того как это понадобится в следующий раз. И опять всё сначала. Кстати, помимо nginx есть еще несколько подобных контр-интуитивных вещей, которые никак не удаётся “выучить раз и навсегда”: например, язык bash, или регулярные выражения. Про bash есть даже шутка:

[https://twitter.com/jakewharton/status/1334177665356587008](https://twitter.com/jakewharton/status/1334177665356587008)

Поэтому я решил очень кратко законспектировать основные правила, по которым nginx разруливает запросы, чтоб в следующий раз иметь быстрое напоминание, без необходимости перечитывать документацию. Тут поверхностно описаны только самые часто-употребляемые директивы, так что на звание “руководства” не претендую.

**http**

Итак. Конфигурация состоит из вложенных друг в друга “**контекстов**”, отделенных символами `{`  и `}`. Директивы распространяются на контекст в котором они объявлены и на все вложенные в него, при этом вложенные контексты могут переопределять эти директивы - примерно как одноименные переменные во вложенной функции в javascript перекрывают переменные из более “внешней” функции. Всё начинается в так называемом “**главном контексте**” - это всё что снаружи фигурных скобок. Этот контекст нам не очень интересен, там задаются какие-то дефолтные настройки. Интересен вложенный в него контекст `http`, а в нем - контекст(ы) `server`.

**server**

Один контекст `server` содержит директивы для одного “**виртуального сервера**”, и таких серверов может быть множество. При поступлении HTTP-запроса, nginx определяет соответствующий виртуальный сервер по запрашиваемому IP-адресу, порту и содержимому заголовка `Host`. Пример:

```nginx
 # "Главный" контекст

http {
	# Какие-то общие настройки по-умолчанию,
	# могут быть переопределены во вложенных контекстах
	client_max_body_size 100M;

	# Виртуальные сервера:
	
	# 1. этот конфиг будет применен при запросах на порт 80 
	# с заголовком Host: mysite1.com или Host: mysite2.com
	server {
		listen 80;
		server_name mysite1.com mysite2.com;
	}

	# 2. этот конфиг будет применен при запросах на порт 80 
	# с заголовком Host:another-mysite.com
	server {
		listen 80;
		server_name another-mysite.com;
	}

	# 3. а этот - при запросах на порт 80 
	# со всеми другими значениями заголовка Host или вообще без него.
	server {
		# конфигурация по-умолчанию для запросов на порт 80
		listen 80 default_server;
	}

	# ВАЖНО и неочевидно: если Host не совпал ни с одним из server_name, 
	# и default_server для порта не указан явно, 
	# будет применен конфиг ПЕРВОГО виртуального сервера для этого порта.
	# То есть, при конфиге такого вида:
	server {
		listen 192.168.1.17:81;
		server_name mysite1.com;
	}
	server {
		listen 192.168.1.1:81;
		server_name mysite2.com;
	}
	# запрос к 192.168.1.1:81 c заголовком Host: anothersite.com будет обработан
	# первым виртуальным сервером, несмотря на несовпедение Host с server_name
}
```

Директива `server_name` может содержать регулярные выражения или переменные, подробнее тут: [http://nginx.org/ru/docs/http/ngx_http_core_module.html#server_name](http://nginx.org/ru/docs/http/ngx_http_core_module.html#server_name) и тут: [http://nginx.org/ru/docs/http/request_processing.html](http://nginx.org/ru/docs/http/request_processing.html).

**location**

Внутри каждого контекста `server` может быть несколько контекстов `location`, которые определяют правила для разных запрашиваемых URI. Сравнивается только URI-часть запроса, после её URL-раскодирования и разрешения относительных путей (`.` и `..`), **без GET-параметров.** Подробности в примере:

```nginx
server {
	listen 80 default_server;
	
	# Общие настройки для всего виртуального сервера, 
	# могут быть переопределены во вложенных контекстах

	# Корневая папка, в которой сервер будет искать файлы, 
	# может быть переопределена в location'ах
	root /var/www/mysite;

	# Конфигурации для конкретных URI:
	
	# 1. точное соответсвие (=)
	location = /exact-uri {
		return 200 "Requested URI equals '/exact-uri'";
	}

	# 2. соответствие по префиксу (/)
	location /prefix {
		return 200 "Requested URI starts with '/prefix'";
	}

	# 3. регистрозависимое соответствие регулярному выражению (~)
	location ~ ^regexp$ {
		return 200 "Requested URI matches '^regexp$'";
	}

	# 4. регистроНЕзависимое соответствие регулярному выражению (~*)
	location ~* ^regexp$ {
		return 200 "Requested URI matches '^regexp$' or '^REGEXP$'";
	}
}
```

Вкратце алгоритм поиска соответствия выглядит так:

1) если найдено точное совпадение с `location`'ом **точного соответствия,** применяется эта конфигурация и поиск останавливается. 

2) иначе находится **самый длинный** из **префиксных** `location`'ов совпадающий с URI и запоминается;

3) ищется совпадение среди `location`'ов заданных **регулярным выражением,** и применяется **первое** совпадение;

4) если же ни одно из регулярных выражений не совпало с URI, применяется **префиксный** `location`, запомненный ранее на шаге 2.

Если мы хотим остановить поиск на шаге 2 и не искать по регулярным выражениям, можно добавить к префиксному `location`'у знак `^~`, вот так:

```nginx
location ^~ /prefix {
	 # если URI начинается с /prefix, применится этот блок и поиск будет остановлен
}

location ~ /prefix/.* {
	# это не будет применено, поиск остановится в предыдущем блоке
}
```

`location`'ы можно вкладывать друг в друга, при этом вложенные должны указывать **абсолютные, а не относительные URI:**

```nginx
location /outer {
	# абсолютные URI - префикс /outer дублируется во всех вложенных определениях!
	location /outer/inner { 
		return 200 "Requested URI starts with '/outer/inner'";
	}
	location ~ ^/outer/regexp.* {
		return 200 "Requested URI matches '^/outer/regexp.*'";
	}
}
```

Подробнее можно почитать тут: [http://nginx.org/ru/docs/http/ngx_http_core_module.html#location](http://nginx.org/ru/docs/http/ngx_http_core_module.html#location)

**index**

Чаще всего используются директивы `index` и `try_files`.

Они могут находиться как на уровне `server`, так и на уровне `location`.

`index` указывает какой файл отдавать по умолчанию если в запрошенном URI не указан конкретный файл, то есть если URI выглядит как путь к папке, а не к файлу. **Указанные файлы проверяются в порядке слева направо и, как только находится существующий файл, происходит внутренний редирект на URI с добавлением имени этого фала.**  

Внутренний редирект означает **исполнение всего блока** `server` **заново**, с измененным URI. Это не то же самое что HTTP-редирект, т.к. URI видоизменяется только “внутри” nginx’a и клиент этого не видит. То есть:

```nginx
server {
	# ...

	# при запросе www.mysite.com/some/long/path, если существует файл 
	# /var/www/mysite_root/some/long/path/existing-file.html,
	# в ответ будет получено 'hello'

	root /var/www/mysite_root;
	
	index 
		not-existing-file.html # этого файла не существует в папке some/long/path
		existing-file.html # а этот существует в папке some/long/path
		another-existing-file.html; # и этот тоже, но это уже не важно)
	
	# это не применится, потому что несуществующие файлы пропускаются
	location /not-existing-file.html {
		return 200 ':(';
	}
	
	# это применится, потому что файл существует и внутренний редирект происходит
	location /existing-file.html {
		return 200 'hello'; 
	}
}
```

При этом внутренний редирект происходит только один раз:

```nginx
server {
	# ...

	location / {
		# если файл i1.html существует,
		# добавлеяем /i1.html к URI и ищем подходящий location заново
	  index i1.html; 
	}
	
	location /i1.html {
		# даже если файл i2.html существует, второй редирект не происходит (!),
		# т.к. у нас уже есть конкретный файл для ответа - i1.html.
		index i2.html;
	}
	
	location /i2.html {
		return 200 ':('; # это не выполнится
	}
}
```

Для PHP приложений с единой точкой входа (как в большинстве современных фреймворков) достаточно указать на уровне виртуального сервера:

```nginx
# направлять все запросы без указания конкретного файла на index.php
index index.php;

location ~ index.php$ {
	# проксируем запрос на PHP-FPM...
}
```

 Документация: [http://nginx.org/ru/docs/http/ngx_http_index_module.html](http://nginx.org/ru/docs/http/ngx_http_index_module.html).

**try_files**

Директива `try_files` позволяет последовательно проверить несколько файлов и отдать первый существующий, а также указать fallback на случай если ни один из файлов не найден. Чаще всего она используется в сочетании с переменной `$uri`, которая содержит текущий URI с учетом всех внутренних преобразований (см. `index` и `rewrite`). Пример:

```nginx
server {
	#...

	root /var/www;

	location ~ \.jpg$ {
		# при запросе www.mysite.com/path/to/image.jpg
		# будет проверено существование файла /var/www/path/to/image.jpg,
		# затем, если такого нет, будет проверен /var/www/path/to/images/image.jpg,
		# если и его нет, будет внутренний редирект на /fallback-uri.html
		try_files $uri /images/$uri /fallback-image.jpg;
	}

	location /fallback-image.jpg$ {
		# это выполнится если запрошенный файл не существует
		# ни в запрошенной директории ни в поддиректории images
		return 200 'default image';

		# кстати, если нужно чтоб этот location не был доступен извне, 
		# а только для внутренних редиректов, добавляем директиву internal
		internal;
	}

	# в качестве fallback можно указать код ошибки, чаще всего 404
	location ~ \.html$ {
		try_files $uri =404;
	}
	
	# ...или named location
	location / {
		try_files $uri @phpfpm;
	}

	location @phpfpm {
		# проксируем запрос на PHP-FPM...
	}
}
```

В `try_files` можно указать `$uri/` (с слешом в конце), тогда nginx будет искать запрашиваемую директорию. По умолчанию, nginx не отдает содержимое директорий (если только не указано `autoindex on`), поэтому в случае если директория существует, в дело вступает директива `index`, указывающая какой файл из этой директории отдавать (см. выше).

```nginx
server {
	#...

	root /var/www;
	index index.html;

	location / {
		try_files $uri $uri/ =404;
	}

	location ~ index.html$ {
		return 200 'serving file $uri/index.thml';
	}
}
```

`try_files` **не сохраняет GET-параметры текущего запроса при внутреннем редиректе**. Чтоб избежать их потери используется конструкция из переменных `$is_args` и `$args`:

```nginx
server {
	# ...
	
	try_files $uri $uri/ /index.php$is_args$args;

	# или так, если хотим добавить новые пареметры и сохранить текущие
	try_files $uri $uri/ /index.php?additional_param=1&$args;

	location = index.php {
		# тут $args будет содержать все изначальные и добавленные GET-параметры 
	}
}
```

Документация: [http://nginx.org/ru/docs/http/ngx_http_core_module.html#try_files](http://nginx.org/ru/docs/http/ngx_http_core_module.html#try_files).

Внутренние редиректы могут быть результатом и других директив, например `rewrite` или `error_page`. 

**rewrite**

Эта директива делает внутренний или внешний редирект - если новый URI начинается с `http://`, `https://`, или переменной `$scheme`, то произойдет внешний (HTTP) редирект, иначе - внутренний. **Если новый URI содержит GET-параметры, они будут добавлены к тем что уже есть в текущем запросе**. Подробнее в документации: [http://nginx.org/ru/docs/http/ngx_http_rewrite_module.html#rewrite](http://nginx.org/ru/docs/http/ngx_http_rewrite_module.html#rewrite). Пример:

```nginx
server {
	# ...
	
	location = /rewritten {
		rewrite .* /another-location?additional_param=1; # внутренний редирект
	}

	location = /another-location {
		rewrite .* 'http://mysite.com/another-location; # а теперь HTTP-редирект 
	}
}
```

**error_page**

Указывает URI, на который следует сделать внутренний редирект, если в процессе обработки запроса возникла ошибка. Детали в документации: [http://nginx.org/ru/docs/http/ngx_http_core_module.html#error_page](http://nginx.org/ru/docs/http/ngx_http_core_module.html#error_page). Пример:

```nginx
server {
	# ...
	
	# заменять все 404 коды на 200 и редиректить на другой location
	error_page 404 =200 /404_error.html;
	
	location = /404_error.html {
		# этот блок будет выполнен если в процессе возникнет ошибка с кодом 404 
	}
}
```

`error_page`, как и `try_files`, **не сохраняет GET-параметры текущего запроса при внутреннем редиректе.** Так что для их сохранения нужно явно добавлять переменную `$args` к новому URI (см. выше).

**Количество внутренних редиректов ограничено 10-ю за один HTTP-запрос**, во избежание бесконечных циклов. Если происходит больше 10-ти, nginx вернёт ошибку. 

Директивы `location` и `rewrite` могут содержать регулярные выражения з захватывающими группами,  значения из которых могут быть использованы в последующих директивах:

```nginx
server {
	# ...
	
	rewrite /user-(\d+)/product-(\d+) /user/$1/product/$2;

	location /new/user/(\d+)/product/(\d+) {
		return 200 'you requested user with id $1 and product with id $2';
	}
}
```

Также можно использовать встроенные переменные: [http://nginx.org/en/docs/http/ngx_http_core_module.html#variables](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables).

**alias**

Еще одна полезная директива - `alias`. Она позволяет заменить часть URI, например чтобы обратиться к файлам, чей путь в файловой системе не совпадает с URI.  Например:

```nginx
server {
	# ...

	root /var/www/mysiteroot;

	location /styles {
		# файлы из папки /var/www/mysiteroot/web/assets/css 
		# будут доступны по URI /styles/{имя файла}
		alias /web/assets/css;
	}
	location /scripts {
		# файлы из папки /var/www/mysiteroot/web/assets/js 
		# будут доступны по URI /scripts/{имя файла}
		alias /web/assets/js;
	}
}
```

`alias` удобно совмещать с `location`'ами, заданными регулярными выражениями:

```nginx
server {
	# ...

	root /var/www/mysiteroot;

	location /admin/(.*) {
		# "маппинг" базового URI на другую корневую директорию
		# (в данном случае на /var/www/mysiteroot/protected/web)
		alias /protected/web/$1;
	}
}
```

Документация: [http://nginx.org/ru/docs/http/ngx_http_core_module.html#alias](http://nginx.org/ru/docs/http/ngx_http_core_module.html#alias).

На этом всё. С помощью рассмотренных директив можно решить большинство простых задач, хотя это, наверное, одна сотая часть возможностей nginx’a. Хорошо что остальные возможности вряд ли пригодятся рядовому PHP-разработчику : )