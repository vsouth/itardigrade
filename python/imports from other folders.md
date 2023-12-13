```
E:\CODE\ADDITIONAL\STUDY-DB
│   .env
│
├───alembic
│   │   env.py
│
├───study_db
│   │   models.py
│   │   __init__.py
```
in *study-db/alembic/env.py* to import from *.env* and *models.py* :
```
import sys, os

sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

import study_db.config as my_config

from study_db.models import Base
```
