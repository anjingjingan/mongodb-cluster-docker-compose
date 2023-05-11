
### 在实体服务器上部署mangodb集群
### Deploy the mangodb cluster on the entity server

<br>

#### 在需要部署的服务器拉取配置文件
#### Pull the configuration file on the server that needs to be deployed
````
ssh root@192.168.1.2
git@github.com:anjingjingan/mongodb-cluster-docker-compose.git

ssh root@192.168.1.3
git@github.com:anjingjingan/mongodb-cluster-docker-compose.git

ssh root@192.168.1.4
git@github.com:anjingjingan/mongodb-cluster-docker-compose.git
````

#### 启动容器(在每台服务器执行)
##### Start the container(Execute on each server)
````
cd mongodb-cluster-docker-compose
docker-compose up -d
````


#### 可能遇到的问题
#### possible problems
````
permissions on /var/mongo-keyfile are too open
````

#### 解决
#### Solution
````
#设置只读权限
#Set read-only permissions
chmod -R 400 key

#设置mongodb所属的权限
#Set the permissions to which mongodb belongs
chown -R 999:999 key    

#重启
#start
docker-compose up -d
````
#### 进入容器
#### into the container
````
docker exec -it mongo-cluster-1 mongosh admin
````
#### 创建副本集群
#### Create a replica cluster
````
rs.initiate({
   _id: "rs01",
   members: [
     { _id: 0, host: "192.168.1.2:27017" },
     { _id: 1, host: "192.168.1.3:27017" },
     { _id: 2, host: "192.168.1.4:27017" }
   ]
 })
````

#### 查看集群状态
#### View cluster status
````
rs.status()
````
`stateStr` 属性值为 "PRIMARY" 的是节点，以下账户在主节点设置

The `stateStr` attribute value is "PRIMARY" is the node, and the following accounts are set on the primary node
#### 创建超级管理员账户
#### Create a super administrator account
````
#pwd 设置为你需要的密码
#pwd Set to your desired password
db.createUser(
   {
     user: "admin",
     pwd: "password",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
   }
 )
````
#### 验证账户密码
#### Verify account password
````
db.auth("admin","password")
````

#### 创建只读角色
#### Create a read-only role
````
db.createRole({
     role: "readOnly",
     privileges: [
         { resource: { db: "", collection: "" }, actions: [ "find" ] }
     ],
     roles: []
 })
````

##### 创建只读账户
#### Create a read-only account
````
db.createUser({
     user: "read",
     pwd: "Pfcuo4tUMwPE78gh",
     roles: [
         { role: "readOnly", db: "admin" }
     ]
 })
````
#### admin加入clusterAdmin角色
#### admin joins the clusterAdmin role
````
db.grantRolesToUser("admin", ["clusterAdmin"])
````
#### admin 加入 clusterManager 角色
#### admin joins the clusterManager role
````
db.grantRolesToUser("admin", ["clusterManager"])
````

#### 查看 admin 用户
#### View admin users
````
db.getUser("admin")
````