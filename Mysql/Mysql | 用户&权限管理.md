[TOC]

### 用户管理

#### 1、创建用户

```mysql
mysql> CREATE USER 'testuser'@'localhost' IDENTIFIED BY '123';
```

注：用户名testuser，密码123，此时用户没有任何权限，只能连接到数据库。

如果想创建一个没有密码的用户，请使用如下语句：

```mysql
mysql> CREATE USER 'testuser'@'localhost';
```

#### 2、删除用户

```mysql
mysql> drop user testuser;
```

#### 3、用户重命名

```mysql
mysql> RENAME USER 'testuser'@'localhost' TO 'newname'@'localhost';
```



### 权限管理

#### 1、GRANT命令使用说明

```mysql
mysql> grant all privileges on *.* to jack@'localhost' identified by "123456" with grant option;
```

创建一个只允许从本地登录的超级用户jack，并允许将权限赋予别的用户，密码为：123456

GRANT命令说明：

- `ON` 用来指定权限针对哪些库和表,`*.*`  中前面的`*`号用来指定数据库名，后面的`*`号用来指定表名。
- `TO` 表示将权限赋予某个用户。
- `jack@'localhost'`表示jack用户，@后面接限制的主机，可以是IP、IP段、域名以及%，**%表示任何地方**。注意：这里%有的版本不包括本地，以前碰到过给某个用户设置了%允许任何地方登录，但是在本地登录不了，这个和版本有关系，遇到这个问题再加一个localhost的用户就可以了。
- `IDENTIFIED BY` 指定用户的登录密码。
- `WITH GRANT OPTION` 这个选项表示该用户可以将自己拥有的权限授权给别人。注意：经常有人在创建操作用户的时候不指定WITH GRANT OPTION选项导致后来该用户不能使用GRANT命令创建用户或者给其它用户授权。

备注：可以使用GRANT重复给用户添加权限，权限叠加，比如你先给用户添加一个select权限，然后又给用户添加一个insert权限，那么该用户就同时拥有了select和insert权限。

#### 2、刷新权限

使用这个命令使权限生效，尤其是你对那些权限表user、db、host等做了update或者delete更新的时候。以前遇到过使用grant后权限没有更新的情况，只要对权限做了更改就使用FLUSH PRIVILEGES命令来刷新权限。

```mysql
mysql> flush privileges;
```

#### 3、查看权限

查看当前用户的权限

```mysql
mysql> show grants;
```

#### 4、回收权限

```mysql
mysql> revoke delete on *.* from 'jack'@'localhost';
```