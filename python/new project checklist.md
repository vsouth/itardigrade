## start a new project
### [[poetry]]
- [ ] > poetry new study-api
- [ ] > poetry shell
- [ ] ctrl+shift+p > python: select interpreter > choose the poetry env one
### .env + [[git#.gitignore]]
- [ ] .env
- [ ] > poetry add dotenv
- [ ] .gitignore
```
*.pyc
*.egg-info
# Настройки и хранение секретных данных 
/.env
```
### [[git]]
- [ ] > git init
- [ ] > git add .
- [ ] > git commit -m "Started" # or smth like that
- [ ] stuff on github website    
- [ ] > git branch -M main
- [ ] > git remote add origin git@github.com:vsouth/study-api.git
- [ ] > git push -u origin main
--- 
## do smth
### app
- [ ] setup.py
- [ ] db
	- [ ] models > [[sqlalchemy]]
	- [ ] migrations > [[alembic]]
- [ ] api
	- [ ] serialization
	- [ ] handlers
### unit-tests
- [ ] handlers
- [ ] migrations
### deploy and things like that?
- [ ] docker
- [ ] CI
- [ ] deploy

### load testing
- [ ] 
