#  **Файловые хранилища - NFS, SMB, FTP**

## **Homework**

Vagrant stand NFS или SAMBA
NFS или SAMBA на выбор:

- vagrant up должен поднимать 2 виртуалки: сервер и клиент
- на сервер должна быть расшарена директория
- на клиента она должна автоматически монтироваться при старте (fstab или autofs)
- в шаре должна быть папка upload с правами на запись
- требования для NFS: NFSv3 по UDP, включенный firewall

* Настроить аутентификацию через KERBEROS 

Для работы с KERBEROS настроим DNC и NTP:
- DNC  домен otus.test;
- NTP chronyd, сервер синхронизурется с сервером из пула 0.rhel.pool.ntp.org, а клиент - с нашим сервером.


**Развернут MIT Kerberos5 сервер, аутентификация происходит в домене OTUS.TEST.**

Работа протокола Kerberos:
- клиент авторизируется;
- запрос AS_REQ - Hash (логин+пароль=хэш) + TimeStamp (синхронизированное время) + ID (id клиента);
- получение Ticket-Granting Ticket;
- получение  сессионных ключей TGS для доступа к сервисам.

Также устанавливаем nfs сервер и открываем порты для сервисов:

- SSH (22/tcp)
- NTP (123/udp)
- DNS (53/tcp, 53/udp)
- Kerberos (88/tcp, 88/udp, 794/tcp)
- NFS (111/tcp, 111/upd - rpc-bind, 2049/tcp, 2049/udp - nfs сервер)

Для включенного firewall прописываем:

firewall-cmd --permanent --add-service=rpc-bind

firewall-cmd --permanent --add-service=mountd

firewall-cmd --permanent --add-port=2049/tcp

firewall-cmd --permanent --add-port=2049/udp

firewall-cmd --reload

## **Проверка**

Сначала заходим на client. Делаем touch /mnt/upload/natali и видим отказ доступа. Напоминаю, что сначала стоит делать kinint:

[root@admin OtusHW26]# vagrant ssh client

DEPRECATION: The 'sudo' option for the Ansible provisioner is deprecated.

Please use the 'become' option instead.

The 'sudo' option will be removed in a future release of Vagrant.

Last login: Sun Apr 28 09:25:49 2019 from 10.0.2.2

[vagrant@client ~]$ touch /mnt/upload

touch: cannot touch ‘/mnt/upload’: Permission denied

[vagrant@client ~]$ kinit

Password for vagrant@OTUS.TEST: 

[vagrant@client ~]$ touch /mnt/upload/natali

[vagrant@client ~]$ ll /mnt/upload/

total 0

-rw-rw-r--. 1 vagrant vagrant 0 Apr 28 09:28 natali

По логам на сервере ns видим успешность выполненного задания:

[vagrant@ns ~]$ sudo tail -f /var/log/krb5kdc.log

Apr 28 09:27:26 ns.otus.test krb5kdc[7985](info): AS_REQ (6 etypes {16 23 20 18 19 17}) 192.168.50.100: NEEDED_PREAUTH: nfs/client.otus.test@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST, Additional pre-authentication required

Apr 28 09:27:26 ns.otus.test krb5kdc[7985](info): AS_REQ (6 etypes {16 23 20 18 19 17}) 192.168.50.100: ISSUE: authtime 1556432846, etypes {rep=16 tkt=16 ses=16}, nfs/client.otus.test@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST

Apr 28 09:27:26 ns.otus.test krb5kdc[7985](info): AS_REQ (6 etypes {16 23 20 18 19 17}) 192.168.50.100: NEEDED_PREAUTH: host/client.otus.test@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST, Additional pre-authentication required

Apr 28 09:27:26 ns.otus.test krb5kdc[7985](info): AS_REQ (6 etypes {16 23 20 18 19 17}) 192.168.50.100: ISSUE: authtime 1556432846, etypes {rep=16 tkt=16 ses=16}, host/client.otus.test@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST

Apr 28 09:27:26 ns.otus.test krb5kdc[7985](info): TGS_REQ (6 etypes {20 18 19 17 16 23}) 192.168.50.100: ISSUE: authtime 1556432846, etypes {rep=16 tkt=16 ses=16}, host/client.otus.test@OTUS.TEST for nfs/ns.otus.test@OTUS.TEST

Apr 28 09:27:35 ns.otus.test krb5kdc[7985](info): AS_REQ (6 etypes {20 18 19 17 16 23}) 192.168.50.100: NEEDED_PREAUTH: vagrant@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST, Additional pre-authentication required

Apr 28 09:27:48 ns.otus.test krb5kdc[7985](info): AS_REQ (6 etypes {20 18 19 17 16 23}) 192.168.50.100: ISSUE: authtime 1556432868, etypes {rep=16 tkt=16 ses=16}, vagrant@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST

Apr 28 09:27:56 ns.otus.test krb5kdc[7985](info): TGS_REQ (6 etypes {20 18 19 17 16 23}) 192.168.50.100: ISSUE: authtime 1556432868, etypes {rep=16 tkt=16 ses=16}, vagrant@OTUS.TEST for nfs/ns.otus.test@OTUS.TEST

Apr 28 09:32:48 ns.otus.test krb5kdc[7985](info): AS_REQ (1 etypes {16}) 127.0.0.1: NEEDED_PREAUTH: vagrant@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST, Additional pre-authentication required

Apr 28 09:32:51 ns.otus.test krb5kdc[7985](info): AS_REQ (1 etypes {16}) 127.0.0.1: ISSUE: authtime 1556433171, etypes {rep=16 tkt=16 ses=16}, vagrant@OTUS.TEST for krbtgt/OTUS.TEST@OTUS.TEST

