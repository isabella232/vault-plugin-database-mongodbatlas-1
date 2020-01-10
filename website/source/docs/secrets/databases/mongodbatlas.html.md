---
layout: "docs"
page_title: "MongoDB Atlas - Database - Secrets Engines"
sidebar_title: "MongoDB Atlas"
sidebar_current: "docs-secrets-databases-mongodb-atlas"
description: |-
  MongoDB Atlas is one of the supported plugins for the Database Secrets Engine. This
  plugin generates MongoDB Atlas Database User credentials dynamically based on configured Vault roles for
  the MongoDB Atlas.
---

# MongoDB Atlas Database Secrets Engine

MongoDB Atlas is one of the supported plugins for the database secrets engine. This
plugin generates database credentials dynamically based on configured roles for
the MongoDB database.

## Setup

1. Enable the database secrets engine if it is not already enabled:

    ```text
    $ vault secrets enable database
    Success! Enabled the database secrets engine at: database/
    ```

    The secrets engine will be enabled at the default path which is name of the engine. To
    enable the secrets engine at a different path use the `-path` argument.

1. Configure Vault with the proper plugin and connection information:

    ```text
    $ vault write database/config/my-mongodbatlas-database \
        plugin_name=mongodbatlas-database-plugin \
        allowed_roles="my-role" \
        public_key="a-public-key" \
        private_key="a-private-key!" \
        project_id="a-project-id"
    ```

2. Configure a role that maps a name in Vault to a MongoDB Atlas command that executes and
   creates the Database User credential:

    ```text
    $ vault write database/roles/my-role \
        db_name=my-mongodbatlas-database \
        creation_statements='{ "database_name": "admin", "roles": [{"databaseName":"admin","roleName":"atlasAdmin"}]}' \
        default_ttl="1h" \
        max_ttl="24h"
    Success! Data written to: database/roles/my-role
    ```

## Usage

After the secrets engine is configured and a user/machine has a Vault token with
the proper permission, it can generate credentials.

1. Generate a new credential by reading from the `/creds` endpoint with the name
of the role:

    ```text
    $ vault read database/creds/my-role
    Key                Value
    ---                -----
    lease_id           database/creds/my-role/2f6a614c-4aa2-7b19-24b9-ad944a8d4de6
    lease_duration     1h
    lease_renewable    true
    password           A1a-QwxApKgnfCp1AJYN
    username           v-root-my-role-5WFTBKdwOTLOqWLgsjvH-1565815206
    ```

## API

The full list of configurable options can be seen in the [MongoDB database
plugin API](/api/secret/databases/mongodb.html) page.

For more information on the database secrets engine's HTTP API please see the
[Database secrets engine API](/api/secret/databases/index.html) page.
