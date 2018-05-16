

### 单表操作

简单单表增删改查操作
```
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine

from  models import UserInfo

engine = create_engine('mysql+pymysql://root:qwer1011@192.168.1.6:3306/db1?charset=utf8', max_overflow=0, pool_size=5)
SessionClass = sessionmaker(bind=engine)

# 每次执行数据库操作时，都需要创建一个session
session = SessionClass()

# ############# 添加 #############
obj1 = UserInfo(name="2p",email='2p@qq.com')
session.add(obj1)
# 提交事务
session.commit()

# ############# 查询 #############
result = session.query(UserInfo).all()
for row in result:
    print(row.id,row.name,row.email)

# result = session.query(UserInfo).filter(UserInfo.id > 3)
# for row in result:
#     print(row.id,row.name,row.email)

# ############# 修改 #############
effect_row_num = session.query(UserInfo).filter(UserInfo.id > 3).update({'name':'new_name',})
session.commit()
print('update',effect_row_num)

# ############# 删除 #############
effect_row_num = session.query(UserInfo).filter(UserInfo.id > 3).delete()
session.commit()
print('del',effect_row_num)


# 关闭session
session.close()
```
