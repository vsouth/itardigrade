# Links
https://habr.com/ru/articles/470285/ - about SQLalchemy
https://prettyprinted.com/flasksql/
https://soshace.com/optimizing-database-interactions-in-python-sqlalchemy-best-practices/

https://habr.com/ru/companies/domclick/articles/581304/


---
# Notes
[[get_or_create()]]

---
# Workflow
## model.py < tables of db
```
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Date

Base = declarative_base()

class Table(Base):
    __tablename__ = "Tablename"
    id = Column(Integer, primary_key=True)
    column_name = Column(<Type>)
    
    def __repr__(self):
        return f"<Tablename(column_name={self.column_name})>"
```
## config.py < uri of db
```
# in .env file:
# Scheme: "postgresql+psycopg2://<USERNAME>:<PASSWORD>@<IP_ADDRESS>:<PORT>/<DATABASE_NAME>"
DATABASE_URI=postgresql+psycopg2://postgres:nimda@localhost:5432/books

# in config.py file later:
import os
from dotenv import load_dotenv

load_dotenv()
DATABASE_URI = os.getenv("DATABASE_URI")
```
## crud.py or db.py
- create an engine (crud.py | db.py) + connect to db
```
from sqlalchemy import create_engine
engine = create_engine(DATABASE_URI)
```
- create tables
```
from models import Base

def create_database():
    Base.metadata.create_all(engine)
```
- create a session to interact with the new table
``` in crud.py file:
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)

# later
s = Session()
s.close()
```
- or use [[session scope]]:
```
@contextmanager
def session_scope():
    session = Session()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()


if __name__ == "__main__":
    with session_scope() as s:
        print(*s.query(Book).all(), sep="\n")
```
## Insert data
```
# load data from yaml file
recreate_database()
s = Session()
for data in yaml.load_all(open("books.yaml"), Loader=yaml.SafeLoader):
    book = Book(**data)
    s.add(book)

s.commit()

print(*s.query(Book).all(), sep="\n")

s.close()
```
So,
- *s = Session()* %% just use session_scope() as s, please %%
- use model to store data
- s.add(model)
- *s.commit()*
- *s.close()*
## Change data
```
book = s.query(Book).filter(Book.title == "Mother").first()
book.author = "J. Mama"
```
## Delete data
```
book = s.query(Book).filter(Book.title == "Mother").first()
s.delete(book)
```
## Queries
query == SELECT
filter, filter_by == WHERE
- filter_by - for simpler queries
- filter - for complex queries, more verbose and more readable
```
r = s.query(Book).filter_by(title="Deep Learning").first()
r = s.query(Book).filter(Book.title == "Deep Learning").first()
```
ilike == ILIKE
- ilike - works with filter !
```
r = s.query(Book).filter(Book.title.ilike('deep learning')).first()
```
between == BETWEEN
```
start_date = datetime(2009, 1, 1)
end_date = datetime(2012, 1, 1)

s.query(Book).filter(Book.published.between(start_date, end_date)).all()
```
and_, or_ need to be imported!
```
from sqlalchemy import and_, or_

r = s.query(Book).filter(
        and_(
            Book.pages > 750, 
            Book.published > datetime(2016, 1, 1)
            )
        ).all()
```
order_by, limit
```
r = s.query(Book).order_by(Book.pages.desc()).limit(2).all()
```
BESIDES, it's possible to:
```
s.query(Book)\
   .options(joinedload('author'))\
   .filter(...)\
   .filter(...)\
   .order_by(...)\
   .limit()\
   .all()
```
## Join
https://soshace.com/optimizing-database-interactions-in-python-sqlalchemy-best-practices/
https://habr.com/ru/companies/domclick/articles/581304/
https://prettyprinted.com/flasksql/

**back_populates vs backref**
	back_populates подробнее и легче читается, тк надо указать у обеих таблиц

**Author --> Book <--(book_genre)--> Genre**

