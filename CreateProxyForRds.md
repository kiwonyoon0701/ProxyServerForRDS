This document describes how to create proxy server in front of RDS to access RDS with static ip.  
Clients are able to access RDS instance using static IP through proxy server.  
Please press 'Like' & 'Subscribe' and 'Alarm'!!! ;)

```
Server Information
===================================================================
Proxy Server : 172.31.0.254
root@ip-172-31-0-254:/root# grep . /etc/*-release
/etc/lsb-release:DISTRIB_ID=Ubuntu
/etc/lsb-release:DISTRIB_RELEASE=16.04
/etc/lsb-release:DISTRIB_CODENAME=xenial
/etc/lsb-release:DISTRIB_DESCRIPTION="Ubuntu 16.04.6 LTS"
/etc/os-release:NAME="Ubuntu"
/etc/os-release:VERSION="16.04.6 LTS (Xenial Xerus)"
/etc/os-release:ID=ubuntu
/etc/os-release:ID_LIKE=debian
/etc/os-release:PRETTY_NAME="Ubuntu 16.04.6 LTS"
/etc/os-release:VERSION_ID="16.04"
/etc/os-release:HOME_URL="http://www.ubuntu.com/"
/etc/os-release:SUPPORT_URL="http://help.ubuntu.com/"
/etc/os-release:BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
/etc/os-release:VERSION_CODENAME=xenial
/etc/os-release:UBUNTU_CODENAME=xenial

MariaDB on EC2 : 172.31.18.102
RDS MariaDB : mysql.cf89XXXXXXXX.ap-northeast-2.rds.amazonaws.com

MariaDB Client & Oracle Client & MsSQL Client : 172.31.3.220
root@ip-172-31-3-220:/root# grep . /etc/*-release
/etc/os-release:NAME="Amazon Linux AMI"
/etc/os-release:VERSION="2018.03"
/etc/os-release:ID="amzn"
/etc/os-release:ID_LIKE="rhel fedora"
/etc/os-release:VERSION_ID="2018.03"
/etc/os-release:PRETTY_NAME="Amazon Linux AMI 2018.03"
/etc/os-release:ANSI_COLOR="0;33"
/etc/os-release:CPE_NAME="cpe:/o:amazon:linux:2018.03:ga"
/etc/os-release:HOME_URL="http://aws.amazon.com/amazon-linux-ami/"
/etc/system-release:Amazon Linux AMI release 2018.03

```

**Connect to MariaDB on EC through Proxy**

```
root@ip-172-31-0-254:/etc/nginx# apt-get install nginx

root@ip-172-31-0-254:/etc/nginx# cp nginx.conf ~/backup/.

root@ip-172-31-0-254:/etc/nginx# diff ~/backup/nginx.conf ./nginx.conf
64a65,70
> stream {
> 	server {
> 		listen	3306;
> 		proxy_pass 172.31.18.102:3306;
> 	}
> }

------------------------------------------
stream {
	server {
		listen	3306;
		proxy_pass 172.31.18.102:3306;
	}
}
------------------------------------------

root@ip-172-31-0-254:/root# systemctl restart nginx.service
```

**Connection test MariaDB on EC2 through nginx Proxy**

```
root@ip-172-31-3-220:/root# mysql -uOSHOP -h172.31.0.254 -p -e "show databases";
Enter password:
+--------------------+
| Database           |
+--------------------+
| OSHOP              |
| information_schema |
| maria_log          |
| mysql              |
| performance_schema |
+--------------------+
```

**Connection Test to RDS MariaDB through proxy**

```
root@ip-172-31-0-254:/root# cat /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_disable "msie6";

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

stream {
	server {
		listen	3306;
		proxy_pass mysql.cf89XXXXXXXX.ap-northeast-2.rds.amazonaws.com:3306;
	}
}

root@ip-172-31-0-254:/root# systemctl restart nginx.service

root@ip-172-31-3-220:/root# mysql -uadmin  -h172.31.0.254 -p RDS_MARIADB -e "select * from t1;"
Enter password:
+------+
| id   |
+------+
|    1 |
+------+

### Error Case ###
root@ip-172-31-3-220:/root# mysql -uadmin  -h172.31.0.254 -p

[Error Log]
AWS RDS many connection errors; unblock with 'mysqladmin flush-hosts'

[Solution]
# Connect to RDS and execute following commands
FLUSH HOSTS;

```

**Proxy Setup for RDS Oracle**

