---
layout: "yandex"
page_title: "Yandex: yandex_mdb_mongodb_cluster"
sidebar_current: "docs-yandex-mdb-mongodb-cluster"
description: |-
  Manages a MongoDB cluster within Yandex.Cloud.
---

# yandex\_mdb\_mongodb\_cluster

Manages a MongoDB cluster within the Yandex.Cloud. For more information, see
[the official documentation](https://cloud.yandex.com/docs/managed-mongodb/concepts).

## Example Usage

Example of creating a Single Node MongoDB.

```hcl
resource "yandex_vpc_network" "foo" {}

resource "yandex_vpc_subnet" "foo" {
  zone           = "ru-central1-a"
  network_id     = "${yandex_vpc_network.foo.id}"
  v4_cidr_blocks = ["10.1.0.0/24"]
}

resource "yandex_mdb_mongodb_cluster" "foo" {
  name        = "test"
  environment = "PRESTABLE"
  network_id  = "${yandex_vpc_network.foo.id}"

  cluster_config {
    version = "4.2"
  }

  labels = {
    test_key = "test_value"
  }

  database {
    name = "testdb"
  }

  user {
    name     = "john"
    password = "password"
    permission {
      database_name = "testdb"
    }
  }

  resources_mongod {
    resource_preset_id = "s2.small"
    disk_size          = 16
    disk_type_id       = "network-hdd"
  }

  resources_mongos {
    resource_preset_id = "s2.small"
    disk_size          = 14
    disk_type_id       = "network-hdd"
  }

  resources_mongocfg {
    resource_preset_id = "s2.small"
    disk_size          = 14
    disk_type_id       = "network-hdd"
  }

  host {
    zone_id   = "ru-central1-a"
    subnet_id = "${yandex_vpc_subnet.foo.id}"
  }
  
  maintenance_window {
    type = "ANYTIME"
  }
}
```

## Argument Reference

The following arguments are supported:

* `name` - (Required) Name of the MongoDB cluster. Provided by the client when the cluster is created.

* `network_id` - (Required, ForceNew) ID of the network, to which the MongoDB cluster belongs.

* `environment` - (Required, ForceNew) Deployment environment of the MongoDB cluster. Can be either `PRESTABLE` or `PRODUCTION`.

* `cluster_config` - (Required) Configuration of the MongoDB subcluster. The structure is documented below.

* `user` - (Required) A user of the MongoDB cluster. The structure is documented below.

* `database` - (Required) A database of the MongoDB cluster. The structure is documented below.

* `host` - (Required) A host of the MongoDB cluster. The structure is documented below.

* `resources_mongod` - (Optional) Resources allocated to `mongod` hosts of the MongoDB cluster. The structure is documented below.

* `resources_mongocfg` - (Optional) Resources allocated to `mongocfg` hosts of the MongoDB cluster. The structure is documented below.

* `resources_mongos` - (Optional) Resources allocated to `mongos` hosts of the MongoDB cluster. The structure is documented below.

* `resources_mongoinfra` - (Optional) Resources allocated to `mongoinfra` hosts of the MongoDB cluster. The structure is documented below.

* `resources` - (**DEPRECATED**, use `resources_*` instead) Resources allocated to hosts of the MongoDB cluster. The structure is documented below.

- - -

* `description` - (Optional) Description of the MongoDB cluster.

* `labels` - (Optional) A set of key/value label pairs to assign to the MongoDB cluster.

* `folder_id` - (Optional) The ID of the folder that the resource belongs to. If it
  is not provided, the default provider folder is used.

* `security_group_ids` - (Optional) A set of ids of security groups assigned to hosts of the cluster.

* `maintenance_window` - (Optional) Maintenance window settings of the MongoDB cluster. The structure is documented below.

* `deletion_protection` - (Optional) Inhibits deletion of the cluster.  Can be either `true` or `false`.
- - -

* `restore` - (Optional, ForceNew) The cluster will be created from the specified backup. The structure is documented below.

- - -

The `cluster_config` block supports:

* `version` - (Required) Version of the MongoDB server software. Can be either `4.2`, `4.4`, `4.4-enterprise`, `5.0`, `5.0-enterprise`, `6.0` and `6.0-enterprise`.

* `feature_compatibility_version` - (Optional) Feature compatibility version of MongoDB. If not provided version is taken. Can be either `6.0`, `5.0`, `4.4` and `4.2`.

* `backup_window_start` - (Optional) Time to start the daily backup, in the UTC timezone. The structure is documented below.

* `access` - (Optional) Access policy to the MongoDB cluster. The structure is documented below.

* `mongod` - (Optional) Configuration of the mongod service. The structure is documented below.

* `mongocfg` - (Optional) Configuration of the mongocfg service. The structure is documented below.

* `mongos` - (Optional) Configuration of the mongos service. The structure is documented below.

The `backup_window_start` block supports:

* `hours` - (Optional) The hour at which backup will be started.

* `minutes` - (Optional) The minute at which backup will be started.

The `resources`, `resources_mongod`, `resources_mongos`, `resources_mongocfg`, `resources_mongoinfra`,  blocks supports:

* `resources_preset_id` - (Required) The ID of the preset for computational resources available to a MongoDB host (CPU, memory etc.). 
  For more information, see [the official documentation](https://cloud.yandex.com/docs/managed-mongodb/concepts).

* `disk_size` - (Required) Volume of the storage available to a MongoDB host, in gigabytes.

* `disk_type_id` - (Required) Type of the storage of MongoDB hosts.
  For more information see [the official documentation](https://cloud.yandex.com/docs/managed-clickhouse/concepts/storage).

The `user` block supports:

* `name` - (Required) The name of the user.

* `password` - (Required) The password of the user.

* `permission` - (Optional) Set of permissions granted to the user. The structure is documented below.

The `permission` block supports:

* `database_name` - (Required) The name of the database that the permission grants access to.

* `roles` - (Optional) The roles of the user in this database. For more information see [the official documentation](https://cloud.yandex.com/docs/managed-mongodb/concepts/users-and-roles).

The `database` block supports:

* `name` - (Required) The name of the database.

The `host` block supports:

* `name` - (Computed) The fully qualified domain name of the host. Computed on server side.

* `zone_id` - (Required) The availability zone where the MongoDB host will be created.
  For more information see [the official documentation](https://cloud.yandex.com/docs/overview/concepts/geo-scope).

* `role` - (Optional) The role of the cluster (either PRIMARY or SECONDARY).

* `health` - (Computed) The health of the host.

* `subnet_id` - (Required) The ID of the subnet, to which the host belongs. The subnet must
  be a part of the network to which the cluster belongs.
  
* `assign_public_ip` -(Optional)  Should this host have assigned public IP assigned. Can be either `true` or `false`.

* `shard_name` - (Optional) The name of the shard to which the host belongs. Only for sharded cluster.

* `type` - (Optional) type of mongo daemon which runs on this host (mongod, mongos, mongocfg, mongoinfra). Defaults to mongod.

The `access` block supports:

* `data_lens` - (Optional) Allow access for [Yandex DataLens](https://cloud.yandex.com/services/datalens).
* `data_transfer` - (Optional) Allow access for [DataTransfer](https://cloud.yandex.com/services/data-transfer)

The `maintenance_window` block supports:

* `type` - (Required) Type of maintenance window. Can be either `ANYTIME` or `WEEKLY`. A day and hour of window need to be specified with weekly window.
* `hour` - (Optional) Hour of day in UTC time zone (1-24) for maintenance window if window type is weekly.
* `day` - (Optional) Day of week for maintenance window if window type is weekly. Possible values: `MON`, `TUE`, `WED`, `THU`, `FRI`, `SAT`, `SUN`.

The `mongod` block supports:

* `security` - (Optional) A set of MongoDB Security settings
  (see the [security](https://www.mongodb.com/docs/manual/reference/configuration-options/#security-options) option).
  The structure is documented below. Available only in enterprise edition.

* `audit_log` - (Optional) A set of audit log settings 
  (see the [auditLog](https://www.mongodb.com/docs/manual/reference/configuration-options/#auditlog-options) option). 
  The structure is documented below. Available only in enterprise edition.

* `set_parameter` - (Optional) A set of MongoDB Server Parameters 
  (see the [setParameter](https://www.mongodb.com/docs/manual/reference/configuration-options/#setparameter-option) option). 
  The structure is documented below.

* `operation_profiling` - (Optional) A set of profiling settings
  (see the [operationProfiling](https://www.mongodb.com/docs/manual/reference/configuration-options/#operationprofiling-options) option). 
  The structure is documented below.

* `net` - (Optional) A set of network settings
  (see the [net](https://www.mongodb.com/docs/manual/reference/configuration-options/#net-options) option). 
  The structure is documented below.

* `storage` - (Optional) A set of storage settings
  (see the [storage](https://www.mongodb.com/docs/manual/reference/configuration-options/#storage-options) option). 
  The structure is documented below.

The `mongocfg` block supports:

* `operation_profiling` - (Optional) A set of profiling settings
  (see the [operationProfiling](https://www.mongodb.com/docs/manual/reference/configuration-options/#operationprofiling-options) option).
  The structure is documented below.

* `net` - (Optional) A set of network settings
  (see the [net](https://www.mongodb.com/docs/manual/reference/configuration-options/#net-options) option).
  The structure is documented below.

* `storage` - (Optional) A set of storage settings
  (see the [storage](https://www.mongodb.com/docs/manual/reference/configuration-options/#storage-options) option).
  The structure is documented below.

The `mongos` block supports:

* `net` - (Optional) A set of network settings
  (see the [net](https://www.mongodb.com/docs/manual/reference/configuration-options/#net-options) option).
  The structure is documented below.


The `security` block supports:

* `enable_encryption` - (Optional) Enables the encryption for the WiredTiger storage engine. Can be either true or false.
  For more information see [security.enableEncryption](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-security.enableEncryption)
  description in the official documentation. Available only in enterprise edition.

* `kmip` - (Optional) Configuration of the third party key management appliance via the Key Management Interoperability Protocol (KMIP)
  (see [Encryption tutorial](https://www.mongodb.com/docs/rapid/tutorial/configure-encryption) ). Requires `enable_encryption` to be true.
  The structure is documented below. Available only in enterprise edition.


The `audit_log` block supports:

* `filter` - (Optional) Configuration of the audit log filter in JSON format.
  For more information see [auditLog.filter](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-auditLog.filter)
  description in the official documentation. Available only in enterprise edition.

* `runtime_configuration` - (Optional) Specifies if a node allows runtime configuration of audit filters and the auditAuthorizationSuccess variable.
  For more information see [auditLog.runtimeConfiguration](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-auditLog.runtimeConfiguration)
  description in the official documentation. Available only in enterprise edition.


The `set_parameter` block supports:

* `audit_authorization_success` - (Optional) Enables the auditing of authorization successes. Can be either true or false.
  For more information, see the [auditAuthorizationSuccess](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.auditAuthorizationSuccess)
  description in the official documentation. Available only in enterprise edition.


The `operation_profiling` block supports:

* `mode` - (Optional) Specifies which operations should be profiled. The following profiler levels are available: off, slow_op, all.
  For more information, see the [operationProfiling.mode](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-operationProfiling.mode)
  description in the official documentation.

* `slow_op_threshold` - (Optional) The slow operation time threshold, in milliseconds. Operations that run for longer than this threshold are considered slow.
  For more information, see the [operationProfiling.slowOpThresholdMs](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs)
  description in the official documentation.


The `net` block supports:

* `max_incoming_connections` - (Optional) The maximum number of simultaneous connections that host will accept.
  For more information, see the [net.maxIncomingConnections](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.maxIncomingConnections)
  description in the official documentation.


The `storage` block supports:

* `wired_tiger` - (Optional) The WiredTiger engine settings.
  (see the [storage.wiredTiger](https://www.mongodb.com/docs/manual/reference/configuration-options/#storage.wiredtiger-options) option).
  These settings available only on `mongod` hosts. The structure is documented below.

* `journal` - (Optional) The durability journal to ensure data files remain valid and recoverable.
  The structure is documented below.


The `wired_tiger` block supports:

* `cache_size_gb` - (Optional) Defines the maximum size of the internal cache that WiredTiger will use for all data.
  For more information, see the [storage.wiredTiger.engineConfig.cacheSizeGB](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.wiredTiger.engineConfig.cacheSizeGB)
  description in the official documentation.

* `block_compressor` - (Optional) Specifies the default compression for collection data. You can override this on a per-collection basis when creating collections.
  Available compressors are: none, snappy, zlib, zstd. This setting available only on `mongod` hosts.
  For more information, see the [storage.wiredTiger.collectionConfig.blockCompressor](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.wiredTiger.collectionConfig.blockCompressor)
  description in the official documentation.


The `journal` block supports:

* `commit_interval` - (Optional) The maximum amount of time in milliseconds that the mongod process allows between journal operations.
  For more information, see the [storage.journal.commitIntervalMs](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.journal.commitIntervalMs)
  description in the official documentation.


The `kmip` block supports:

* `server_name` - (Required) Hostname or IP address of the KMIP server to connect to.
  For more information see [security.kmip.serverName](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-security.kmip.serverName)
  description in the official documentation.

* `port` - (Optional) Port number to use to communicate with the KMIP server. Default: 5696
  For more information see [security.kmip.port](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-security.kmip.port)
  description in the official documentation.

* `server_ca` - (Required) Path to CA File. Used for validating secure client connection to KMIP server.
  For more information see [security.kmip.serverCAFile](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-security.kmip.serverCAFile)
  description in the official documentation.

* `client_certificate` - (Required) String containing the client certificate used for authenticating MongoDB to the KMIP server.
  For more information see [security.kmip.clientCertificateFile](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-security.kmip.clientCertificateFile)
  description in the official documentation.

* `key_identifier` - (Optional) Unique KMIP identifier for an existing key within the KMIP server.
  For more information see [security.kmip.keyIdentifier](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-security.kmip.keyIdentifier)
  description in the official documentation.


The `restore` block supports:

* `backup_id` - (Required, ForceNew) Backup ID. The cluster will be created from the specified backup. [How to get a list of PostgreSQL backups](https://cloud.yandex.com/en-ru/docs/managed-mongodb/operations/cluster-backups)

* `time` - (Optional, ForceNew) Timestamp of the moment to which the MongoDB cluster should be restored. (Format: "2006-01-02T15:04:05" - UTC). When not set, current time is used.

## Attributes Reference

In addition to the arguments listed above, the following computed attributes are exported:

* `created_at` - Creation timestamp of the key.

* `health` - Aggregated health of the cluster. Can be either `ALIVE`, `DEGRADED`, `DEAD` or `HEALTH_UNKNOWN`.
  For more information see `health` field of JSON representation in [the official documentation](https://cloud.yandex.com/docs/managed-mongodb/api-ref/Cluster/).

* `status` - Status of the cluster. Can be either `CREATING`, `STARTING`, `RUNNING`, `UPDATING`, `STOPPING`, `STOPPED`, `ERROR` or `STATUS_UNKNOWN`.
  For more information see `status` field of JSON representation in [the official documentation](https://cloud.yandex.com/docs/managed-mongodb/api-ref/Cluster/).

* `cluster_id` - The ID of the cluster.

* `sharded` - MongoDB Cluster mode enabled/disabled.

## Import

A cluster can be imported using the `id` of the resource, e.g.

```
$ terraform import yandex_mdb_mongodb_cluster.foo cluster_id
```
