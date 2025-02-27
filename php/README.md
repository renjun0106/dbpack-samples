# dbpack-samples for php

Simply, you can build the environment by running the `make run` command. then, you can test by 
- `curl -XPOST http://localhost:13000/v1/order/create` for transaction commit test, or
- `curl -XPOST http://localhost:13000/v1/order/create2` for transaction rollback test.

Then check whether the database data is consistent.  
Alternatively, you can manually build the environment according to the following steps.  

### Prerequisite
- php version == 7.4
- php pdo extension is installed
- when using pdo to connect to mysql server, should set `PDO::ATTR_EMULATE_PREPARES => false` to enable dpback distributed transaction

### Step0: Clone dbpack and dbpack-samples
```shell
git clone git@github.com:cectc/dbpack.git
git clone git@github.com:cectc/dbpack-samples.git
cd dbpack-samples
```

### Step1: Setup etcd

### Step2: Setup mysql, initialize the database with the following sql script
```
./scripts/order.sql
./scripts/product.sql
```

### Step3: run dbpack
```bash
vim ./configs/config-aggregation.yaml
# update distributed_transaction.etcd_config.endpoints

vim ./configs/config-product.yaml
# update data_source_cluster.dsn
# update distributed_transaction.etcd_config.endpoints

vim ./configs/config-order.yaml
# update data_source_cluster.dsn
# update distributed_transaction.etcd_config.endpoints

cd ../dbpack

# local build
make build-local
# production build
make build

./dist/dbpack start --config ../dbpack-samples/configs/config-aggregation.yaml

./dist/dbpack start --config ../dbpack-samples/configs/config-product.yaml

./dist/dbpack start --config ../dbpack-samples/configs/config-order.yaml
```

### Step4: setup aggregation_svc client
```bash
# config your apache/nginx to use php
# config your apache/nginx to /path/to/dbpack-samples/php/aggregation_svc
```

### Step5: setup order_svc client
```bash
# config your apache/nginx to use php
# config your apache/nginx to /path/to/dbpack-samples/php/order_svc
# update dsn in order_svc/svc/svc.php
```

### Step6: setup product_svc client
```bash
# config your apache/nginx to use php
# config your apache/nginx to /path/to/dbpack-samples/php/product_svc
# update dsn in product_svc/svc/svc.php
```

### Step7: access and test
```
# commit test
curl -XPOST http://localhost:13000/v1/order/create

# rollback test
curl -XPOST http://localhost:13000/v1/order/create2
```