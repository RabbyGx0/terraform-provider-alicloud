---
subcategory: "RDS"
layout: "alicloud"
page_title: "Alicloud: alicloud_rds_db_instance_endpoint_address"
sidebar_current: "docs-alicloud-resource-rds-db-instance-endpoint-address"
description: |-
  Provide RDS cluster instance endpoint public connection resources.
---

# alicloud\_rds\_db\_instance\_endpoint\_address

Provide RDS cluster instance endpoint public connection resources.

Information about RDS MySQL cluster endpoint address and how to use it, see [What is RDS MySQL cluster endpoint address](https://www.alibabacloud.com/help/en/apsaradb-for-rds/latest/api-doc-rds-2014-08-15-api-doc-createdbinstanceendpointaddress).

-> **NOTE:** Available in 1.204.0+.

## Example Usage

```
variable "creation" {
  default = "Rds"
}

variable "name" {
  default = "dbInstancevpc"
}

data "alicloud_zones" "default" {
  available_resource_creation = var.creation
}

resource "alicloud_vpc" "default" {
  vpc_name       = var.name
  cidr_block     = "172.16.0.0/16"
}

resource "alicloud_vswitch" "default" {
  vpc_id            = alicloud_vpc.default.id
  cidr_block        = "172.16.0.0/24"
  zone_id           = data.alicloud_zones.default.zones[0].id
  vswitch_name      = var.name
}

resource "alicloud_db_instance" "default" {
  engine                    = "MySQL"
  engine_version            = "8.0"
  instance_type             = "mysql.n2.medium.xc"
  instance_storage          = "20"
  instance_charge_type      = "Postpaid"
  instance_name             = var.name
  vswitch_id                = alicloud_vswitch.default.id
  db_instance_storage_type  = "cloud_essd"
  zone_id                   = data.alicloud_zones.default.ids.0
  zone_id_slave_a           = data.alicloud_zones.default.ids.0
}

resource "alicloud_rds_db_node" "default" {
  db_instance_id = alicloud_db_instance.default.id
  class_code     = alicloud_db_instance.default.instance_type
  zone_id        = alicloud_db_instance.default.zone_id
}

resource "alicloud_rds_db_instance_endpoint" "default" {
  db_instance_id                   = alicloud_db_instance.default.db_instance_id                
  vpc_id                           = alicloud_db_instance.default.vpc_id
  vswitch_id                       = alicloud_db_instance.default.vswitch_id
  connection_string_prefix         = "test001"
  port                             = "3307"
  db_instance_endpoint_description = "test111"
  node_items {
      node_id = alicloud_rds_db_node.default.node_id
      weight = 25
  }
}

resource "alicloud_rds_db_instance_endpoint_address" "default" {
  db_instance_id            = alicloud_db_instance.default.db_instance_id
  db_instance_endpoint_id   = alicloud_rds_db_instance_endpoint.default.db_instance_endpoint_id
  connection_string_prefix  = "tf-testaddress001"
  port                      = "3307"
}
```

## Argument Reference

The following arguments are supported:

* `db_instance_id` - (Required, ForceNew) The ID of the instance.
* `db_instance_endpoint_id` - (Required, ForceNew) The Endpoint ID of the instance.
* `connection_string_prefix` - (Required) The prefix of the public endpoint.
* `port` - (Required) The port number of the public endpoint.

## Attributes Reference

The following attributes are exported:

* `id` - The resource ID in terraform of endpoint public connection.The value formats as `<db_instance_id>:<db_instance_endpoint_id>`.
* `ip_address` - The IP address of the endpoint.
* `connection_string` - The endpoint of the instance.
* `ip_type` - The type of the IP address.

### Timeouts

The `timeouts` block allows you to specify [timeouts](https://www.terraform.io/docs/configuration-0-11/resources.html#timeouts) for certain actions:

* `create` - (Defaults to 30 mins) Use when opening exclusive agent (until it reaches the initial `Running` status).
* `update` - (Defaults to 30 mins) Used when updating exclusive agent (until it reaches the initial `Running` status).
* `delete` - (Defaults to 20 mins) Use when closing exclusive agent.

## Import

RDS database endpoint public address feature can be imported using the id, e.g.

```shell
$ terraform import alicloud_rds_db_instance_endpoint_address.example <db_instance_id>:<db_instance_endpoint_id>
```
