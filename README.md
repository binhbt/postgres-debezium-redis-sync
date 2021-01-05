## Caching Update with Debezium

This demonstration shows how you can use Debezium to create, update, and delete data cached in Redis.


```
+------------+      +-----------+       +---------+
|            |      |           |       |         |
| PostgreSQL | ---> | Debdezium |  ---> |  Redis  |
|            |      |           |       |         |
+------------+      +-----------+       +---------+
```

**Prerequisites**

* Docker
   * To run Postgresl & Redis
* Java Developer Kit (8 or later)
* Apache Maven

### Running the demonstration

1. Get the source code from the repository

    ```
    > docker-compose up -d
    ```

1. Start Redis

    The Java Spring application use Jedis to connect to Redis. The connection string is located in the `application.properties` file.

    Open a new terminal and run the command:

    ```
    > docker run -it --rm --name redis -p 6379:6379 redis
    ```



1. Connect to PostgreSQL & create tables
    
    Use the Docker container to connect to PostgreSQL.

    ```
    > docker exec -it postgresql-db bash
    ```

    Connect to PostgreSQL
    ```
    bash-5.0# psql -U postgres
    ```

    Create table and insert rows
    
    ```sql
    CREATE TABLE customers (
        customer_id serial PRIMARY KEY,
        username VARCHAR ( 50 ) UNIQUE NOT NULL,
        first_name VARCHAR ( 50 )  NOT NULL,
        last_name VARCHAR ( 50 )  NOT NULL,
        email VARCHAR ( 255 ) UNIQUE NOT NULL
    );


    CREATE TABLE support_level (
        level VARCHAR (30)  UNIQUE NOT NULL
    );

    CREATE TABLE orders (
        cust_id INT NOT NULL,
        order_type VARCHAR (10) NOT NULL,
        order_date DATE DEFAULT CURRENT_DATE,
        comments TEXT,
        PRIMARY KEY (cust_id, order_type)
    );

    ```

    You can check that tables are created using the command:

    ```sql
    \dt
    ```

1. Insert rows into tables

    ```sql 
    INSERT INTO customers
    (username, first_name, last_name, email)
    VALUES
    ('jdoe1', 'John', 'Doe', 'jdoe1@demo.com');
    ```


1. Start the Debezium server

    Open your browser and go to:

    http://localhost:8082/start



1. Insert new rows into tables

    ```sql 
    INSERT INTO customers
    (username, first_name, last_name, email)
    VALUES
    ('jdoe2', 'John', 'Doe', 'jdoe2@demo.com');
    ```


    ```sql 
    INSERT INTO orders
    (cust_id, order_type, comments)
    VALUES(1, 'ONLINE', 'comment for first order' );
    ```

    ```sql
    INSERT INTO support_level
    (level)
    VALUES
    ('GOLD');
    ```
        ```sql
    select * from customers;
    ```
    This last row is not synchronized to Redis since the `support_level` table is not in the `table.include.list`. (`application.properties`)  

    Check with REDIS  
            ```
    > docker exec -it redis bash
    > redis-cli HGETALL customers:customer_id:1
    > redis-cli HGETALL orders:cust_id:1:order_type:ONLINE
    ```

Note: When you restart the Debezium service, you need to send a transaction to start the replication, so be sure you reset the service using `http://localhost:8082/reset` if you have anty issue. 