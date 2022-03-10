# mongodb
## Задание

__Цель:__ В результате выполнения ДЗ вы научитесь разворачивать MongoDB, заполнять данными и делать запросы.

__Описание/инструкция выполнения домашнего задания:__
1. Установить MongoDB одним из способов: локально, докер или ВМ(облачный сервис);
2. Заполнить данными (примеры датасетов https://habr.com/ru/company/edison/blog/480408/);
3. Написать несколько запросов на выборку и обновление данных;
4. Создать индексы и сравнить производительность.

## Решение 

### Установка на ubuntu через командную строку

#### mongodb
```
$ wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
$ apt-key list
$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
$ deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse
$ sudo apt update
$ sudo apt install mongodb-org
```

#### MongoDB Compass
```
$ wget https://downloads.mongodb.com/compass/mongodb-compass_1.26.1_amd64.deb
$ sudo apt install ./mongodb-compass_1.26.1_amd64.deb
```

### Простые команды
После открытия MongoDB shell
```
$ mongo
```

Создадим базу данных, добавим коллекцию и в неё добавим данные
```
> use example
switched to db example
> db.createCollection("users")
{ "ok" : 1 }
> db.users.insertOne({"name":"Ally", "email":"ally@gmail.com", "age":18})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("6229d619ca04649fd3befee5")
}
> db.users.insertOne({"name":"Sam", "email":"sammy@gmail.com"})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("6229d6c2ca04649fd3befee6")
}
```

Так выглядит коллекция users
![](https://github.com/FAnastasiaF/mongodb/blob/main/1.png)


### Заполнение данными
Для заполнения я выбрала [Jeopardy Dataset](https://www.reddit.com/r/datasets/comments/1uyd0t/200000_jeopardy_questions_in_a_json_file/) —  216,930 записей «вопрос-ответ» из телевизионной игры.

После открытия MongoDB shell
```
$ mongo
```

Создадим базу данных, добавим коллекцию 
```
> use Jeopardy
switched to db Jeopardy
> db.createCollection("data")
{ "ok" : 1 }
```

Дальше заходим в MongoDB Compass и переходим в коллекцию Jeopardy.data и загружаем наш json файл
![](https://github.com/FAnastasiaF/mongodb/blob/main/2.png)

Так выглядит наша коллекция
![](https://github.com/FAnastasiaF/mongodb/blob/main/3.png)
![](https://github.com/FAnastasiaF/mongodb/blob/main/4.png)

### Обновление данных

#### Меняем все null value на $0
![](https://github.com/FAnastasiaF/mongodb/blob/main/6.png)

Было
![](https://github.com/FAnastasiaF/mongodb/blob/main/5.png)

Стало
![](https://github.com/FAnastasiaF/mongodb/blob/main/7.png)

#### Оставляем в поле value только цифры
![](https://github.com/FAnastasiaF/mongodb/blob/main/8.png)

Было
![](https://github.com/FAnastasiaF/mongodb/blob/main/9.png)

Стало
![](https://github.com/FAnastasiaF/mongodb/blob/main/10.png)

#### Пребразуем строку, состоящую из чисел, в число
![](https://github.com/FAnastasiaF/mongodb/blob/main/11.png)

Было
![](https://github.com/FAnastasiaF/mongodb/blob/main/10.png)

Стало
![](https://github.com/FAnastasiaF/mongodb/blob/main/12.png)

### Запросы на выборку

#### Получаем Топ 3 по стоимости вопросов по истории с 2000 года до 2005 года 
В командной строке
![](https://github.com/FAnastasiaF/mongodb/blob/main/13.png)
В MongoDB Compass
![](https://github.com/FAnastasiaF/mongodb/blob/main/14.png)

#### Уникальные категории, у которых ответ 15 и 17 века
![](https://github.com/FAnastasiaF/mongodb/blob/main/15.png)

#### Первые 10 строк с вопросами про столицу в раундах Final Jeopardy! и Jeopardy!, цена которых была не меньше 600, отсортированые по ответу
```
db.data.find({round: {$in: ["Final Jeopardy!", "Jeopardy!"]}, question: {$regex: /This capital/}, value:{$gte: 600}}).sort({answer: 1}).limit(10)

```
![](https://github.com/FAnastasiaF/mongodb/blob/main/16.png)

### Создание индексов и сравнение производительности
#### Создаём индекс
![](https://github.com/FAnastasiaF/mongodb/blob/main/17.png)
![](https://github.com/FAnastasiaF/mongodb/blob/main/18.png)
![](https://github.com/FAnastasiaF/mongodb/blob/main/19.png)
Без индекса
![](https://github.com/FAnastasiaF/mongodb/blob/main/20.png)
С индексом
![](https://github.com/FAnastasiaF/mongodb/blob/main/21.png)

##### Мы ощутимо улучшили время работы за счёт индексации
Вместо перебора большой коллекции целиком, мы перебираем индекс, который проще добавить в память и работать с ним.

#### Создаём составной индекс
![](https://github.com/FAnastasiaF/mongodb/blob/main/22.png)

Без индекса
![](https://github.com/FAnastasiaF/mongodb/blob/main/23.png)
С cоставным индексом
![](https://github.com/FAnastasiaF/mongodb/blob/main/24.png)
С индексом значения
![](https://github.com/FAnastasiaF/mongodb/blob/main/25.png)
С индексом категории
![](https://github.com/FAnastasiaF/mongodb/blob/main/26.png)

##### Тут тоже видно улучшение времени работы
Также можем заметить, что время работы зависит от порядка в find
