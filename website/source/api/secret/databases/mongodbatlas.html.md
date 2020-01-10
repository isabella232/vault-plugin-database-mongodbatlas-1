---
layout: "api"
page_title: "MongoDB Atlas - Database - Secrets Engines - HTTP API"
sidebar_title: "MongoDB Atlas"
sidebar_current: "api-http-secret-databases-mongodbatlas"
description: |-
  The MongoDB Atlas plugin for Vault's Database Secrets Engine generates MongoDB Database User credentials for MongoDB Atlas.
---

# MongoDB Atlas Database Plugin HTTP API

The MongoDB Atlas plugin is one of the supported plugins for the Database
Secrets Engine. This plugin generates MongoDB Atlas Database User credentials dynamically based on
configured roles.

  ~> **Notice:** The following will be accurate after review and approval by Hashicorp, which is in
    progress. Until then follow the instructions in the [README developing section](./../../../../../README.md).


## Configure Connection

In addition to the parameters defined by the [Database
Backend](/api/secret/databases/index.html#configure-connection), this plugin
has a number of parameters to further configure a connection.

| Method   | Path                         |
| :--------------------------- | :--------------------- |
| `POST`   | `/database/config/:name`     |

### Parameters

- `public_key` `(string: <required>)` – The Public Programmatic API Key used to authenticate with the MongoDB Atlas API.
- `private_key` `(string: <required>)` - The Private Programmatic API Key used to connect with MongoDB Atlas API.
- `project_id` `(string: <required>)` - The [Project ID](https://docs.atlas.mongodb.com/api/#group-id) the Database User should be created within.

### Sample Payload

```json
{
  "plugin_name": "mongodbatlas-database-plugin",
  "allowed_roles": "readonly",
  "public_key": "aPublicKey",
  "private_key": "aPrivateKey",
  "project_id": "aProjectID",
}
```

### Sample Request

```
$ curl \
    --header "X-Vault-Token: ..." \
    --request POST \
    --data @payload.json \
    http://127.0.0.1:8200/v1/database/config/mongodbatlas
```

## Statements

Statements are configured during Vault role creation and are used by the plugin to
determine what is sent to MongoDB Atlas upon user creation, renewal, and
revocation. For more information on configuring roles see the [Role API](/api/secret/databases/index.html#create-role)
in the Database Secrets Engine docs.

### Parameters

The following are the statements used by this plugin. If not mentioned in this
list the plugin does not support that statement type.

- `creation_statements` `(string: <required>)` – Specifies the database
  statements executed to create and configure a user. Must be a
  serialized JSON object, or a base64-encoded serialized JSON object.
  The object can optionally contain a "database_name", the name of
  the authentication database to log into MongoDB. In Atlas deployments of
  MongoDB, the authentication database is always the admin database. And
  also must contain a "roles" array. This array contains objects that holds
  a series of roles "roleName", an optional "databaseName" and "collectionName"
  value. For more information regarding the `roles` field, refer to
  [MongoDB Atlas documentation](https://docs.atlas.mongodb.com/reference/api/database-users-create-a-user/).
- `default_ttl` `(string/int): 0` - Specifies the TTL for the leases associated with this role.
  Accepts time suffixed strings ("1h") or an integer number of seconds. Defaults to system/engine default TTL time.
  `max_ttl` `(string/int): 0` - Specifies the maximum TTL for the leases associated with this role. Accepts time
  suffixed strings ("1h") or an integer number of seconds. Defaults to system/mount default TTL time; this value
  is allowed to be less than the mount max TTL (or, if not set, the system max TTL),
  but it is not allowed to be longer. See also [The TTL General Case](https://www.vaultproject.io/docs/concepts/tokens.html#the-general-case).


### Sample Creation Statement

```json
{
	"database_name": "admin",
	"roles": [{
		"databaseName": "admin",
		"roleName": "atlasAdmin"
	},
  {
    "collectionName": "acollection",
    "roleName": "read"
  }]
}
```