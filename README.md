# Инструкции

* [Как начать Git](git_quick_start.md)
* [Как начать Vagrant](vagrant_quick_start.md)

## otus-linux

Используйте этот [Vagrantfile](Vagrantfile) - для тестового стенда.

## Задание

- Выполнил основное задание на 10 рейде с 4мя дисками
  
`mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e,}`
  
## Задание со *

- Поломаем старую машинку `vagrant destroy otuslinux`

- Поднимать машинку с готовым рейдом будем с помощью провизнера Ansible
  
Создадим `ansible.cfg` для загрузки и подклбчения роли

```conf
[defaults]
inventory = ./inventory.yml
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False
roles_path = ./roles
```

Загрузим роль из galaxy создав файл зависимостей `requirements.yml` и подключив его командой `ansible-galaxy install -r requirements.yml`

```yml
---
- name: mdadm
  src: mrlesmithjr.mdadm
```

Внесем изменения в `.gitignore`, добавив `mdadm` чтоб не подгружать данную роль в репу

Добавим Ansible в `Vagrantfile`:

```json
box.vm.provision "ansible" do |ansible|
    ansible.playbook = "raid.yml"
    ansible.groups = {
    "vb" => ["otuslinux"]  
    }
    ansible.extra_vars = {
      mdadm_arrays: [{
        name: "md0",
        devices: ["/dev/sdb", "/dev/sdc", "/dev/sdd", "/dev/sde"],
        level: "10",
        filesystem: "ext4",
        mountpoint: "/mnt/md0",
        state: "present"
        }]
    }
end
```

Запускаем `vagrant up`

И получаем примонтированный рейд:

```bash
[vagrant@otuslinux ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        40G  3.9G   37G  10% /
devtmpfs        488M     0  488M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.7M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/md0        473M  2.3M  442M   1% /mnt/md0
tmpfs           100M     0  100M   0% /run/user/1000
```
