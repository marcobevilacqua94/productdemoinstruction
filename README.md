
# Product Demo Instruction

## Installation

### Installing Couchbase on EC2 (Ubuntu)
We will use an ec2 to restore a backup with the data on a Capella Cluster.
NOTICE: You do not need an ec2 if you have good internet connections. You can use cbbackupmgr directly from your machine.

Run the following commands to install Couchbase:

```bash
curl -O https://packages.couchbase.com/releases/couchbase-release/couchbase-release-1.0-noarch.deb
sudo dpkg -i ./couchbase-release-1.0-noarch.deb
sudo apt-get update
sudo apt-get install couchbase-server
```

Turn to superuser (not mandatory, but I had issues with permissions using cbbackupmgr and this fixes them):

```bash
sudo su -
```

Then export AWS credentials (via export on Linux).

## Backup Operations

We have to restore a backup saved on S3 to get the data we will work on.

### Create Backup Repository (NOT NEEDED FOR DEMO - JUST INSTRUCTIONS TO BUILD BACK THE REPO IN EMERGENCY SITUATIONS)

The repository setup is required only for the first backup:

```bash
/opt/couchbase/bin/cbbackupmgr config -a s3://demobackupcouchbase/archive --r demo --obj-staging-dir /staging --capella --obj-region us-east-1
```

### Create Backup (NOT NEEDED FOR DEMO - JUST INSTRUCTIONS TO BUILD BACK THE BACKUP IN EMERGENCY SITUATIONS)

```bash
/opt/couchbase/bin/cbbackupmgr backup --archive s3://demobackupcouchbase/archive --repo demo --obj-staging-dir /staging -c couchbases://cb.kwh9supmzlotmb.cloud.couchbase.com -u app -p Couchbase123! --full-backup --threads 2 --obj-region us-east-1
```

## Restore Operations (NEEDED FOR DEMO)

To restore, create a Capella cluster with all services, allow access from anywhere, create the user `app` with password `Couchbase123!`, and create the bucket `productDemo`:

```bash
/opt/couchbase/bin/cbbackupmgr restore --archive s3://demobackupcouchbase/archive --repo demo -c couchbases://cb.hiostdibesgjvau2.cloud.couchbase.com --obj-region us-east-1 -u app -p Couchbase123! --obj-staging-dir /staging
```

**Note:** It takes about one hourm, Analytics or Columnar links must be created manually.

## What is in the cluster 
### This is the simple data model:

### users - 3M
```json
"address": "Suite 290 0406 Kemmer Grove, Balistreriborough, WY 22556-8531",
"gender": "Female",
"age": 24,
"username": "murray.strosin@gmail.com"
```

### products - 1M
```json
"price": 28.22,
"productName": "Incredible Aluminum Bottle",
"stocks": [
  {
    "warehouseId": 2862,
    "stockQuantity": 743
  },
  {
    "warehouseId": 5869,
    "stockQuantity": 611
  },
  {
    "warehouseId": 4202,
    "stockQuantity": 55
  },
  {
    "warehouseId": 3579,
    "stockQuantity": 900
  },
  {
    "warehouseId": 396,
    "stockQuantity": 933
  }
],
"supplier": "Mills-Hoeger"
```

### ratings - 20M
```json
"rating": 5,
"ratingDate": "2024-06-22T07:26:57Z",
"productId": 216738,
"userId": 2845652
```

### transactions - 25M
```json
"paymentMethod": "PayPal",
"transactionDate": "2023-12-07T21:40:22Z",
"purchases": [
  {
    "productId": 805036,
    "quantityPurchased": 5,
    "discountApplied": 0
  },
  {
    "productId": 126457,
    "quantityPurchased": 1,
    "discountApplied": 0
  },
  {
    "productId": 900152,
    "quantityPurchased": 2,
    "discountApplied": 11
  }
],
"userId": 224644,
"device": "Desktop",
"shippingMethod": "Express"
```

