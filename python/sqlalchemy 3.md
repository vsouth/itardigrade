https://habr.com/ru/companies/true_engineering/articles/226521/



##### [Difference between SQLAlchemy Select and Query API](https://stackoverflow.com/questions/72828293/difference-between-sqlalchemy-select-and-query-api)
The biggest difference is how the `select` statement is constructed. The new method creates a [`select` object](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.select) which is more dynamic since it can be constructed from other select statements, [without explicit subquery definition](https://docs.sqlalchemy.org/en/14/core/selectable.html#selectable-foundational-constructors):

```python
# select from a subqeuery styled query
q = select(Users).filter_by(name='name', email='mail@example.com')
q = select(Users.name, Users.email).select_from(q)
```

The outcome is more "native sql" construction of querying, as per the latest [selectable API](https://docs.sqlalchemy.org/en/14/core/selectable.html#sqlalchemy.sql.expression.Select). Queries can be defined and passed throughout statements in various functionalities such as where clauses, having, select_from, intersect, union, and so on.

Version 2.0 removes many differences between ORM and Core and makes working with them more uniform. Comparing select() and query() doesn't really make sense anymore.

