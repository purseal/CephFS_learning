#Пример разворачивания Ceph

В данном отчете использовались 3 машины с CentOS 6.4. Ниже представлена их конфигурация.

![Конфигурация](./img/ceph_install_config.jpg)

Настраиваем **ceph-node1** для регистрации на остальных узлах через SSH без пароля.

Выполните следующие команды на ceph-node1:

При настройке SSH оставьте фразу-пароль пустой и продолжайте с настройками по умолчанию:

```
# ssh-keygen
```

Скопируйте ID ключи SSH на **ceph-node2** и **ceph-node3** предоставляя их пароли root. После этого вы должны быть в состоянии войти на эти узлы без пароля:

````
# ssh-copy-id ceph-node2
````

Устанавливаем и настраиваем **EPEL** на всех узлах Ceph:

```
# rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

Убедитесь, что параметр `baserul` разрешен в файле `/etc/yum.repos.d/epel.repo`. Также проверьте, что параметр `mirrorlist` должен быть запрещен (присутствовать в виде комментария) в этом файле.

Устанавливаем `ceph-deploy` на машине **ceph-node1**, выполнив следующую команду на узле ceph-node1:

```
# yum install ceph-deploy
```

Далее мы создаем кластер Ceph с применением `ceph-deploy`, путем выполнения следующей команды на **ceph-node1**:

```
# ceph-deploy new ceph-node1
# mkdir /etc/ceph
# cd /etc/ceph
```

Для установки двоичных кодов программного обеспечения Ceph на все машины выполним следующую команду на **ceph-node1**:

```
# ceph-deploy install --release emperor ceph-node1 ceph-node2 cephnode3
```

Чтобы проверить версию Ceph и состояние Ceph на всех узлах, выполните команду:

```
# ceph –v
```

Создаем наш первый монитор на **ceph-node1**:

```
# ceph-deploy mon create-initial
```

На данном этапе кластер не работоспособен. Для проверки состояния монитора выполните следующую команду.

```
# ceph status
```

Далее создаем OSD и добавляем его в кластер Ceph.

Выводим перечень дисков виртуальной машины:

```
# ceph-deploy disk list ceph-node1
```

В данном отчете мы используем диски SDB, SDC и SDD.

Подкоманда `disk zap` уничтожит существующую таблицу разделов и содержание диска.

```
# ceph-deploy disk zap ceph-node1:sdb ceph-node1:sdc cephnode1:sdd
# ceph-deploy mon create-initial
```

Подкоманда `osd create` сначала подготовит диск, то есть сотрет диск с файловой системой, которой по умолчанию является xfs. Затем она активирует первый раздел диска в качестве раздела данных и второй раздел как журнал:

```
# ceph-deploy osd create ceph-node1:sdb ceph-node1:sdc cephnode1:sdd
```

Проверьте состояние кластера для новых элементов OSD:

```
# ceph status
```

Настраиваем MDS:

```
# ceph-deploy mds create ceph-node2

```
#####Добавление монитора

Теперь у нас есть кластер с одним узлом. Чтобы сделать его распределенным, надежным кластером хранения, мы должны выполнить его масштабирование. Для расширения кластера мы должны добавить больше узлов мониторов и OSD. Теперь будем настраивать машины **ceph-node2** и **ceph-node3** как узлы мониторов и OSD.

Правила межсетевого экрана не должны блокировать связь между узлами мониторов Ceph. Если такие правила имеются, вы должны исправить эти правила межсетевого экрана чтобы мониторы могли формировать кворум. Поскольку в нашем случае мы имеем дело с тестовой установкой, мы запретим межсетевые экраны на всех трех узлах. Запустим эти команды с машины **ceph-node1**:

```
# service iptables stop
# chkconfig iptables off
# ssh ceph-node2 service iptables stop
# ssh ceph-node2 chkconfig iptables off
# ssh ceph-node3 service iptables stop
# ssh ceph-node3 chkconfig iptables off
```

Развернем монитор на **ceph-node2** и **ceph-node3**

```
# ceph-deploy mon create ceph-node2
# ceph-deploy mon create ceph-node3
```

Мы можем столкнуться с предупредительными сообщениями, вызванными расфазировкой синхросигналов на новых узлах мониторов. Чтобы решить эту проблему, нуж6о установить **Network Time Protocol (NTP)** на новых узлах монитора:

```
# chkconfig ntpd on
# ssh ceph-node2 chkconfig ntpd on
# ssh ceph-node3 chkconfig ntpd on
# ntpdate pool.ntp.org
# ssh ceph-node2 ntpdate pool.ntp.org
# ssh ceph-node3 ntpdate pool.ntp.org
# /etc/init.d/ntpd start
# ssh ceph-node2 /etc/init.d/ntpd start
# ssh ceph-node3 /etc/init.d/ntpd start
```

#####Добавление OSD

Для осуществления этой задачи мы будем выполнять последующие команды с машины **ceph-node2**:

```
# ceph-deploy disk list ceph-node2 ceph-node3
# ceph-deploy disk zap ceph-node2:sdb ceph-node2:sdc ceph-node2:sdd
# ceph-deploy disk zap ceph-node3:sdb ceph-node3:sdc ceph-node3:sdd
# ceph-deploy osd create ceph-node2:sdb ceph-node2:sdc ceph-node2:sdd
# ceph-deploy osd create ceph-node3:sdb ceph-node3:sdc ceph-node3:sdd
# ceph status
```