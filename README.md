
![Alt text](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/06/1403445197RabbitMQ.sh-600x600-300x300.png)
![Alt text](http://cdn.mysitemyway.com/etc-mysitemyway/icons/legacy-previews/icons-256/black-white-pearls-icons-alphanumeric/069302-black-white-pearl-icon-alphanumeric-plus-sign.png)
![Alt text](http://agileengine.com/wordpress/wp-content/uploads/2016/08/ansible-300x300.png)
```
├── README.md
├── add_node.yml				#Playbook по добавлению ноды в кластер RabbitMQ
├── add_user.yml				#Playbook по созданию пользователей на Production кластере и Developer кластере
├── create_cluster.yml				#Playbook по созданию класстера RabbitMQ
├── del_node.yml				#Playbook по удалинию ноды из кластера RabbitMQ
├── files
│   ├── deployment_id_rsa			#Privet key
│   ├── erlang.cookie				#Cookie используется для авторизации между нодами
│   ├── rabbitmq-server.service			#За тюннерный сервис, который разрешает открывать свыше 65К дескрипторов
│   └── rabbitmq.sh				#Скрипт который создает ENV для RabbitMQ
├── invetory
│   ├── add_user				#Invetory файл для добавления пользователей. Любая нода из кластера Production и кластера Developer
│   └── cluster					#Invetory файл для создания кластера, добавления хоста, удаления хоста.
├── tasks
│   ├── add_node.yml				#Task для добавления ноды в кластер
│   ├── production.yml				#Task по созданию пользователя на Production
│   ├── qa_and_stage.yml			#Task по созданию пользователя на Developer
│   └── slave.yml				#Task по настройки slave в кластере
├── templates
│   └── rabbitmq.config.j2			#Template конфиг файл для RabbitMQ
└── vars.yml					#Файл с переменными
```
# How this work?
## Создать user, vhost, policies.
#### Редактируем vars.yml. Редактируем только переменную new_user
```
######### required #########
new_user	: empty
############################

tag: monitoring
prod_vhost: 'prod_{{new_user}}'
stage_vhost: 'stage_{{new_user}}'
qa_vhost: 'qa_{{new_user}}'
```


#### Запускаем playbook
```
ansible-playbook add_user.yml -i invetory/add_user -b
```


#### На выходе получаем реквизиты подключения. Их и отправляем заказчику
```
		"############## PRODUCTION #################",
		"URL Monitor: https://clusterrmq.sberned.ru ",
		"HOST: rmqcluster.rmq ",
		"PORT: 5672 ",
		"USER: empty ",
		"PASSWORD: L5ozrnpt3HBIN79b ",
		"VHOST: prod_empty",
		"###########################################"
```
```
		"############## TEST STAND #################",
		"URL Monitor: http://tank.rmq:15672/ ",
		"HOST: tank.rmq ",
		"PORT: 5672 ",
		"USER: empty ",
		"PASSWORD: vF5tnLHQW3_wJmwV ",
		"VHOST: qa_empty, stage_empty",
		"###########################################"
```
## Создать cluster RabbitMQ. Добавить или удалить ноду.
* #### Создать cluster. Открываем invetory/cluster. В группе [master] добавляем одну ноду мастера, а в [slave] все остальные ноды.
```
	ansible-playbook create.yml -i invetory/cluster
```

* #### Добавить ноду в кластер. Открываем invetory/cluster. В группе [add] указываем hostname машин которые необходимо добавить в кластер
```
	ansible-playbook add.yml -i invetory/cluster
```

* #### Удалить ноду из кластер. Открываем invetory/cluster. В группе [delete] указываем hostname машин которые необходимо удалить из кластер
```
	ansible-playbook delete.yml -i invetory/cluster
```
