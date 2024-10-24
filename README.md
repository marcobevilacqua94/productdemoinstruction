# productdemoinstruction

creation of backup repository

if you have bad network install on ec2 otherwise can do it locally

install couchbase on ec2 ubuntu, region north virginia

curl -O https://packages.couchbase.com/releases/couchbase-release/couchbase-release-1.0-noarch.deb

sudo dpkg -i ./couchbase-release-1.0-noarch.deb

sudo apt-get update

sudo apt-get install couchbase-server

need to have archive folder in the bucket
do it from an ec2 for reduced latency

you need to export aws credentials

sudo su -

CREATE BACKUP REPOSITORY

set up repository line, needed only with first backup

/opt/couchbase/bin/cbbackupmgr config -a s3://demobackupcouchbase/archive --r demo --obj-staging-dir /staging --capella --obj-region us-east-1

CREATE BACKUP 

/opt/couchbase/bin/cbbackupmgr backup --archive s3://demobackupcouchbase/archive --repo demo --obj-staging-dir /staging -c couchbases://cb.kwh9supmzlotmb.cloud.couchbase.com -u app -p Couchbase123! --full-backup --threads 2 --obj-region us-east-1

RESTORE BACKUP 

To restore create a Capella cluster with all services, allow access from anywhere, create user app Couchbase123!, create bucket productDemo

/opt/couchbase/bin/cbbackupmgr restore --archive s3://demobackupcouchbase/archive --repo demo -c couchbases://cb.hiostdibesgjvau2.cloud.couchbase.com  --obj-region us-east-1 -u app -p Couchbase123! --obj-staging-dir /staging

analytics or columnar links must be created manually

there are eventing functions to show customers

it takes about 1 hour

PUT CLUSTER UNDER LOAD

 /opt/couchbase/bin/cbc-pillowfight --spec couchbases://cb.zy7dcemerlfgdi1u.cloud.couchbase.com/productDemo --username app --password Couchbase123! --collection productDemo.products

 add -t 4 for more load

 add -B 1 for less load

OPERATIONAL AND SEARCH QUERIES 

create text file queries.txt

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

{"statement":"select r.rating, r.userId from productDemo.productDemo.ratings r where r.userId is not missing and r.productId = 10"}

{"statement":"select * from productDemo.productDemo.products p where SEARCH(p.productName, 'knife')"}

{"statement":"select p.productName from productDemo.productDemo.products p where SEARCH(p.productName, 'marble')"}

{"statement":"select p.productName, ROUND(p.averageRating, 2) as averageRating from productDemo.productDemo.products p where SEARCH(p.productName, 'practical')"}

/opt/couchbase/bin/cbc-n1qlback -f queries.txt -u app -P Couchbase123! --spec couchbases://cb.zy7dcemerlfgdi1u.cloud.couchbase.com/productDemo -t 15

ANALYTICS QUERIES

GET aggregated by month values of all items sold 2024 

SELECT FLOOR(SUM(p.price * t_p.quantityPurchased)) AS total_price_value, month

FROM transactions t, t.purchases t_p

JOIN products p on TOSTRING(t_p.productId) = meta(p).id

LET month = DATE_PART_STR(t.transactionDate, 'month')

WHERE DATE_PART_STR(t.transactionDate, 'year') = 2024

GROUP BY month

ORDER BY month ASC


GET number of transactions made in 2023 per gender and agegroup

SELECT COUNT(t) as transactions, u.gender, ageGroup

FROM transactions t

JOIN users u on TOSTRING(t.userId) = meta(u).id

LET ageGroup = FLOOR(u.age/5) * 5

WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023

GROUP BY u.gender, ageGroup

ORDER BY ageGroup, u.gender DESC


GET 100 most rated products of 2023 

SELECT p.productName, COUNT(r) as ratings, ROUND(AVG(r.rating), 3) as average

FROM products p

JOIN ratings r on TOSTRING(r.productId) = meta(p).id

WHERE DATE_PART_STR(r.ratingDate, 'year') = 2023

GROUP BY p.productName

ORDER BY COUNT(r) DESC

LIMIT 100


GET 100 worst rated products of 2023

SELECT p.productName, COUNT(r) as ratings, ROUND(AVG(r.rating), 3) as average

FROM products p

JOIN ratings r on TOSTRING(r.productId) = meta(p).id

WHERE DATE_PART_STR(r.ratingDate, 'year') = 2023

GROUP BY p.productName

ORDER BY average ASC

LIMIT 100

GET 1000 most bought products of 2023

SELECT p.productName, COUNT(p) as purchases

FROM transactions t, t.purchases tp

JOIN products p on TOSTRING(tp.productId) = meta(p).id

WHERE DATE_PART_STR(t.transactionDate, 'year') = 2023

GROUP BY p.productName

ORDER BY purchases DESC

LIMIT 1000

GET most purchased product of 2023 for ageGroup

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