```
root@ip-172-31-0-254:/etc/nginx# cat nginx.conf
...
stream {
	server {
		listen	1521;
		proxy_pass oracle-ee-1120424.cf89XXXXXXXX.ap-northeast-2.rds.amazonaws.com:1521;
	}
}
root@ip-172-31-0-254:/root# systemctl restart nginx.service


root@ip-172-31-3-220:/root/instantclient_19_6/network/admin# cat tnsnames.ora
rds =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = oracle-ee-1120424.cf89XXXXXXX.ap-northeast-2.rds.amazonaws.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SID=sales)
    )
  )

rds-proxy =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.31.0.254)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SID=sales)
    )
  )

root@ip-172-31-3-220:/root/instantclient_19_6/network/admin# sqlplus admin@rds
SQL> select instance_name from v$instance;

INSTANCE_NAME
----------------
SALES

SQL> create table t1 (id number, name varchar2(100));

Table created.

SQL> insert into t1 values (&id, '&name');
Enter value for id: 1
Enter value for name: kiwon
1 row created.

SQL> insert into t1 values (&id, '&name');
Enter value for id: 2
Enter value for name: john
1 row created.

SQL> commit;
Commit complete.

SQL> select * from t1;

	ID NAME
----------------------------------------------
	 1 kiwon
	 2 john


root@ip-172-31-3-220:/root# sqlplus admin@rds-proxy
Enter password:

SQL>  select instance_name from v$instance;

INSTANCE_NAME
----------------
SALES

SQL> select * from t1;

	ID NAME
----------------------------------------------
	 1 kiwon
	 2 john

```

**MSSQL on OEL**

```
root@ip-172-31-0-254:/root# cat /etc/nginx/nginx.conf
...
stream {
	server {
		listen	1433;
		proxy_pass mssql-ee-1120424.cf89XXXXXXXX.ap-northeast-2.rds.amazonaws.com:1433;
	}
}
root@ip-172-31-0-254:/root# systemctl restart nginx.service

root@ip-172-31-3-220:/root# sudo ACCEPT_EULA=Y yum install mssql-tools unixODBC-devel -y --disablerepo=amzn*

root@ip-172-31-3-220:/root# echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
root@ip-172-31-3-220:/root# echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
root@ip-172-31-3-220:/root# source ~/.bashrc

root@ip-172-31-3-220:/root# sqlcmd -S 172.31.0.254 -U admin
Password:
1> use sales;
2> select * from t1;
3> go
Changed database context to 'sales'.
id
-----------
          1
          1
          2
          2
          3
         99
         88

(7 rows affected)

1> insert into t1 values (777);
2> commit;
3> go

1> select * from t1;
2> go
id
-----------
          1
          1
          2
          2
          3
         99
         88
        777

(8 rows affected)

```

**Tomcat Connection Pool with Proxy**

```
root@ip-172-31-5-125:/var/lib/tomcat8/webapps/ROOT# cat dbCon.jsp
<%@ page language="java" import="java.sql.*" %>
<%
String DB_HOST_IP="172.31.0.254";
String DB_URL = "jdbc:oracle:thin:@" + DB_HOST_IP + ":1521:sales";
String DB_USER = "oshop";
String DB_PASSWORD = "PASSWORD";
Connection con = null;
Statement stmt = null;
ResultSet rs = null;
String sql=null;
try
 {
 Class.forName("oracle.jdbc.driver.OracleDriver");
 con = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
 out.println("Oracle Database Connected!");
 }catch(SQLException e){out.println(e);}
%>


root@ip-172-31-5-125:/var/lib/tomcat8/webapps/ROOT# cat dbconnection.jsp
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ include file="dbCon.jsp" %> <!-- dbCon.jsp import -->
<%@ page import="java.util.*,java.text.*"%>
<html>
<head>
<title>test</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link title=menustyle href="../adminstyle.css" type="text/css" rel="stylesheet">
<script language="JavaScript">
<!--
 function MM_openBrWindow(theURL,winName,features){
 window.open(theURL,winName,features);
 }
//-->
</script>
</head>


<body bgcolor="#FFFFFF" text="#000000" leftmargin="0" topmargin="0" marginwidth="0" marginheight="0">
<table width="630" border="0" cellspacing="0" cellpadding="0">
<BR>
<HR>

 <%
String ServerIP = request.getLocalAddr();
out.println("<B><U><Font size=6>Tomcat Server IP : " + ServerIP+"<BR><HR></font></U></B>");
 sql="select ENAME, JOB, MGR, HIREDATE from emp";

  stmt = con.createStatement();
  rs = stmt.executeQuery(sql);

  while(rs.next()) {
          String name=rs.getString("ENAME");
          String job=rs.getString("JOB");
          String mgr=rs.getString("MGR");
          String hiredate=rs.getString("HIREDATE");
          out.println(" ENAME : "+name+" JOB : "+job+" MRG :"+mgr+" DATE :"+hiredate+" <hr>");
 }

    if(rs != null) rs.close();
    if(stmt != null)stmt.close();
    if(con != null)con.close();
 %>

</body>
</html>


```
