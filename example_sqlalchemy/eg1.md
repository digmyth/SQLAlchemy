# example_sqlalchemy 完整示例学习

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

    # hosts = relationship('Host',secondary='Project2Host',backref='Project')

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

### 一、普通单表插入数据
在没有使用relationship字段时直接插入表数据可能写起来复杂些，主要复杂在手动处理多表关系


### 二、relationship的一键插入
在使用了relationship字段时插入数据代码写起来简单多了，SQLAlchemy内部自动处理多表关系


### 三、总结
*
*
*

