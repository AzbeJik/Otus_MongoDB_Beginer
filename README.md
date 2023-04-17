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
use admin
db.createUser(
   {
       user: "Admin", 
       pwd: passwordPrompt(), 
       roles:["root"]
   })
```
##
2. Заполнение базы

2.1 При помощи https://generatedata.com/generator генерируем данные и импортируем в бд (использовал формат JSON)

` mongoimport -d otus -c less2 --file /tmp/data.json --jsonArray -u root --authenticationDatabase admin `

![image](https://user-images.githubusercontent.com/121313424/231906301-ab5a2264-1948-49bf-8bab-2b77bc8c4736.png)

``` 
otus> db.less2.countDocuments()
500
```
![image](https://user-images.githubusercontent.com/121313424/231908180-bf1c9094-f5f0-4c3d-839d-3a643fefe48f.png)

###
3. Выборка и обновление данных

3.1 Вывести города имеющие больше одной записи и колличество повторений:

```sql
db.less2.aggregate([
  {
    $group: {
      _id: "$city",
      count: { $sum: 1 }
    }
  },
  {
    $match: {
      count: { $gt: 1 }
    }
  },
  {
    $sort: {
      count: -1
    }
  }
])
```

Результат: 

![image](https://user-images.githubusercontent.com/121313424/232605318-da21e777-5af1-4fd5-8557-fa3cf2a4cadc.png)


Вывести города имеющие больше одной записи и колличество повторений и их номера

Код: 
```sql
 use('otus');
config.set("displayBetchSize", 300)

db.less2.aggregate([
    {
      $group: {
        _id: "$city",
        count: { $sum: 1 },
        ids: { $push: "$_id" }
      }
    },
    {
      $match: {
        count: { $gt: 1 }
      }
    },
    {
      $sort: {
        _id: 1
      }
    }
  ]) 
```
![image](https://user-images.githubusercontent.com/121313424/232605367-10e322c8-05d6-4ee4-87cc-8a4a445cfab0.png)

3.2 Вывести самого имя индентификатор и возраст самого молодого человека в БД

Код:
```sql
use('otus');

db.less2.aggregate([
    {
      $project: {
        name: 1,
        numberrange: 1,
        age: {
          $floor: {
            $divide: [
              { $subtract: [new Date(), { $toDate: "$birthdaydate" }] },
              31536000000 // количество миллисекунд в году
            ]
          }
        }
      }
    },
    {
      $sort: { age: 1 } // сортируем возраст по возрастанию
    },
    {
      $limit: 1 // выбираем только самого молодого человека
    }
  ])
```

![image](https://user-images.githubusercontent.com/121313424/232602636-3e1fc5c7-03aa-424a-bc42-e1e294cc35fb.png)









