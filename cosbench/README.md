## Installation (Ubuntu 16.04)

### Install Java 8
```sh
apt-get update -y; apt-get install openjdk-8-headless-jre -y
```

### Install COSBench
Download latest COSBench release zip from https://github.com/intel-cloud/cosbench/releases , Unzip and extract. Make sure to `chmod` all `.sh` files as executable.
```sh
$ wget https://github.com/intel-cloud/cosbench/releases/download/v0.4.2.c4/0.4.2.c4.zip
$ unzip 0.4.2.c4.zip
$ ln -s 0.4.2.c4 cos
$ cd cos; chmod +x *.sh
```

### Setup COSBench
There two important components in COSBench one is `controller` and another is a `driver`.

- Controller node drives the benchmarks and aggregate results.
- Driver nodes which runs test driver processes. Each driver node can host multiple driver processes as configured.

Run this on each driver node. Possible to run on master node too if master node is itself of the driver nodes. Can specify [num_drivers][IP address of all drivers][base port] as optional args in `conf/driver_template.conf`. Each driver then runs at port [base_port + index].  Default driver base port is **18088**.
```sh
$ cd cos; ./start_driver.sh
```

Only to be run on master node. Its `conf/controller.conf` should have entries for each worker node in [driver&lt;n&gt;] sections. Default controller port is **19088**.

Sample `conf/controller.conf`
```
[controller]
drivers = 10
log_level = INFO
log_file = log/system.log
archive_dir = archive

[driver1]
name = driver1
url = http://147.75.80.141:18088/driver
....
....
[driver10]
name = driver1
url = http://147.75.80.151:18088/driver
```

```sh
$ cd cos; ./start_controller.sh
```

### Configuration files
`conf/controller.conf`
– Specifies how many drivers, and URLs of all driver process endpoints. If driver nodes run multiple driver processes, this should have endpoints for every process.

`conf/driver.conf`
– These are actually overwritten by `start-driver.sh`. So no point modifying these directly instead modify `conf/driver_template.conf` if you wish to run more than one driver per node.

### Start tests
Actual testing procedure is configured using a **workload configuration file**, download any one of the workload configuration available at [Cosbench workloads](https://github.com/minio/benchmarks/blob/master/cosbench)

Look for &lt;storage&gt; tag attributes.
- `endpoint` should be the Minio host:port
- `path_style_access=true` is extremely important for Minio; Otherwise, the Amazon S3 SDK library used by
   COSBench defaults to virtual host style access, tries to contact bucket URLs like http://s3testqwer1.endpoint/.
   S3 client fails with “Unknown name or service s3testqwer1.endpoint”

**Start a test**:
Pick a test such as Write test with 1024 concurrency, object size 32MiB.
```
./cli.sh submit packet-baremetal_3/write/32MB/config-1024-writeonly.xml
```
**Monitor a test**:
From a browser, open http://&lt;cosbench-controller-node:19088/controller/. Then open the item under active workload, and drill down into workload, work stage and missions by clicking on “view details”.

### Troubleshooting
COSBench provides extensive logging abilities to debug if client is misbehaving, following list contains the location of these log files for debugging.

- `log/system.log` – the controller's log.Logging level is set by log_level in [controller] section of `conf/controller.conf`. Set to DEBUG|INFO.
- `log/mission/[mission-id].log` – Actual worker logs. This is where any S3 client errors are recorded. Set “log_level” to DEBUG|INFO in [driver_n] sections of `conf/controller.conf`. Set “log_level” to DEBUG|INFO in [driver] section of `conf/driver_template.conf`.