### warehouses - 10K
```json
"owner": "Kerry Stroman",
"address": "Apt. 331 678 Keisha Village, Venaberg, UT 05695",
"phone": "1-307-706-5338 x44299",
"email": "gretta.dickens@example.com"
```
### Indexes 
These are the indexes present on the cluster. You need to build them by clicking on the little arrow in the index ui.
The name of the index is autoexplicative on the indexed fields.  

- adv_ratings_by_uesrId_and_productId  
- adv_transactions_by_userId  
- adv_users_by_uesrname  
- adv_warehouses_by_email

### Search Index
The is a search index on the products collection indexing averageRating, numberOfRatings, price and productName fields.  

- productSearch  

You find test queries later in the Operational and Search Queries Part

### Eventing
There are three eventing functions defined in the cluster. Do not redeploy them as they have already done their job but you can show them
- dateFormatterRating turns the ratings document date into iso8601 format
```javascript
function OnUpdate(doc, meta) {
    doc["ratingDate"] = iso8601(doc["ratingDate"])
    ratings[meta.id] = doc
    log("Doc date updated", meta.id);
}


function iso8601(date) {
    const months = {
        Jan: '01', Feb: '02', Mar: '03', Apr: '04', May: '05', Jun: '06',
        Jul: '07', Aug: '08', Sep: '09', Oct: '10', Nov: '11', Dec: '12'
    };

    const year = date.substr(24, 4);
    const month = months[date.substr(4, 3)];
    const day = date.substr(8, 2);
    const time = date.substr(11, 8);

    return `${year}-${month}-${day}T${time}Z`;
}
```
- dateFormatteTransaction turns the transactions document date into iso8601 format
```javascript
function OnUpdate(doc, meta) {
    doc["ratingDate"] = iso8601(doc["ratingDate"])
    ratings[meta.id] = doc
    log("Doc date updated", meta.id);
}


function iso8601(date) {
    const months = {
        Jan: '01', Feb: '02', Mar: '03', Apr: '04', May: '05', Jun: '06',
        Jul: '07', Aug: '08', Sep: '09', Oct: '10', Nov: '11', Dec: '12'
    };

    const year = date.substr(24, 4);
    const month = months[date.substr(4, 3)];
    const day = date.substr(8, 2);
    const time = date.substr(11, 8);

    return `${year}-${month}-${day}T${time}Z`;
}
```
- updateRatingAverageOnProduct mantains average rating and number of rating info on products documents
```javascript
function OnUpdate(doc, meta) {
    doc["ratingDate"] = iso8601(doc["ratingDate"])
    ratings[meta.id] = doc
    log("Doc date updated", meta.id);
}


function iso8601(date) {
    const months = {
        Jan: '01', Feb: '02', Mar: '03', Apr: '04', May: '05', Jun: '06',
        Jul: '07', Aug: '08', Sep: '09', Oct: '10', Nov: '11', Dec: '12'
    };

    const year = date.substr(24, 4);
    const month = months[date.substr(4, 3)];
    const day = date.substr(8, 2);
    const time = date.substr(11, 8);

    return `${year}-${month}-${day}T${time}Z`;
}
```
  
## Load Testing

To simulate load on the cluster:

```bash
/opt/couchbase/bin/cbc-pillowfight --spec couchbases://cb.zy7dcemerlfgdi1u.cloud.couchbase.com/productDemo --username app --password Couchbase123! --collection productDemo.products -B 4
```

- Add `-t 4` for more load.
- Add `-B 1` for less load.

## Operational and Search Queries

Create a text file `queries.txt` containing the following:

