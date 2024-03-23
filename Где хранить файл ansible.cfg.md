
Ansible будет искать файл ansible.cfg в следующих местопо-
ложениях в указанном порядке:
	• файл, указанный в переменной окружения ANSIBLE_CONFIG;
	• ./ansible.cfg (ansible.cfg в текущем каталоге);
	• ~/.ansible.cfg (.ansible.cfg в вашем домашнем каталоге);
	• /etc/ansible/ansible.cfg (Linux) или /usr/local/etc/ansible/ansible.cfg (BSD).
Пример 2.2. ansible.cfg
```
[defaults]
inventory = inventory/vagrant.ini
host_key_checking = False
stdout_callback = yaml
callback_enabled = timer
```
