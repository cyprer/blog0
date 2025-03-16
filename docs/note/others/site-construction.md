# 建站经历
+ 由于本人是Windows端,因此一项仅提供windows端的搭建教程
+ 本网站是基于mkdocs并用github page部署的,意味着你需要拥有自己的github账号
+ python环境,建议直接前往官网下载最新版即可
+ 一个文本编辑器,这边推荐vscode
+ git,直接去官网下载即可
+ 新建一个文件夹并用vscode打开,打开终端输入以下代码
+ python -m venv venv 
+ venv/scripts/activate(这里也许会无法识别venv,~~可以换个时间再试试~~)
+ pip install mkdocs-material
+ mkdocs new .
+ mkdocs serve
+ 在你键的文件夹下新建一个.github文件夹并在.github文件夹下新建一个workflows文件夹,然后在这个文件夹下创建一个ci.yml文件,内容是
```
    name: ci
on:
  push:
    branches:
      - master
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure Git Credentials
        run: |
          git config user.name github-actions[bot]
          git config user.email 41898282+github-actions[bot]@users.noreply.github.com
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV
      - uses: actions/cache@v4
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```
+ git init
+ 这里在venv文件夹下新建一个.gitnore文件内容是
```
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
.python-version

# pipenv
#   According to pypa/pipenv#598, it is recommended to include Pipfile.lock in version control.
#   However, in case of collaboration, if having platform-specific dependencies or dependencies
#   having no cross-platform support, pipenv may install dependencies that don't work, or not
#   install all needed dependencies.
#Pipfile.lock

# PEP 582; used by e.g. github.com/David-OConnor/pyflow
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/
```
+ 然后可以git add .
+ git commit -m "initial commit"
+ 创建一个github仓库(应该都会吧我累了)
+ git remote add 你自己的github仓库网站(以.git结尾)
+ git branch -M main
+ git push -u origin main
+ 然后转到github仓库点击seting,再点击pages,Branch下选择gh-pages并save
+ 然后在seting那一栏点击actions,你会看到workflows在部署你的网站等待一会就可以看到你的网站生成啦!!!