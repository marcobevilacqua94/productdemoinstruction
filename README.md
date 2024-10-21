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

it takes about 1 hour

on S3 there is a bucket called oldtransactions which contains 10M transactions from 2022 and can be connected to columnar


define this function to manage dates in columnar 

CREATE OR REPLACE FUNCTION iso8601(date)
{
  SELECT
  SUBSTR(date, 24, 4) || '-' || 
  CASE SUBSTR(date, 4, 3)
    WHEN 'Jan' THEN '01'
    WHEN 'Feb' THEN '02'
    WHEN 'Mar' THEN '03'
    WHEN 'Apr' THEN '04'
    WHEN 'May' THEN '05'
    WHEN 'Jun' THEN '06'
    WHEN 'Jul' THEN '07'
    WHEN 'Aug' THEN '08'
    WHEN 'Sep' THEN '09'
    WHEN 'Oct' THEN '10'
    WHEN 'Nov' THEN '11'
    WHEN 'Dec' THEN '12'
  END || '-' ||
  SUBSTR(date, 8, 2) || 'T' ||
  SUBSTR(date, 11, 8) || 'Z'
};

use it like this iso8601(t.transactionDate)[0].\`$1\`
