sqlalchemy
https://stackoverflow.com/questions/2546207/does-sqlalchemy-have-an-equivalent-of-djangos-get-or-create

чекнуть коммент про flush() вместо commit()
https://stackoverflow.com/questions/4201455/sqlalchemy-whats-the-difference-between-flush-and-commit


```python
def get_or_create(session, model, **kwargs):
    instance = session.query(model).filter_by(**kwargs).first()
    if instance:
        return instance
    else:
        instance = model(**kwargs)
        session.add(instance)
        session.commit()
        return instance
```