Author:
	books = relationship("Book", back_populates="author")
	%% у автора.книги, у книги.автор %%
Book:
	author_id = Column(Integer, ForeignKey("authors.id"))
    author = relationship("Author", back_populates="books")
    genres = relationship("Genre", secondary="book_genre")
Genre:
	%% ничего, потому что не планировала жанр.книги смотреть, хотя мб надо было. зависит от целей  %%
book_genre:
	Table("book_genre", Base.metadata,
    Column("book_id", Integer, ForeignKey("books.id"), primary_key=True),
    Column("genre_id", Integer, ForeignKey("genres.id"), primary_key=True),)
    %% table вместо класса - потому что напрямую не взаимодействуем, нет доп. инфы важной %%
```
class Author(Base):
    __tablename__ = "authors"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    biography = Column(Text)

    books = relationship("Book", back_populates="author")

    def __repr__(self):
        return (
            f" <Author(id={self.id}, name='{self.name}', biography='{self.biography}')>"
        )


class Genre(Base):
    __tablename__ = "genres"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    description = Column(Text)

    def __str__(self):
        return f" <Genre(id={self.id}, name='{self.name}', description='{self.description}')>"

    def __repr__(self):
        return f"{self.name}"


class Book(Base):
    __tablename__ = "books"
    id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)

    author_id = Column(Integer, ForeignKey("authors.id"))

    author = relationship("Author", back_populates="books")
    genres = relationship("Genre", secondary="book_genre")

    def __repr__(self):
        return f"<Book(id={self.id}, title='{self.title}', author='{self.author.name}', genres={self.genres})>"


book_genre = Table(
    "book_genre",
    Base.metadata,
    Column("book_id", Integer, ForeignKey("books.id"), primary_key=True),
    Column("genre_id", Integer, ForeignKey("genres.id"), primary_key=True),
)
```
- add data in both tables
- make several queries
- [[sqlalchemy 2]]
	- joinedload + load_only - имба

## Migrations > [[alembic]]


---
# Something else
[[sqlalchemy 3]]

## Получение данных

Чтобы сделать запрос в базу данных используется метод `query()` объекта `session`. Он возвращает объект типа `sqlalchemy.orm.query.Query`, который называется просто `Query`. Он представляет собой инструкцию `SELECT`, которая будет использована для запроса в базу данных. В следующей таблице перечислены распространенные методы класса `Query`.

|Метод|Описание|
|---|---|
|_all()_|Возвращает результат запроса (объект `Query`) в виде списка|
|_count()_|Возвращает общее количество записей в запросе|
|_first()_|Возвращает первый результат из запроса или `None`, если записей нет|
|_scalar()_|Возвращает первую колонку первой записи или `None`, если результат пустой. Если записей несколько, то бросает исключение `MultipleResultsFound`|
|_one_|Возвращает одну запись. Если их несколько, бросает исключение `MutlipleResultsFound`. Если данных нет, бросает `NoResultFound`|
|_get(pk)_|Возвращает объект по первичному ключу (`pk`) или `None`, если объект не был найден|
|_filter(*criterion)_|Возвращает экземпляр `Query` после применения оператора WHERE|
|_limit(limit)_|Возвращает экземпляр `Query` после применения оператора LIMIT|
|_offset(offset)_|Возвращает экземпляр `Query` после применения оператора OFFSET|
|_order_by(*criterion)_|Возвращает экземпляр `Query` после применения оператора ORDER BY|
|_join(*props, **kwargs)_|Возвращает экземпляр `Query` после создания SQL INNER JOIN|
|_outerjoin(*props, **kwargs)_|Возвращает экземпляр `Query` после создания _SQL LEFT OUTER JOIN_|
|_group_by(*criterion)_|Возвращает экземпляр `Query` после добавления оператора GROUP BY к запросу|
|_having(criterion)_|Возвращает экземпляр `Query` после добавления оператора HAVING|