```json
{"statement":"select w.owner, w.phone from productDemo.productDemo.warehouses w where email = 'cecila.schaefer@example.com'"}
{"statement":"select w.owner, w.phone from productDemo.productDemo.warehouses w where email = 'alecia.bode@example.com'"}
{"statement":"select w.owner, w.phone from productDemo.productDemo.warehouses w where email = 'jacinda.schneider@example.com'"}
{"statement":"select meta(u).id, u.age, u.address, u.username from productDemo.productDemo.users u where u.username = 'asuncion.abbott@hotmail.com'"}
{"statement":"select meta(u).id, u.age, u.address, u.username from productDemo.productDemo.users u where u.username = 'babette.blanda@hotmail.com'"}
{"statement":"select meta(u).id, u.age, u.address, u.username from productDemo.productDemo.users u where u.username = 'alecia.abernathy@hotmail.com'"}
{"statement":"select * from productDemo.productDemo.transactions t where t.userId = 442745"}
{"statement":"select t.purchases from productDemo.productDemo.transactions t where t.userId = 400545"}
{"statement":"select t.device, t.paymentMethod from productDemo.productDemo.transactions t where t.userId = 114"}
{"statement":"select * from productDemo.productDemo.ratings r where r.userId = 1140"}
{"statement":"select r.rating, r.ratingDate from productDemo.productDemo.ratings r where r.userId = 150 and r.productId = 288494"}
{"statement":"select r.rating, r.userId from productDemo.productDemo.ratings r where r.userId = 1541 and r.productId = 111742"}
{"statement":"select * from productDemo.productDemo.products p where SEARCH(p.productName, 'knife')"}
{"statement":"select p.productName from productDemo.productDemo.products p where SEARCH(p.productName, 'marble')"}
{"statement":"select p.productName, ROUND(p.averageRating, 2) as averageRating from productDemo.productDemo.products p where SEARCH(p.productName, 'practical')"}
{"statement":"select p.productName, ROUND(p.averageRating, 2) as averageRating from productDemo.productDemo.products p USE INDEX(USING FTS) where SEARCH(p.productName, 'table') and p.averageRating > 3 ORDER BY p.averageRating DESC LIMIT 100"}
{"statement":"select p.productName, p.price, ROUND(p.averageRating, 2) as averageRating from productDemo.productDemo.products p USE INDEX(USING FTS) where SEARCH(p.productName, 'table') and p.averageRating > 3 and p.price < 50 ORDER BY p.price DESC LIMIT 100"}
```
Last 3 queries leverage the search index we have.
Execute queries:

```bash
/opt/couchbase/bin/cbc-n1qlback -f queries.txt -u app -P Couchbase123! --spec couchbases://cb.zy7dcemerlfgdi1u.cloud.couchbase.com/productDemo -t 5
```

## Analytics Queries

### Monthly Sales in 2024

```sql
SELECT FLOOR(SUM(p.price * t_p.quantityPurchased * discount)) AS total_price_value, month
FROM transactions t, t.purchases t_p
JOIN products p on TOSTRING(t_p.productId) = meta(p).id
LET month = DATE_PART_STR(t.transactionDate, 'month'), discount = 100 - t_p.discountApplied
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2024
GROUP BY month
ORDER BY month ASC;
```

### Transactions in 2023 by Gender and Age Group

```sql
SELECT COUNT(t) as transactions, u.gender, ageGroup
FROM transactions t
JOIN users u on TOSTRING(t.userId) = meta(u).id
LET ageGroup = FLOOR(u.age/5) * 5
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023
GROUP BY u.gender, ageGroup
ORDER BY ageGroup, u.gender DESC;
```

### 100 Most Rated Products of 2023

```sql
SELECT p.productName, COUNT(r) as ratings, ROUND(AVG(r.rating), 3) as average
FROM products p
JOIN ratings r on TOSTRING(r.productId) = meta(p).id
WHERE DATE_PART_STR(r.ratingDate, 'year') = 2023
GROUP BY p.productName
ORDER BY COUNT(r) DESC
LIMIT 100;
```

### 100 Worst Rated Products of 2023

```sql
SELECT p.productName, COUNT(r) as ratings, ROUND(AVG(r.rating), 3) as average
FROM products p
JOIN ratings r on TOSTRING(r.productId) = meta(p).id
WHERE DATE_PART_STR(r.ratingDate, 'year') = 2023
GROUP BY p.productName
ORDER BY average ASC
LIMIT 100

```

