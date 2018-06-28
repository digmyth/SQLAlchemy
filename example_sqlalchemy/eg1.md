# example_sqlalchemy 完整示例学习

创建连接数据库指定字符集，否则表设定字符集将不生效
```
CREATE DATABASE db1 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

当有如下表
```
# models.py
import datetime
# from sqlalchemy import create_engine
# from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime, UniqueConstraint, Index
from pro_23 import db
from sqlalchemy.orm import relationship

class UserInfo(db.Model):
    """
    用户表
    """
    __tablename__ = 'userinfo'

    id = Column(Integer, primary_key=True)
    username = Column(String(32), unique=True)
    password = Column(String(64))
    nickname = Column(String(32))


class Host(db.Model):
    """
    主机表
    """
    __tablename__ = 'host'
    id = Column(Integer, primary_key=True)
    hostname = Column(String(32),unique=True)
    port = Column(Integer)


class Project(db.Model):
    """
    项目表
    """
    __tablename__ = 'project'

    id = Column(Integer, primary_key=True)
    title = Column(String(32), unique=True)
    name =  Column(String(32), unique=True)
    repository =  Column(String(128))

    hosts = relationship('Host',secondary='project_2_host',backref='projects')

class Project2Host(db.Model):
    """
    项目主机关系表
    """
    __tablename__ = 'project_2_host'

    id = Column(Integer, primary_key=True)
    project_id = Column(Integer,ForeignKey('project.id'))
    host_id = Column(Integer,ForeignKey('host.id'))

    __table_args__ = (
        UniqueConstraint('project_id','host_id',name='uix_project_host'),
    )


class Deploy(db.Model):
    """
    代码发布表
    """
    __tablename__ = 'deploy'
    id = Column(Integer, primary_key=True)

    project_id = Column(Integer, ForeignKey('project.id'))
    user_id = Column(Integer, ForeignKey('userinfo.id'))

    version = Column(String(32))

    status_choices = {
        1:'未发布',
        2:'已发布',
        3: '发布失败',
        4: '回滚',
    }
    status_id = Column(Integer,default=1)

    deploy_type_choices = {
        1: '代码',
        2: 'SQL',
        3: '静态文件',
    }
    deploy_type = Column(Integer,default=1)

    dtime = Column(DateTime)
    ctime = Column(DateTime,default=datetime.datetime.now)

    # 发布之后自动检测
    ext_path = Column(String(128),nullable=True)


class DeployRecord(db.Model):
    __tablename__ = 'deployrecord'

    id = Column(Integer, primary_key=True)

    deploy_id = Column(Integer, ForeignKey('deploy.id'))
    host_id = Column(Integer, ForeignKey('host.id'))

    status_choices = {
        1: '成功',
        2: '失败',
    }
    status_id = Column(Integer, default=1)
```

创建表
```
from manage import app  # 这个导入其实是单例
from pro_23 import db

# app = create_app()  # 虽然生成新的实例，但是我们这里只用到了app的应用上下文，属静态配置，所以无关是不是新的实例或单例（都可以）

with app.app_context():
    db.drop_all()
    db.create_all()
```

### 一、普通单表插入数据
在没有使用relationship字段时直接插入表数据可能写起来复杂些，主要复杂在手动处理多表关系（不要关闭前一个表的db.seession会话）
```
# db_init.py
from manage import app
from pro_23 import db,models

with app.app_context():
    # obj1 = models.UserInfo(username='wxq', password='123', nickname='digmyth'),

    obj1 = models.Host(hostname='c1.com', port=80)
    obj2 = models.Host(hostname='c2.com', port=80)
    obj3 = models.Project(title='川云二期',name='AK47',repository='git://xx.git')
    db.session.add(obj1)
    db.session.add(obj2)
    db.session.add(obj3)
    db.session.commit()
    # db.session.remove()    # 不会关闭会话

    # 在没有relationship的情况下需要先提交才能获得obj3.id
    db.session.add_all([
        models.Project2Host(host_id=obj1.id,project_id=obj3.id),
        models.Project2Host(host_id=obj2.id,project_id=obj3.id),
    ])
    db.session.commit()
    s = db.session.query(models.Project2Host.host_id).all()
    print(s)
    db.session.remove()
```

### 二、relationship的一键插入
在使用了relationship字段时插入数据代码写起来简单多了，SQLAlchemy内部自动处理多表关系

```
pro1 = models.Project(title='川云二期',name='AK47',repository='git://xx.git')
pro1.hosts = [models.Host(hostname='c1.com', port=80),models.Host(hostname='c2.com', port=80)]

db.session.add(pro1)
db.session.commit()
s = db.session.query(models.Project2Host.host_id).all()
a = db.session.query(models.Host.projects).all()
print(s,a)
db.session.remove()
```


### 三、总结
* 单表插入时没有commit不会得到id号
* 高效插入就要用到relationship
* 第一个字段class Host,第二个字段是关联的第三张表名，第三个字段为反向关联时定义的字段
* db=SQLAlchemy()封装了所有查询相关信息，session/engine/declarative_base等

