# Links
https://www.freecodecamp.org/news/how-to-setup-virtual-environments-in-python/
https://docs.python.org/3/library/venv.html

--- 
# Usual workflow

---
# Notes
venv - создавать на уровень выше, чтобы не лезло в гит (а потом cd на уровень ниже и активировать), либо добавлять в гитигнор

---
# Commands
## To start
python -m venv /path/to/new/virtual/environment
## To work in env
env/Scripts/Activate.ps1 *( in powershell )*
## To stop working in env
deactivate
## To add modules
pip install %%smth%%
## To remove modules
pip uninstall %%smth%%
## To make requirements.txt
pip freeze > requirements.txt
## To show modules
pip list
## To install
pip install -r requirements.txt
