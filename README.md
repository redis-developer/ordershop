# OrderShop 

Building an Event Store using Redis.


## Architecture


The following diagram shows the inter-connectivity of nine decoupled microservices that use an event store built with Redis Streams for inter-services communication. They do this by listening to any newly created events on the specific event stream in an event store, i.e. a Redis instance.

![image](https://user-images.githubusercontent.com/313480/151544356-a716df28-2798-49fd-b46d-7ac715d04e4e.png)


## A Quick Look at OrderShop Domain Model

The domain model for our OrderShop application consists of the following five entities:

- Customer
- Product
- Inventory
- Order
- Billing

By listening to the domain events and keeping the entity cache up to date, the aggregate functions of the event store has to be called only once or on reply.

![image](https://user-images.githubusercontent.com/313480/151543255-80c8273a-93e6-492d-b593-a7b9e1cd92e1.png)

## Under the hood

Below are some sample test cases from [client.py](https://github.com/redis-developer/ordershop/blob/master/client/client.py), along with corresponding Redis data types and keys.

![image](https://user-images.githubusercontent.com/313480/151546605-2b01b1d2-22c5-4d7d-a011-4ca26a0a9616.png)

I chose the Streams data type to save these events because the abstract data type behind them is a transaction log, which perfectly fits our use case of a continuous event stream. I chose different keys to distribute the partitions and decided to generate my own entry ID for each stream, consisting of the timestamp in seconds “-” microseconds (to be unique and preserve the order of the events across keys/partitions).

```
127.0.0.1:6379> XINFO STREAM events:order_created
 1) “length”
 2) (integer) 10
 3) “radix-tree-keys”
 4) (integer) 1
 5) “radix-tree-nodes”
 6) (integer) 2
 7) “groups”
 8) (integer) 0
 9) “last-generated-id”
10) “1548699679211-658”
11) “first-entry”
12) 1) “1548699678802-91”
    2) 1) “event_id”
       2) “fdd528d9-d469-42c1-be95-8ce2b2edbd63”
       3) “entity”
       4) “{\”id\”: \”b7663295-b973-42dc-b7bf-8e488e829d10\”, \”product_ids\”:
[\”7380449c-d4ed-41b8-9b6d-73805b944939\”, \”d3c32e76-c175-4037-ade3-ec6b76c8045d\”,
\”7380449c-d4ed-41b8-9b6d-73805b944939\”, \”93be6597-19d2-464e-882a-e4920154ba0e\”,
\”2093893d-53e9-4d97-bbf8-8a943ba5afde\”, \”7380449c-d4ed-41b8-9b6d-73805b944939\”],
\”customer_id\”: \”63a95f27-42c5-4aa8-9e40-1b59b0626756\”}”
13) “last-entry”
14) 1) “1548699679211-658”
    2) 1) “event_id”
       2) “164f9f4e-bfd7-4aaf-8717-70fc0c7b3647”
       3) “entity”
       4) “{\”id\”: \”1ea7f394-e9e9-4b02-8c29-547f8bcd2dde\”, \”product_ids\”:
[\”2093893d-53e9-4d97-bbf8-8a943ba5afde\”], \”customer_id\”:
\”8e8471c7-2f48-4e45-87ac-3c840cb63e60\”}”
```

I choose Sets to store the IDs (UUIDs) and Lists and Hashes to model the data, since it reflects their structure and the entity cache is just a simple projection of the domain model.

```
127.0.0.1:6379> TYPE customer_ids
set

127.0.0.1:6379> SMEMBERS customer_ids
1) “3b1c09fa-2feb-4c73-9e85-06131ec2548f”
2) “47c33e78-5e50-4f0f-8048-dd33efff777e”
3) “8bedc5f3-98f0-4623-8aba-4a477c1dd1d2”
4) “5f12bda4-be4d-48d4-bc42-e9d9d37881ed”
5) “aceb5838-e21b-4cc3-b59c-aefae5389335”
6) “63a95f27-42c5-4aa8-9e40-1b59b0626756”
7) “8e8471c7-2f48-4e45-87ac-3c840cb63e60”
8) “fe897703-826b-49ba-b000-27ba5da20505”
9) “67ded96e-a4b4-404e-ace6-3b8f4dea4038”

127.0.0.1:6379> TYPE customer_entity:67ded96e-a4b4-404e-ace6-3b8f4dea4038
hash

127.0.0.1:6379> HVALS customer_entity:67ded96e-a4b4-404e-ace6-3b8f4dea4038
1) “67ded96e-a4b4-404e-ace6-3b8f4dea4038”
2) “Ximnezmdmb”
3) “ximnezmdmb@server.com”
```


When a customer, an inventory item or an order is created/deleted, an event gets  communicated asynchronously to the CRM service using RESP to manage OrderShop’s interactions with current and potential customers. The CRM service gets started and stopped during runtime without any impact to other microservices. This necessitates that all messages sent to it during its downtime be captured for processing.





## Getting Started

## Prerequisite:

- Docker
- Docker Compose
- Python 3.6

### Step 1. Clone the repository

```
git clone https://github.com/redis-developer/ordershop
```

### Step 2. Bring up the application

```
docker-compose up
```

## Step 3. Install the dependencies 

```
pip3 install -r client/requirements.txt
```

## Step 4. Execute the client

```
python3 -m unittest client/client.py
```

## Step 5. Testng

```
python3 -m unittest client/client.py
```

## Step 6. Bringing down the app

```
docker-compose down
```

If you want to stop the CRM-service

```
docker-compose stop crm-service
```

Re-execute the client and you’ll see that the application functions w/o any error

