# Links
https://python-poetry.org/docs/cli/

---
# Usual workflow
## Create env
- poetry new %%study-api%%
- poetry init
## Or install env
- ?
## Work in it
- poetry shell
- ctrl+shift+p > python: select interpreter > choose the poetry env one

---
# Notes
poetry new %%study-api%% - создает новую директорию!! 
poetry shell - баганый, если отказывается работать тк уже запущен енв:
- poetry env list - чтобы найти путь
- потом активировать вручную
	C:\\Users\\humanoid\\AppData\\Local\\pypoetry\\Cache\\virtualenvs\\youtube-downloader-VQGRZMYJ-py3.11\\Scripts\\activate.ps1

--- 
# Commands
## To start
- poetry new %%study-api%% *( создает новую директорию!! )*
- poetry init
## To work in env
- poetry shell
- ctrl+shift+p > python: select interpreter > choose the poetry env one
## To stop working in env
- deactivate
## To add modules
- poetry add %%smth%%
## To remove modules
- poetry remove %%smth%%
## To update
- poetry update
## To show modules
- poetry show
## To install
- ?
