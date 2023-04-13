Цель:

1. Установить MongoDB одним из способов: ВМ, докер;
2. Заполнить данными;
3. Написать несколько запросов на выборку и обновление данных
4. Создать индексы и сравнить производительность.

#
1. Установка:

 1.1 Добавление репозитория и установка MongoDB последней стабильной версии на opensuse 15
```bash
sudo zypper ref
sudo zypper up -y
sudo rpm --import https://www.mongodb.org/static/pgp/server-6.0.asc
sudo zypper addrepo --gpgcheck "https://repo.mongodb.org/zypper/suse/15/mongodb-org/6.0/x86_64/" mongodb
sudo zypper -n install mongodb-org
sudo systemctl enable mongod
sudo systemctl enable mongod
sudo firewall-cmd --add-port=27017/tcp --permanent
sudo firewall-cmd --reload
```
![image](https://user-images.githubusercontent.com/121313424/231887287-210954c6-a4d6-4706-a05e-aa84ad0c4b60.png)

 1.2 Конфигурирование mongoDB

Замена конфига /etc/mongod.conf 

<img width="251" alt="1" src="https://user-images.githubusercontent.com/121313424/231899719-82b761c3-aa79-473f-affc-ba7994eb3f8d.png"> 

` sudo systemctl restart mongod `

1.3 Создание админа в БД в mongosh

```sql
sql use admin
db.createUser(
{
user: "mongouser",
pwd: passwordPrompt(), // or cleartext password
roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
}
)
```
##
2. Заполнение базы

2.1 При помощи https://generatedata.com/generator генерируем данные и импортируем в бд (использовал формат JSON)

` mongoimport -d otus -c less2 --file /tmp/data.json --jsonArray -u root --authenticationDatabase admin `

![image](https://user-images.githubusercontent.com/121313424/231906301-ab5a2264-1948-49bf-8bab-2b77bc8c4736.png)





