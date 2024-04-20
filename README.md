###  OTUS Linux Professional Lesson #8 | Subject: Практические навыки работы с ZFS
#### Домашнее задание

####  Цель:
Отработать навыки работы с созданием томов export/import и установкой параметров;

определить алгоритм с наилучшим сжатием;
определить настройки pool’a;
найти сообщение от преподавателей.
составить список команд, которыми получен результат с их выводами.

Описание/Пошаговая инструкция выполнения домашнего задания:
Для выполнения домашнего задания используйте методичку

### Практические навыки работы с ZFS

####  Что нужно сделать?

1. Определить алгоритм с наилучшим сжатием:
2. Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
создать 4 файловых системы на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов.
Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
   
- размер хранилища;
    
- тип pool;
    
- значение recordsize;
   
- какое сжатие используется;
   
- какая контрольная сумма используется.
3. Работа со снапшотами:
скопировать файл из удаленной директории;
восстановить файл локально. zfs receive;
найти зашифрованное сообщение в файле secret_message.

#### Критерии оценки:
Статус «Принято» ставится при выполнении следующих условий:

Сcылка на репозиторий GitHub.
Vagrantfile с Bash-скриптом, который будет конфигурировать сервер.
Документация по каждому заданию:
Создайте файл README.md и снабдите его следующей информацией:
название выполняемого задания;
текст задания;
описание команд и их вывод;
особенности проектирования и реализации решения;
заметки, если считаете, что имеет смысл их зафиксировать в репозитории.


#### Выполнение


Cоздаем виртуальную машину
Создаём каталог, в котором будут храниться настройки виртуальной машины и дополнительные диски. В каталоге создаём файл с именем Vagrantfile, добавляем в него следующее содержимое: 
````
# -*- mode: ruby -*-
# vim: set ft=ruby :
disk_controller = 'IDE' # MacOS. This setting is OS dependent. Details https://github.com/hashicorp/vagrant/issues/8105

