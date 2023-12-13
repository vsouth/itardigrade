[[sqlalchemy]]
### Session helper 
in srud.py or db.py ?
```
from contextlib import contextmanager

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