---

---

[toc]

.gitignore的作用？

```python
git软件，本地进行版本管理
	git init
    git add
    git commit
码云/github/gitlab,代码托管
```
## 2. Git仓库管理

### 2.1 创建gitee账号和远程仓库

```python
https://gitee.com/gonghaochen/saas.git
```

![image-20210301232843241](https://gitee.com/gonghaochen/blogimg/raw/master/img/20210320194510.png)

### 2.2 Git代码推送到远程仓库

.gitignore内容

```python
# pycharm
.idea/
.DS_Store

__pycache__/
*.py[cod]
*$py.class

# Django stuff:
local_settings.py
*.sqlite3

# database migrations
*/migrations/*.py
!*/migrations/__init__.py
```

git项目管理：cmd执行

```python
C:\Users\gong>E:

E:\>cd  E:\Python\ghc1\saas

E:\Python\ghc1\saas>git init
Initialized empty Git repository in E:/Python/ghc1/saas/.git/

E:\Python\ghc1\saas>git add .
warning: LF will be replaced by CRLF in .gitignore.
The file will have its original line endings in your working directory

E:\Python\ghc1\saas>git commit -m '第一次提交'
[master (root-commit) ea0bd8f] '第一次提交'
 14 files changed, 217 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 app01/__init__.py
 create mode 100644 app01/admin.py
 create mode 100644 app01/apps.py
 create mode 100644 app01/migrations/__init__.py
 create mode 100644 app01/models.py
 create mode 100644 app01/tests.py
 create mode 100644 app01/views.py
 create mode 100644 manage.py
 create mode 100644 requirements.txt
 create mode 100644 saas/__init__.py
 create mode 100644 saas/settings.py
 create mode 100644 saas/urls.py
 create mode 100644 saas/wsgi.py
```

将代码推送到仓库

```python
git remote add origin https://gitee.com/gonghaochen/saas.git # 关联远程仓库
    
git push origin master
```

获取仓库里的代码

```python
git clone https://gitee.com/gonghaochen/saas.git
```

```python
pip freeze > requirements.txt
pip install -r requirements.txt #别人安装项目所需模块

# 上传到仓库
git add .
git commit -m 'requirements'
git push origin master
```

![image-20210302002551945](https://gitee.com/gonghaochen/blogimg/raw/master/img/20210320173454.png)

```python
# 如果你想要保留本地当前改动：
git push -u origin master
 
# 如果不要当前的改动，请重置到库的最新版本：
git reset --hard origin/master
```

![image-20210302002606253](https://gitee.com/gonghaochen/blogimg/raw/master/img/20210320173503.png)

若是提示：error: remote origin already exists. 如果远程仓库已经改名

```python
git remote rm origin
```

![image-20210302003217535](https://gitee.com/gonghaochen/blogimg/raw/master/img/20210320193201.png)

git push origin master时，若提示：! [rejected]        master -> master (fetch first)

```python
git push -u origin master -f
```

git 不加注释

```python
git commit --allow-empty-message -m ""
```

### 2.3同时推送到github和gitee

```python
# 已经将代码上传到gitee 名字为origin
# 查看关联的远程仓库的名字
git remote 
>>>origin
# 查看关联的远程仓库地址
git remote -v
>>>origin  https://gitee.com/gonghaochen/blog.git (fetch)
>>>origin  https://gitee.com/gonghaochen/blog.git (push)
# 重命名
git remote rename origin gitee
>>>gitee   https://gitee.com/gonghaochen/blog.git (fetch)
>>>gitee   https://gitee.com/gonghaochen/blog.git (push)
# 关联 github远程仓库
git remote add github https://github.com/gonghaochen/blog.git
# 将代码推送
git push github master
```

