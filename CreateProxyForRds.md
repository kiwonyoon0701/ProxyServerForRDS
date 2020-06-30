
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

---
stream {
	server {
		listen	3306;
		proxy_pass 172.31.18.102:3306;
	}
}
---

root@ip-172-31-0-254:/etc/nginx# service nginx restart

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



RDS

[Error]
AWS RDS many connection errors; unblock with 'mysqladmin flush-hosts'
[Solution]
FLUSH HOSTS;


root@ip-172-31-3-220:/root# mysql -uadmin  -h172.31.0.254 -p



**Oracle RDS**
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
		#listen	3306;
		#proxy_pass mysql.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:3306;
		listen	1521;
		proxy_pass oracle-ee-1120424.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:1521;
	}
}

root@ip-172-31-0-254:/root# service nginx restart

root@ip-172-31-3-220:/root/instantclient_19_6/network/admin# cat tnsnames.ora
rds =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = oracle-ee-1120424.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com)(PORT = 1521))
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

root@ip-172-31-3-220:/root# sqlplus admin@rds

SQL>

root@ip-172-31-3-220:/root# sqlplus admin@rds-proxy

SQL>


**MSSQL on OEL**
stream {
	server {
		#listen	3306;
		#proxy_pass mysql.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:3306;
		#listen	1521;
		#proxy_pass oracle-ee-1120424.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:1521;
		listen	1433;
		proxy_pass mssql-ee.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:1433;
	}
}

root@ip-172-31-0-254:/root# service nginx restart

C:\Users\Administrator>sqlcmd -S 172.31.0.254 -U admin
Password: Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login failed for user 'admin'..

C:\Users\Administrator>sqlcmd -S 172.31.0.254 -U admin
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

(5 rows affected)