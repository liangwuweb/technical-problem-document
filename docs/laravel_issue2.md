# Laravel Category (has database) page loading forever, SQLSTATE[HY000] [2002] Connection timed out

### Solution , change DB_HOST in .env file

``` javascript

DB_HOST=192.168.10.10 

//change to default ->

DB_HOST=127.0.0.1

```

