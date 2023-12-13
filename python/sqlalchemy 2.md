https://habr.com/ru/companies/domclick/articles/581304/

[[sqlalchemy 3]]

Подробнее про сессии и рекомендации по их использованию можно прочитать [здесь](https://docs.sqlalchemy.org/en/14/orm/session_basics.html#session-faq-whentocreate).
![[Pasted image 20231124013426.png]]
#### Пример 1

Хотим получить десять последних вопросов из определенного топика.

```
s.query(Question).filter(
		Question.topic_id == 1
		).order_by(Question.id.desc()).limit(10).all()
```

#### Пример 2

Теперь мы хотим найти такие вопросы из первого топика, в тексте которых либо содержится подстрока `fart`, либо не содержится упоминания `dog`, независимо от их регистра. Мы легко изобразим это на SQL:

```
s.query(Question).filter(
		Question.topic_id == 1,
		_or(
			Question.text.ilike("%fart%"),
			~Question.text.ilike("%dog%"))
		)
		
for q in questions: 
	print(q.id)
```
%% запрос исполняется в тот момент, когда начинаем итерировать по вопросам, до этого можно было еще добавить лимит, например %%

#### Пример 3

Этот пример подводит нас к теме оптимизации запросов. 
Допустим, мы хотим получить данные не только самого вопроса, но и связанного с ним топика. В алхимии действует схожий принцип. 
Работа «в лоб» (делает два запроса, сначала за вопросом, потом за топиком):
```
q = s.query(Question).filter(Question.id == 1).first() # sa.3.1 

print(q.topic.title) # sa.3.2
```
**Чтобы отправить в базу только один запрос, нужно воспользоваться опциями:**
```
q = s.query(Question).filter( 
	Question.id == 1, 
).options(joinedload(Question.topic)).first()

print(q.topic.title)
```
В алхимии этот подход носит название Eager Loading, вот [ссылка](https://docs.sqlalchemy.org/en/13/orm/tutorial.html#eager-loading) на примеры. Опции загрузки можно указывать не только при формировании запроса, но и при [описании модели](https://docs.sqlalchemy.org/en/13/orm/loading_relationships.html). Некоторые предпочитают явно указывать опции загрузки именно при формировании запроса.

==ПОЧИТАТЬ ПРО INNER JOIN И OUTER JOIN==
Но `LEFT JOIN` в нашей схеме данных излишен, потому что не может быть вопроса, не привязанного к топику. Мы можем подсказать алхимии при определении relation-а использовать именно **INNER JOIN, который более производителен, чем OUTER**:

```
topic = sa.orm.relationship(Topic, innerjoin=True)
```



#### Пример 4

Этот пример углубит наши знания по технике нетерпеливой подгрузки. Допустим, нам нужен не только связанный с вопросом топик, но ещё и изображение этого топика. В алхимии (если говорить о работе внутри сессии) «магия» библиотеки позволяет получить доступ к связанным объектам без дополнительных действий со стороны разработчика. 

Чтобы явно попросить алхимию заняться нетерпеливой подгрузкой, снова используем опции. На этот раз две:

```
with session_scope() as s:
    for q in s.query(Question).options(
        joinedload('topic'),
        joinedload('topic.image'),
    ).filter(Question.topic_id == 1).all():  # sa.4.4
        print(q.topic.title)  # sa.4.5
        print(q.topic.image.name)  # sa.4.6
```

В этом случае только `sa.3.4` спровоцирует запрос в базу.

%% Т.е. тут еще и двойной переход вглубь отношений? от вопроса к топику, плюс к топик.картинке?? типа поэтому отдельно написано, тк просто топика недостаточно чтобы была и топик.картинка %%

#### Пример 5

Допустим, теперь мы хотим обработать все вопросы у каждого топика. Как и в Django, для решения этой задачи можно написать обычный запрос с фильтрацией. Так мы получим простое, но не оптимизированное решение (запрос за списком топиков и запросы за вопросами на каждый топик):

```
for t in s.query(Topic).filter(Topic.id.in_([1, 2])).all():  # sa.5.1
    print([q.text for q in t.questions])  # sa.5.2
```

Улучшенный вариант (один запрос):
```
for t in s.query(Topic).options(joinedload('questions')).filter(Topic.id.in_([1, 2])).all():
    print([q.text for q in t.questions])
```


#### Пример 6

Допустим, теперь нам нужно подгрузить к топику не все вопросы, а только те, которые отвечают определенному условию. При этом у некоторых топиков может оказаться пустой список вопросов.

==В алхимии фильтровать массив вопросов можно с помощью новой опции и алиаса, что в совокупности выглядит не самым тривиальным способом.==


```
q_stmt = s.query(Question).filter(Question.text.ilike('%fart%')).subquery()  # sa.6.1
alias = aliased(Question, q_stmt)  # sa.6.2

filtered_topics = s.query(Topic).outerjoin(  # sa.6.3
    Topic.questions.of_type(alias),  # sa.6.4    
).options(
    contains_eager(Topic.questions.of_type(alias)),  # sa.6.5
)

for t in filtered_topics:
    print(f'{t.title}')
    for q in t.questions:
        print(f' --- {q.text}')
```

- `sa.6.1`: создаем подзапрос, выбирающий интересующие нас вопросы. В базу запрос на этом этапе еще не уходит.
    
- `sa.6.2`: создаем алиас. Они обычно используются для `SELF JOIN`-а или для замены оригинальной таблицы представляющим её запросом.
    
- `sa.6.3`: явно указываем, что нужно использовать `LEFT OUTER JOIN`.
    
- `sa.6.4`: за счет `of_type` мы джойним не с целой таблицей, а с тем подзапросом, который нам нужен.
    
- `sa.6.5`: просим в `Topic.questions` подгрузить сущности не отдельным запросом, а уже из сформированных нами колонок. Подробнее см. [здесь](https://docs.sqlalchemy.org/en/13/orm/loading_relationships.html#sqlalchemy.orm.contains_eager).

#### Пример 7

В этом примере мы познакомимся с группировкой и подзапросами. Для этого найдем топики, у которых больше десяти вопросов.

В алхимии применяется схожий с Django подход: сначала формируем подзапрос, а потом используем его в основном селекте:
```
topic_ids = s.query(
    Question.topic_id.label('topic_id'),
).group_by(Question.topic_id).having(
    sa.func.count(Question.id) > 1,
)

topics = s.query(Topic).filter(Topic.id.in_(topic_ids))

for t in topics:
    print(f'{t.title}')
```


#### Пример 8

Этот пример посвящен обзору ==common table expression== (CTE), которыми частенько пользуешься при написании сырых запросов к PostgreSQL ради читаемости и удобства.

Допустим, мы хотим достать из базы топики, у которых определенных вопросов больше десяти. Сначала формируем CTE, в котором найдем идентификаторы нужных топиков:
```
topic_ids = s.query(Question.topic_id.label('topic_id')).filter(
    Question.text.ilike('%best%'),
).group_by(Question.topic_id).having(
    sa.func.count(Question.id) > 10,
).cte(name='topic_ids')
```

Далее используем CTE в основном запросе, чтобы достать соответствующие топики:

```
topics = s.query(Topic).filter(Topic.id.in_(sa.select([topic_ids.c.topic_id])))
print(', '.join(t.title for t in topics))
```

#### Пример 9

Здесь рассмотрим запросы к many-to-many отношениям. Допустим, мы хотим для каждого топика получить список пользователей, которые имеют к нему доступ.

И снова можно сгенерировать запрос без дополнительных усилий с нашей стороны, библиотека всё делает за нас. Однако за такую помощь приходится расплачиваться. Если не указать опции, то для каждого топика будет отдельный запрос за списком пользователей. Лучше:
```
for t in s.query(Topic).options(
        joinedload('users'),
).all():
    names = ', '.join(u.name for u in t.users)
    print(f'Topic[{t.title}]: {names}')
```

Если в промежуточной таблице содержатся дополнительные данные, к которым мы хотим получать доступ (у нас это поле `TopicUser.role`), то нужно описывать модели согласно [паттерну Association Object](https://docs.sqlalchemy.org/en/20/orm/basic_relationships.html#association-object). В наших схемах смотри закомментированные строки под комментарием `# association.` Там же есть пример «обратных» зависимостей: когда мы получаем не коллекцию пользователей топика, а набор топиков, доступных пользователю.
```
u = s.query(User).filter(User.id == 1).options(
    joinedload('topics'),
    selectinload('topics.topic'),
).first()

for t in u.topics:
    print(f'{u.name} is {t.role} in topic "{t.topic.title}"')
```
==selectinload()== - ?

	А для конкретных листингов оптимизацией было бы что-то вроде (что-то вокруг #sa.9.1, #sa.9.2):

```
query = sa\
  .select(Topic.id.label('id'), sa.func.array_agg(User.name).label('users'))\
  .where(sa.and_(
	Topic.id == TopicUser.topic_id,
	TopicUser.user_id == User.id
  ))\
  .group_by(sa.literal_column('1'))

with session as s:
  for row in s.execute(query):
	print(f'Topic {row.id}, Users: {", ".join(row.users)}')
```

	Не надо запрашивать то, что не требуется. Если требуется получить только определенные колонки, не надо просить ORM вытащить модель целиком. Гонять новый запрос на каждую строчку связанной модели - моветон. Если средства СУБД позволяют объединить (или схлопнуть несколько строк в одну без потери данных - позвольте это сделать СУБД. Она, во-первых, скорее всего, сделает это эффективнее, во-вторых, будет меньше накладных расходов на запрос и передачу result tuples.

#### Пример 10

Приступим к другой стороне оптимизации. При вставке большого количества новых записей для улучшения производительности рекомендуется пользоваться bulk-операциями. Они позволяют за один запрос вставить много строк.
```
s.bulk_save_objects([
    Topic(title=f'topic {i}', image_id=1)
    for i in range(3)
])
```

#### Пример 11.

Другая сторона bulk-операций — это массовое обновление. В алхимии можно не создавать экземпляры моделей, а ограничиться словарями с PK (primaty key) и полями для обновления.
```
s.bulk_update_mappings(
    User,
    [dict(id=1, name='new 1 name'), dict(id=2, name='new 2 name')]
)
```