MACHINES = {
  :zfs => {
        :box_name => "centos/7",
        :box_version => "2004.01",
    :disks => {
        :sata1 => {
            :dfile => './sata1.vdi',
            :size => 512,
            :port => 1
        },
        :sata2 => {
            :dfile => './sata2.vdi',
            :size => 512, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => './sata3.vdi',
            :size => 512,
            :port => 3
        },
        :sata4 => {
            :dfile => './sata4.vdi',
            :size => 512, 
            :port => 4
        },
        :sata5 => {
            :dfile => './sata5.vdi',
            :size => 512,
            :port => 5
        },
        :sata6 => {
            :dfile => './sata6.vdi',
            :size => 512,
            :port => 6
        },
        :sata7 => {
            :dfile => './sata7.vdi',
            :size => 512, 
            :port => 7
        },
        :sata8 => {
            :dfile => './sata8.vdi',
            :size => 512, 
            :port => 8
        },
    }
        
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.box_version = boxconfig[:box_version]

        box.vm.host_name = "zfs"

        box.vm.provider :virtualbox do |vb|
              vb.customize ["modifyvm", :id, "--memory", "1024"]
              needsController = false
        boxconfig[:disks].each do |dname, dconf|
              unless File.exist?(dconf[:dfile])
              vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
         needsController =  true
         end
        end
            if needsController == true
                vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                boxconfig[:disks].each do |dname, dconf|
                vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                end
             end
          end
        box.vm.provision "shell", inline: <<-SHELL
          #install zfs repo
          yum install -y http://download.zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm
          #import gpg key 
          rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
          #install DKMS style packages for correct work ZFS
          yum install -y epel-release kernel-devel zfs
          #change ZFS repo
          yum-config-manager --disable zfs
          yum-config-manager --enable zfs-kmod
          yum install -y zfs
          #Add kernel module zfs
          modprobe zfs
          #install wget
          yum install -y wget
      SHELL

    end
  end
end
````
Результатом выполнения команды vagrant up станет созданная виртуальная машина, с 8 дисками и уже установленным и готовым к работе ZFS. 
Заходим на сервер: vagrant ssh
Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя: sudo -i

##### ЗАДАНИЕ 1. Определение алгоритма с наилучшим сжатием
1. Смотрим список всех дисков, которые есть в виртуальной машине:
```   
lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk
sdc      8:32   0  512M  0 disk
sdd      8:48   0  512M  0 disk
sde      8:64   0  512M  0 disk
sdf      8:80   0  512M  0 disk
sdg      8:96   0  512M  0 disk
sdh      8:112  0  512M  0 disk
sdi      8:128  0  512M  0 disk
```

Создаём четыре пула из двух дисков в режиме RAID 1:

```
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi
```

Смотрим информацию о пулах: zpool list

```
zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
```

Добавим разные алгоритмы сжатия в каждую файловую систему:
- Алгоритм lzjb: zfs set compression=lzjb otus1
- Алгоритм lz4:  zfs set compression=lz4 otus2
- Алгоритм gzip: zfs set compression=gzip-9 otus3
- Алгоритм zle:  zfs set compression=zle otus4

Проверим, что все файловые системы имеют разные методы сжатия:
```
zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
```

Скачаем один и тот же текстовый файл во все пулы:
```
 for i in {1..4}; do wget -P /otus$i/dataset$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```

Проверим, что файл был скачан во все пулы:

```
ls -lh /otus*/dataset*/
/otus1/dataset1/:
total 22M
-rw-r--r--. 1 root root 40M Apr  2 07:54 pg2600.converter.log

/otus2/dataset2/:
total 18M
-rw-r--r--. 1 root root 40M Apr  2 07:54 pg2600.converter.log

/otus3/dataset3/:
total 11M
-rw-r--r--. 1 root root 40M Apr  2 07:54 pg2600.converter.log

/otus4/dataset4/:
total 40M
-rw-r--r--. 1 root root 40M Apr  2 07:54 pg2600.converter.log
```
Уже на этом этапе видно, что самый оптимальный метод сжатия у нас используется в датасете otus3/dataset3.

Проверим, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов:
```
 zfs list
 NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.9M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.3M   313M     39.2M  /otus4
```

Алгоритм gzip-9 самый эффективный по сжатию.

##### ЗАДАНИЕ 2. Определение настроек пула

Скачиваем архив в домашний каталог: 
```
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download' 
```
Разархивируем его:
```
tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

Проверим, возможно ли импортировать данный каталог в пул:
```
zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                                 ONLINE
          mirror-0                           ONLINE
            /home/vagrant/zpoolexport/filea  ONLINE
            /home/vagrant/zpoolexport/fileb  ONLINE
```
Данный вывод показывает нам имя пула, тип raid и его состав.
Команда `zpool import` ищет устройства для импортирования. Ключ -d говорит искать в указанном каталоге. Если не указать этого, будет искать в /dev по умолчанию. 
> [!NOTE]
> Как вообще делается экспорт пула, например, если нужно перенести пул на другой сервер: выполняем команду `zpool export tank`, zfs скидывает незаписанные данные с кеша на диск, и отмонтирует пул. Дальше
> мы берем эти диски и перетыкаем в другой сервер. На новом сервере запускаем `zpool import`. ZFS начинает сканировать блочные устройства в каталоге /dev и выдает нам найденные для импортирования пулы на наших
> только что вставленных дисках. Далее выполняем импортирование пула в нашу систему как показано ниже.

Сделаем импорт данного пула к нам в ОС:

до выполнения
```
zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.9M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.3M   313M     39.2M  /otus4
```

Импорт
```
zpool import -d zpoolexport/ otus
````
После выполнения импорта
```
zfs list
NAME             USED  AVAIL     REFER  MOUNTPOINT
otus            2.04M   350M       24K  /otus
otus/hometask2  1.88M   350M     1.88M  /otus/hometask2
otus1           21.9M   330M     21.6M  /otus1
otus2           17.7M   334M     17.6M  /otus2
otus3           10.8M   341M     10.7M  /otus3
otus4           39.3M   313M     39.2M  /otus4
```

Команда zpool status выдает нам информацию о составе импортированного пула.
```
zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                                 STATE     READ WRITE CKSUM
        otus                                 ONLINE       0     0     0
          mirror-0                           ONLINE       0     0     0
            /home/vagrant/zpoolexport/filea  ONLINE       0     0     0
            /home/vagrant/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```

Запрос сразу всех параметром файловой системы: zfs get all otus

```
zfs get all otus
NAME  PROPERTY              VALUE                  SOURCE
otus  type                  filesystem             -
otus  creation              Fri May 15  4:00 2020  -
otus  used                  2.04M                  -
otus  available             350M                   -
otus  referenced            24K                    -
otus  compressratio         1.00x                  -
otus  mounted               yes                    -
otus  quota                 none                   default
otus  reservation           none                   default
otus  recordsize            128K                   local
otus  mountpoint            /otus                  default
otus  sharenfs              off                    default
otus  checksum              sha256                 local
otus  compression           zle                    local
otus  atime                 on                     default
otus  devices               on                     default
otus  exec                  on                     default
otus  setuid                on                     default
otus  readonly              off                    default
otus  zoned                 off                    default
otus  snapdir               hidden                 default
otus  aclinherit            restricted             default
otus  createtxg             1                      -
otus  canmount              on                     default
otus  xattr                 on                     default
otus  copies                1                      default
otus  version               5                      -
otus  utf8only              off                    -
otus  normalization         none                   -
otus  casesensitivity       sensitive              -
otus  vscan                 off                    default
otus  nbmand                off                    default
otus  sharesmb              off                    default
otus  refquota              none                   default
otus  refreservation        none                   default
otus  guid                  14592242904030363272   -
otus  primarycache          all                    default
otus  secondarycache        all                    default
otus  usedbysnapshots       0B                     -
otus  usedbydataset         24K                    -
otus  usedbychildren        2.01M                  -
otus  usedbyrefreservation  0B                     -
otus  logbias               latency                default
otus  objsetid              54                     -
otus  dedup                 off                    default
otus  mlslabel              none                   default
otus  sync                  standard               default
otus  dnodesize             legacy                 default
otus  refcompressratio      1.00x                  -
otus  written               24K                    -
otus  logicalused           1020K                  -
otus  logicalreferenced     12K                    -
otus  volmode               default                default
otus  filesystem_limit      none                   default
otus  snapshot_limit        none                   default
otus  filesystem_count      none                   default
otus  snapshot_count        none                   default
otus  snapdev               hidden                 default
otus  acltype               off                    default
otus  context               none                   default
otus  fscontext             none                   default
otus  defcontext            none                   default
otus  rootcontext           none                   default
otus  relatime              off                    default
otus  redundant_metadata    all                    default
otus  overlay               off                    default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default
```

C помощью команды get можно уточнить конкретный параметр, например:
Размер: zfs get available otus
Тип сжатия (или параметр отключения): zfs get compression otus
Тип контрольной суммы: zfs get checksum otus


#### ЗАДАНИЕ 3. Работа со снапшотами, поиск сообщения от преподавателя


Скачаем файл, указанный в задании:
```
wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
```

Восстановим файловую систему из снапшота:

```
zfs receive otus/test@today < otus_task2.file
```

>[!NOTE]
> `zfs receive` создает снапшот, содержимое которого помещается в поток, направляемый на стандартный ввод. Если получен полный поток, то также создается файловая система. В нашем случае мы получаем поток с файла
> otus_task2.file. Этот файл был создан командой `zfs send`, которая создает потоковое представление снапшота и перенаправялет его на стандартный вывод. По умолчанию, zfs send генерирует полный поток.
> В "otus/test@today" today это имя снашота

После выполения этой команды будет создан снапшот и новый датасет. Команды для просмотра списка снапшотов:
```
zfs list -t snapshot
NAME              USED  AVAIL     REFER  MOUNTPOINT
otus/test@today     0B      -     2.83M  -
```
```
zpool get listsnapshots otus
NAME  PROPERTY       VALUE      SOURCE
otus  listsnapshots  off        default
```

Далее, ищем в каталоге /otus/test файл с именем “secret_message”:
```
find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
```

Смотрим содержимое найденного файла:
```
cat /otus/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```
Видим ссылку https://otus.ru/lessons/linux-hl/

### Все задания выполнены
