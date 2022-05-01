# Async Arch Homework 01

AUTH
==

База данных
--
users: userId, email, pass, role, name

API
--
- createUser(name, email, role) -> pass  
Продьюсит событие: userCreated(id, role)

- loginUser(login, pass) -> JWT

TASKS
==

База данных
--
**users**: userId, role  
**tasks**: taskId, text, status, assignee  

Слушает события
--
- userCreated(id, role)  
пишет в юзера в базу, если на него можно ассайнить задачи

API
--
- createTask(text, assignee)  
Пишет задачу в базу  
Вызывает assignTask(assignee), ведь задача не может быть создана без исполнителя  
Продюсит евент taskCreated(taskId, { text, assignee } )  

- assignTask(taskId, assignee)  
Продюсит евент taskChanged(taskId, { assignee })

- assignAll()  
Вызывает по всем юзерам из базы в цикле assingTask()

- completeTask(taskId)  
Продюсит евент taskChanged(taskId, { status: ‘done’ })

- list  
Выводит список задач из бд для конкретного юзера

ACCOUNTING
==

База данных
--
**Tasks**: id, text, assignee, fee, cost, createdAt, completedAt  

**Balances**: userId, balance  
В этой таблице есть запись для подсчета баланса компании (userId === 'UBER_POPUG_CO')  

**Logs**: userId, event, sum, createdAt  

Слушает события
--
- onTaskCreated(id, text, assignee)  
Вычисляем fee, cost, пишем в базу

- onTaskChanged(id, data)  
В транзакции:  
Если изменился assignee:
user.balance -= fee  
Если изменился status:
user.balance += cost  
Изменяем баланс компании  
Продюсит евент **paymentDone(userId, sum)**  
Продюсит евент **companyBalanceChanged(sum)**

- totalRevenue
Берем из базы, запись userId === 'UBER_POPUG_CO'

API
--
- list  
Выводим записи из таблицы **logs**

Cron раз в день - checkout
--
- user.balance > 0  
Пишем в базу log(userId, ‘checkout’, balance)  
Обнуляем баланс у юзера

- user.balance < 0  
Продюсим событие negativeBalance(userId, balance)  

NOTIFICATION
==
Слушает событие userCheckout, ищет его email в AUTH  
Либо сам отправляет емэйл, либо формирует евент, а емэйл отправляет уже AWESOME_EMAIL_SERVICE когда-нибудь потом

ANALYTICS
==
База данных
--
**Events**: event, ts, userId, value

Слушает события:
--
- companyBalanceChanged

- paymentDone(userId, 35)

- negativeBalance(userId, balance)

API
--
Все данные есть внутри бд, собираем по логам  
Баланс компании - последняя запись на заданную дату
- mostExpensiveTask()
- workersWithNegativeBalance()
- companyBalance()

ОБЩЕЕ
==
Проверка на роли
--
Либо каждый сервис сам проверяет/распаковывает JWT токен, либо синхронно стучит в AUTH и получает данные оттуда
