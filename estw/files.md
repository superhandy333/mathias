<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](estw.md) / Files</small>

# Files

- [create_database.sql](create_database.md)
- [create_user.sql](#create_usersql)
- [export.bat](#exportbat)
- [export.sql](export.md)
- [transfer.sql](#transfersql)

## create_user.sql

~~~sql
CREATE USER 'estw'@'192.168.%' IDENTIFIED BY 'estw';
GRANT SELECT  ON *.* TO 'estw'@'192.168.%';
GRANT FILE  ON *.* TO 'estw'@'192.168.%';
FLUSH PRIVILEGES;
~~~

## export.bat

~~~
mysqlsh --uri estw@192.168.xxx.xx:3300/estw --sql -f export.sql
~~~

## transfer.sql

Beispiel. Das Original befindet sich im [Repository](https://github.com/superhandy333).

~~~sql
scp papa@192.168.xxx.xx:/home/papa/docker/export/*.csv d:\rentsch\estwdaten
~~~

<hr>
Autor<br>

Mathias Rentsch<br>
rentsch@online.de<br>
Februar 2026
