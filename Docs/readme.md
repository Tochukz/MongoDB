## MongoDB Docs  

__Enable Access Control__   
Start by adding a `userAdmin` or `userAdminAnyDatabase` role in the __admin__ database.    

```
> use admin  
> db.createUser({ user: 'myAdminUser', pwd: 'myPass', roles: [ { role: "userAdminAnyDatabase", db: "admin"}, "readWriteAnyDatabase"]})
```  
_Tip:_ From version 4.2 you can use `passwordPrompt()` method in place of `myPass`.    

In the above snippet, two roles were added - `userAdminAnyDatabase` and `readWriteAnyDatabase`.  
__NB:__ The `admin` database is the user's `authenticaton database`. The user will authenticate to this database but the user can have role in other databases.  

Restart the MongoDB instance with access control.   
But first shut it down and close the shell window.  
```
> db.adminCommand({ shutdown: 1})
```
Open a new window and  
```
> mongod --auth --port 27017 --dbpath /var/lib/mongodb
```  
If you use an INI format config file, your config file shoudl look ike this:  
```
logpath=C:\MongoDB\Server\4.2\log\mongod.log
dbpath=C:\MongoDB\Server\4.2\data
auth=true
```

Open up the mongo shell and connect using any of the 2 methods.  
__Method 1:__ Connect while luncing the `mongo` shell  
```
> mongo --port 27017 --authenticationDatabase "admin" -u "myAdminUser" -p "myPass"
```
__Method 2:__ Lunch the `mongo` shell first, and then do `db.auth()`   

```
> use admin
> db.auth('myAdminUser', 'myPass')
```  

__Create additional user__  
After conecting as an admin user  
```  
use shopDb
db.createUser({ user: "shopApp", pwd: "shopPass", roles: [ { role: "readWrite", db: "shopDb"}, {role: "read", db: "reportingDb"}]})
```    
Here we grant `readWrite` privilege to the `shopDb` and `read` privilege to the `reportingDb` for for the `shopApp` user.   
__NB:__ Here the users `authentication database` is `shopApp` so the user will authenticate to this database but the user can have toles in other databases.  

__Authnticate as the newly created `shopApp` user__  
Method 1: Authenticate why lunching the `mongo` shell  
```
> mongo --port 27017 -u "shopApp" --authenticationDatabase "shopDb" -p  
```  
Enter password for the user when prompted  

Method 2: Lunch the `mongo` shell and use `db.auth()` method  
```
> use shopDb
> db.auth('shopApp', 'shopPass')
```  
Note that we switch to the user  `authentication database` first before doins `db.auth()`  
