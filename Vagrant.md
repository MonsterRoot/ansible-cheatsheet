```
$ mkdir playbooks
$ cd playbooks
$ vagrant init ubuntu/focal64
$ vagrant up
```
# Подключение к текущей машине по ssh:
`vagrant ssh`
# Вывести конфиг для подключения по ssh
`vagrant ssh-config`

# Остановка тестового сервера
`$ vagrant destroy -f`

# Переадресация портов и частные IP-адреса
Когда вы создаете новый Vagrantfile командой vagrant init, сетевая конфигурация по умолчанию позволяет получить доступ к виртуальной машине Vagrant только через порт SSH, который переадресуется с локального хоста. 
Для машины Vagrant, запускаемой первой, назначается порт 2222, а для каждой последующей будет назначаться другой порт. Как результат, единственный способ получить доступ к машине Vagrant с конфигурацией по умолчанию – это подключиться по SSH к localhost через порт 2222. Vagrant переадресует этот порт в порт 22 внутри вирту-
альной машины Vagrant.

# Переадресация локального порта 8000 в порт 80 машины Vagrant:
### Vagrantfile:
```Ruby
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	# Другие конфигурационные параметры не показаны
	config.vm.network :forwarded_port, host: 8000, guest: 80
	config.vm.network :forwarded_port, host: 8443, guest: 443
end
```
# Назначение IP-адреса машине Vagrant:
### Vagrantfile:
```Ruby
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	# Другие конфигурационные параметры не показаны
	# В этой конфигурации используется частная сеть Vagrant
	config.vm.network "private_network", ip: "192.168.33.10"
end
```
[Дополнительную информацию о различных параметрах настройки сети в Vagrant](https://developer.hashicorp.com/vagrant/docs/networking)

# Включение переадресации агента
### Vagrantfile
```Ruby
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	# Другие конфигурационные параметры не показаны
	# включение переадресации агента ssh
	config.ssh.forward_agent = true
end
```

# Подготовка Docker
Иногда бывает нужно сравнить контейнеры, выполняющиеся в разных вариантах Linux, и разные среды выполнения контейнеров. Vagrant может создать виртуальную машину с нуля, установить Docker или Podman и автоматически запустить образ контейнера за один раз
### Vagrantfile
```Ruby
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	config.vm.box = "ubuntu/focal64"
	config.vm.provision "docker" do |d|
		d.run "nginx"
	end
end
```
# Подготовка локальной версии Ansible
Для Vagrant есть внешние инструменты, называемые провайдерами (provisioners), которые он использует для настройки виртуальной машины после ее запуска. Помимо Ansible, Vagrant также может предоставлять сценарии командной оболочки, устанавливать Chef, Puppet, Salt и CFEngine.
Vagrantfile с настройкой ansible_local, согласно которой на виртуальную машину устанавливается система Ansible и используется в качестве провайдера, в частности, с помощью сценария Ansible с именем playbook.yml.
### Vagrantfile:

```Ruby
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	config.vm.box = "ubuntu/xenial64"
	config.vm.provision "ansible_local" do |ansible|
		ansible.compatibility_mode = "2.0"
		ansible.galaxy_role_file = "roles/requirements.yml"
		ansible.galaxy_roles_path = "roles"
		ansible.playbook = "playbook.yml"
		ansible.verbose = "vv"
	end
end
```
Благодаря этому нет необходимости вручную устанавливать Ansible на свой компьютер. Если в вашем файле Vagrantfile имеется настройка **config.vm.provision "ansible_local"**, то система будет установлена и запущена в виртуальной машине. При использовании настройки **config.vm.provision "ansible"** в Vagrantfile провайдер будет использовать версию Ansible, уже
установленную на вашем компьютере

# Когда запускаются сценарии провайдеров
Когда в первый раз запускается команда vagrant up, Vagrant выполнит сценарий, осуществляющий подготовку и наполнение виртуальной машины, и зафиксирует факт своего запуска. После остановки и повторного запуска виртуальной машины Vagrant «вспомнит», что сценарий провайдера уже выполнялся, и не будет повторно запускать
его.
### При желании можно принудительно запустить сценарий наполнения на запущенной виртуальной машине:
`$ vagrant provision`
### Можно также перезагрузить виртуальную машину и запустить сценарий наполнения после перезагрузки:
`$ vagrant reload --provision`
### Аналогично можно запустить остановленную виртуальную машину с принудительным запуском сценария наполнения:
`$ vagrant up --provision`
Мы часто используем эти команды для запуска сценариев Ansible из
командной строки с некоторым тегом или ограничением.

# Плагины Vagrant
### Пример:
```Ruby
config.vagrant.plugins = ["vagrant-hostmanager", "vagrant-vbguest"]
```
## vagrant-hostmanager:
Плагин vagrant-hostmanager помогает обращаться к нескольким виртуальным машинам по именам хостов. Он изменит имена хостов и добавит гостевые системы в /etc/hosts, а иногда и сам хост, в зависимости от конфигурации:
```Ruby
# управление файлом /etc/hosts
config.hostmanager.enabled = true
config.hostmanager.include_offline = true
config.hostmanager.manage_guest = true
config.hostmanager.manage_host = true
```
### vagrant-vbguest:
Плагин vagrant-vbguest работает в VirtualBox и может автоматически устанавливать или обновлять дополнения для гостевой системы (Guest Additions) в гостевых виртуальных машинах. Бас обычно отключает эти функции в macOS, потому что обмен файлами между гостевыми системами и macOS недостаточно быстр и не всегда надежен. Более того, обмен файлами между хостом и гостевой системой не имитирует порядок развертывания программного обеспечения в окружениях разработки, тестирования, обкатки и промышленной эксплуатации. Но он отлично подходит для изучения Ansible в Windows:
```Ruby
# обновление дополнений гостевых систем
if Vagrant.has_plugin?("vagrant-vbguest")
	config.vbguest.auto_update = true
end
```
# Настройка VirtualBox
### Определить свойства виртуальной машины и ее внешний вид в VirtualBox. Например:

```Ruby
host_config.vm.provider "virtualbox" do |vb|
	vb.name = "web"
	virtualbox.customize ["modifyvm", :id,
			"--audio", "none",
			"--cpus", 2,
			"--memory", 2048,
			"--graphicscontroller", "VMSVGA",
			"--vram", "64"
		]
end
```
# Vagrantfile – это Ruby
Файлы Vagrantfile выполняются интерпретатором Ruby. Это знание может вам пригодиться хотя бы для настройки подсветки синтаксиса в текстовом редакторе. В Vagrantfile можно объявлять переменные, использовать управляющие структуры и циклы и т. д. В примерах ис-
ходного кода, прилагаемых к этой книге, есть более сложный пример файла [Vagrantfile](https://github.com/ansiblebook/ansiblebook/blob/master/chapter02/playbooks/Vagrantfile), который мы используем для работы с 15 различными вариантами Linux
### Для настройки гостевых систем мы используем файл JSON с такими элементами, как:
```json
[
	{
		"name": "centos8",
		"cpus": 1,
		"distro": "centos",
		"family": "redhat",
		"gui": false,
		"box": "centos/stream8",
		"ip_addr": "192.168.56.6",
		"memory": "1024",
		"no_share": false,
		"app_port": "80",
		"forwarded_port": "8006"
	},
	{
		"name": "focal",
		"cpus": 1,
		"distro": "ubuntu",
		"family": "debian",
		"gui": false,
		"box": "ubuntu/focal64",
		"ip_addr": "192.168.56.8",
		"memory": "1024",
		"no_share": false,
		"app_port": "80",
		"forwarded_port": "8008"
	}
]
```
### И в файле Vagrantfile у нас есть пара конструкций для создания одной гостевой системы по имени при входе, например:
`$ vagrant up focal`
### Вот сам файл Vagrantfile:
```Ruby
Vagrant.require_version ">= 2.0.0"
# Подключить модуль JSON
require 'json'
# Прочитать файл JSON с настройками
f = JSON.parse(File.read(File.join(File.dirname(__FILE__), 'config.json')))
# Локальная переменная PATH_SRC для монтирования
$PathSrc = ENV['PATH_SRC'] || "."
Vagrant.configure(2) do |config|
	config.vagrant.plugins = ["vagrant-hostmanager", "vagrant-vbguest"]
	# проверить обновления базового образа
	config.vm.box_check_update = true
	# небольшая задержка
	config.vm.boot_timeout = 1200
	# запретить обновление дополнений гостевой системы
	if Vagrant.has_plugin?("vagrant-vbguest")
		config.vbguest.auto_update = false
	end
	# включить переадресацию агента ssh
	config.ssh.forward_agent = true
	# использовать стандартный для vagrant ключ ssh
	config.ssh.insert_key = false
	# управление файлом /etc/hosts
	config.hostmanager.enabled = true
	config.hostmanager.include_offline = true
	config.hostmanager.manage_guest = true
	config.hostmanager.manage_host = true
	# Цикл по элементам в файле JSON
	f.each do |g|
		config.vm.define g['name'] do |s|
			s.vm.box = g['box']
			s.vm.hostname = g['name']
			s.vm.network 'private_network', ip: g['ip_addr']
			s.vm.network :forwarded_port,
			host: g['forwarded_port'],
			guest: g['app_port']
			# установить значение no_share равным false,
			# чтобы разрешить совместное использование файлов
			s.vm.synced_folder ".", "/vagrant", disabled: g['no_share']
			s.vm.provider :virtualbox do |virtualbox|
				virtualbox.customize ["modifyvm", :id,
					"--audio", "none",
					"--cpus", g['cpus'],
					"--memory", g['memory'],
					"--graphicscontroller", "VMSVGA",
					"--vram", "64"
				]
				virtualbox.gui = g['gui']
				virtualbox.name = g['name']
			end
		end
	end
	config.vm.provision "ansible_local" do |ansible|
		ansible.compatibility_mode = "2.0"
		ansible.galaxy_role_file = "roles/requirements.yml"
		ansible.galaxy_roles_path = "roles"
		ansible.playbook = "playbook.yml"
		ansible.verbose = "vv"
	end
end
```
*Свойства всех виртуальных машин настраиваются в файле config.json*
# Настройка промышленного окружения
Для подключения к машинам **Linux/macOS/BSD Ansible использует SSH**, а для подключения к машинам **Windows – WinRM**. Сетевыми устройствами можно управлять через HTTPS или SSH. Никакого дополнительного программного обеспечения на целевых хостах устанавливать не требуется (**при условии, что на машинах Linux/macOS/BSD установлен**
**Python, а на машинах Windows – PowerShell**).
Традиционные системные администраторы проявляют здоровую осторожность при внедрении инструментов, требующих системных привилегий, потому что обычно только сами системные администраторы имеют такие разрешения. В Unix принято делегировать разработчикам доступ только к определенным командам с помощью sudo с тщательно выверенными файлами в /etc/sudoers.d/.
Этот подход не работает ни с Ansible, ни с такой ограничительной оболочкой, как rbash. Ansible создает временные каталоги со случайными именами для различных сценариев на Python, тогда как sudo нужны точные команды. Альтернативой является смещение фокуса на содержание изменений в системе управления версиями в окружении обкатки и наличие файла с настройками sudo для группы ansible
### например:
`%ansible ALL=(ALL) ALL`