### 1000 Most Bought Products of 2023

```sql
SELECT p.productName, COUNT(p) as purchases
FROM transactions t, t.purchases tp
JOIN products p on TOSTRING(tp.productId) = meta(p).id
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023
GROUP BY p.productName
ORDER BY purchases DESC
LIMIT 1000

```

### Most Purchased Products of 2023 by Age Group

```sql
WITH cte AS (
SELECT p.productName, COUNT(p) as purchases, ageGroup, ROW_NUMBER() OVER (PARTITION BY ageGroup ORDER BY COUNT(p) DESC) rn
FROM transactions t, t.purchases tp
JOIN products p on TOSTRING(tp.productId) = meta(p).id
JOIN users u ON TOSTRING(t.userId) = meta(u).id
LET ageGroup = FLOOR(u.age / 5) * 5
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023 
GROUP BY p.productName, ageGroup
ORDER BY ageGroup ASC
)

SELECT productName, purchases, ageGroup
FROM cte
WHERE rn = 1

```

## Analytics Tabular Views

### Sales Volume by Month in 2024

```sql
CREATE ANALYTICS VIEW sales_by_month_2024(
    total_price_value BIGINT,
    month STRING
)
DEFAULT NULL
AS
SELECT FLOOR(SUM(p.price * t_p.quantityPurchased * discount)) AS total_price_value, month
FROM transactions t, t.purchases t_p
JOIN products p on TOSTRING(t_p.productId) = meta(p).id
LET month = DATE_PART_STR(t.transactionDate, 'month'), discount = 100 - t_p.discountApplied
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2024
GROUP BY month
ORDER BY month ASC

```

### Number of Transactions by Gender and Age Group 2023

```sql
CREATE ANALYTICS VIEW demographics_2023 (
    transactions BIGINT,
    gender STRING,
    ageGroup STRING
)
DEFAULT NULL
AS
SELECT COUNT(t) as transactions, u.gender, ageGroup
FROM transactions t
JOIN users u on TOSTRING(t.userId) = meta(u).id
LET ageGroup = FLOOR(u.age/5) * 5
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023
GROUP BY u.gender, ageGroup
ORDER BY ageGroup, u.gender DESC

```

### Most Bought 1000 products of 2023

```sql
CREATE ANALYTICS VIEW most_bought_2023 (
    productName STRING,
    purchases BIGINT
)
DEFAULT NULL
AS
SELECT p.productName, COUNT(p) as purchases
FROM transactions t, t.purchases tp
JOIN products p on TOSTRING(tp.productId) = meta(p).id
WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023
GROUP BY p.productName
ORDER BY purchases DESC
LIMIT 1000

```

## Columnar Write Back to S3

Set up a Columnar cluster and link all the collections. You can also create views and connect your BI tools directly to Columnar.

### Write Back Sales Volumnes by Month 2024

```sql
COPY (
    SELECT FLOOR(SUM(p.price * t_p.quantityPurchased * discount)) AS total_price_value, month
    FROM transactions t, t.purchases t_p
    JOIN products p on TOSTRING(t_p.productId) = meta(p).id
    LET month = DATE_PART_STR(t.transactionDate, 'month'), discount = 100 - t_p.discountApplied
    WHERE DATE_PART_STR(t.transactionDate, 'year') = 2024
    GROUP BY month
    ORDER BY month ASC
) TO columnardemomarco AT s3Link PATH("");
```

### Write Back Partitioned by Month Transactions 2023

```sql
COPY (
    SELECT t.*
    FROM transactions t
    WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023
) TO columnardemomarco AT s3Link
PATH("2023/month", month)
OVER (
    PARTITION BY TOSTRING(DATE_PART_STR(t.transactionDate, 'month')) AS month
) WITH {
    "compression": "gzip"
};
```

### Reading Back Data Partitioned by Month Transactions 2023

If you have the collection link on the S3 bucket, you can read the transactions:

```sql
SELECT * FROM s3oldtransactions LIMIT 100;
```
