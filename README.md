**Настраиваем центральный сервер для сбора логов**

Все дальнейшие действия были проверены при использовании
```
marozov@iMac-marozov ~ % vagrant -v
Vagrant 2.2.19
```
```
marozov@imac logs % vboxmanage -v
6.1.32r149290
```
CentOS Linux release 7.9 из Vagrant Cloud
```
[root@selinux ~]$ cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

в вагранте поднимаем 2 машины web и log
на web поднимаем nginx
на log настраиваем центральный лог сервер на любой системе на выбор
journald;
rsyslog;
elk.
настраиваем аудит, следящий за изменением конфигов нжинкса
Все критичные логи с web должны собираться и локально и удаленно. 
Все логи с nginx должны уходить на удаленный сервер (локально только критичные). 
Логи аудита должны также уходить на удаленную систему.

**Решение**

Использовал journald, systemd-journal-remote, systemd-journal-upload Для передачи в journald логов nginx, добавил в конфиг nginx вывод ошибок и access в
```
error_log  stderr;
access_log syslog:server=unix:/dev/log;
auditd по умолчанию не выводит в journald сделал systemd юнит который выгружает логи в journald
```

```
ExecStart=/bin/sh -c 'tail -f /var/log/audit/audit.log  | systemd-cat -t nginx_conf_audit'
```
так же юнит для сохрания лога в файл, который отправляется на центральный сервер

Посмотреть и проверить можно следующим образом.
```
vagrant ssh log
sudo journalctl -D /var/log/journal/remote -f
```
Сможем мониторить журанл в реальном времени. На хост проброщен порт с web 8080 , в браузере можно вбить localhost:8080 и увидеть дефолтную веб страницу, в логах отобразиться access. Аудит и ошибки nginx можно проверить так. Изменить на web конфиг nginx. вбить в адрес в браузере не существующую страницу, например

```
localhost:8080/otus.php
```
