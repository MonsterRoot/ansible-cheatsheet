# Запустить playbook
`ansible-playbook webservers.yml`
# Тестирование соединения с хостами
`ansible testserver -i inventory/vagrant.ini -m ping`
# Выполнение команды на хостах
`$ ansible testserver -m command -a uptime`
*Модуль command используется настолько часто, что сделан модулем по-умолчанию, т. е. его имя можно опустить в команде:*
`$ ansible testserver -a uptime`
`$ ansible testserver -a "tail /var/log/dmesg"`

Чтобы выполнить команду с привилегиями **root**, нужно передать **параметр –b или --become**. В этом случае Ansible выполнит команду от лица (become) пользователя root. В Unix/Linux для этого обычно используется такой инструмент, как sudo, который необходимо настроить. В примерах Vagrant в этой книге это было сделано автоматически.

## для доступа к /var/log/syslog требуются привилегии root:
`$ ansible testserver -b -a "tail /var/log/syslog"`
### Результат будет выглядеть примерно так:
```shell
testserver | CHANGED | rc=0 >>
Apr 23 10:39:41 ubuntu-focal multipathd[471]: sdb: failed to get udev uid: Invalid argument
Apr 23 10:39:41 ubuntu-focal multipathd[471]: sdb: failed to get sysfs uid: No data available
Apr 23 10:39:41 ubuntu-focal multipathd[471]: sdb: failed to get sgio uid: No data available
Apr 23 10:39:42 ubuntu-focal multipathd[471]: sda: add missing path Apr 23 10:39:42 ubuntu-focal multipathd[471]: sda: failed to get udev uid: Invalid argument
Apr 23 10:39:42 ubuntu-focal multipathd[471]: sda: failed to get sysfs uid: No data available
Apr 23 10:39:42 ubuntu-focal multipathd[471]: sda: failed to get sgio uid: No data available
Apr 23 10:39:43 ubuntu-focal systemd[1]: session-95.scope: Succeeded.
Apr 23 10:39:44 ubuntu-focal systemd[1]: Started Session 97 of user vagrant.
Apr 23 10:39:44 ubuntu-focal python3[187384]: ansible-command Invoked with_raw_params=tail /var/log/syslog warn=True _uses_shell=False stdin_add_newline=True strip_empty_ends=True argv=None chdir=None executable=None creates=None removes=None stdin=None
```
# установить пакеты:
## NGINX в Ubuntu:
`$ ansible testserver -b -m package -a name=nginx`
## эквивалент команды apt-get update перед установкой пакета, замените аргумент name=nginx на name=nginx update_cache=yes. Чтоб перезапустить Nginx state=restarted
`$ ansible testserver -b -m service -a "name=nginx update_cache=yes state=restarted"`



