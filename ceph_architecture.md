#Архитектура и компоненты Ceph

##Архитектура системы хранения Ceph

Кластер хранения данных состоит из нескольких различных демонов программного обеспечения. Каждый из этих демонов заботится об уникальной функциональности Ceph и добавляет ценность для своих соответствующих компонент.

![Дифграмма стурктуры Ceph](./img/ceph_arch_diagram)

####RADOS

Безотказное автономное распределенное хранилище объектов (RADOS, Reliable Autonomic Distributed Object Store) является основой хранения данных кластера Ceph. Все в Ceph хранится в виде объектов, а хранилище объектов RADOS отвечает за хранение этих объектов, независимо от их типа данных.

Слой RADOS гарантирует, что данные всегда остаются в согласованном состоянии и надежны. Для согласованности данных он выполняет репликацию данных, обнаружение отказов и восстановление данных, а также миграцию данных и изменение баланса в узлах кластера.

####OSD

Устройство хранения объектов Ceph (OSD, Object Storage Device) хранит все клиентские данные в виде объектов и предоставляет те же данные клиентам при получении от них запросов. Кластер Ceph состоит из множества OSD. Для любой операции чтения или записи клиент запрашивает у мониторов карты кластера, а после этого он могут непосредственно взаимодействовать с OSD для выполнения операций ввода/вывода, без вмешательства монитора.

Каждый объект в OSD имеет одну первичную копию и несколько вторичных копий, которые разбросаны по остальным OSD. Поскольку Ceph является распределенной системой, а объекты распределены по нескольким OSD, каждый OSD выступает первичным OSD для некоторых объектов, и в то же время, он становится вторичным OSD для других объектов. Вторичный OSD остается под контролем основного OSD; однако, они способны стать первичным OSD.

В случае сбоя диска демон Ceph OSD интеллектуально обменивается данными с другими равноправными OSD для выполнения операции восстановления. В течение этого времени хранящие реплицированные копии отказавших объектов вторичные OSD повышают ранг до первичных, и одновременно в ходе операции восстановления OSD создаются новые копии вторичных объектов, причем это полностью прозрачно для клиентов.

####MON

Мониторы Ceph (MON, Ceph monitor) отвечают за контроль состояния работоспособности всего кластера. Это демоны, которые поддерживают режим членства в кластере путем сохранения важной информации о кластере, состояния одноранговых узлов, а также информации о настройках кластера. Монитор Ceph выполняет свои задачи поддерживая главную копию кластера. Карта кластера содержит карты монитора, OSD, PG, CRUSH и MDS.