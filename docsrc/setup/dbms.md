Sclera enables a consolidated relational view of multiple underlying relational/non-relational data stores. We now describe how to connect your existing database systems to Sclera. This will enable you to seamlessly execute cross-platform computations via SQL, as discussed in the [ScleraSQL Reference](../sclerasql/sqlregular.md) document.

In addition to the data stores mentioned in this document, Sclera can also incorporate data from CSV files, raw text files, web-services, all within the same SQL query -- for details, please refer to the [ScleraSQL External Data Access](../sclerasql/sqlextdataaccess.md) documentation.

In the following, the keywords appear in upper case to distinguish them from the other terms; however, Sclera is case-insensitive and keywords in actual commands and queries can be in upper or lower case.

## Connecting to Database Systems
Sclera can connect to [Oracle Database 11g Release 2+](http://www.oracle.com), [PostgreSQL 9.1.2+](http://www.postgresql.org), and [MySQL 5.5.28+](http://www.mysql.com) (and also MySQL-compatible systems such as [Google Cloud SQL](https://cloud.google.com/sql)). To connect with a database system not mentioned here, you will need a connector for that database system, which can be built using [Sclera's plugin framework](../sdk/sdkextdbms.md).

<a class="anchor" id="location"></a>In Sclera, a connected database system is called a **location**.

A new location is added to Sclera using the `ADD LOCATION` command, which has the following syntax:

    ADD [ READONLY ] LOCATION location_name AS location_dbms( location_database [ , connection_properties [, ...] ] )

- The optional `READONLY` modifier flags the location as a read-only location; this is [explained later](#read-write-versus-read-only-mode).
- The mandatory `location_name` is a unique name given to the database system being added, for reference within Sclera.
- The mandatory `location_dbms` is the name of the database system: `MYSQL`, or `POSTGRESQL`; each of these are discussed in turn in the subsections that follow.
- The mandatory `location_database` is the name of the database within the `location_dbms` that contains the data we need to access; as discussed in the subsections below.
- The optional `connection_properties` list contains optional connection configuration properties for the `location_dbms`; these are discussed in context of of specific database systems in the subsections below.

### Connecting to Oracle

The following sets up the data location `oraloc`, configured as a [JDBC connection](http://en.wikipedia.org/wiki/Java_Database_Connectivity) to the [Oracle](http://www.oracle.com) database `oradb` on host `orahost`, port `1521`:

    > ADD LOCATION oraloc AS ORACLE("orahost:1521/oradb");

Sclera requires that the database server is accessible over the network and accepts connection via JDBC.

If the the server is running on `localhost` (port `1521`), you can omit the host name:

    > ADD LOCATION oraloc AS ORACLE("oradb");

The JDBC connection is set up using the [Oracle Database Thin JDBC Driver](https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html), which is *not* downloaded during the [Sclera-Oracle Connector](components.md#sclera-oracle) installation. You will need to download this driver before using this component, as explained [later](#sclera-oracle-setup)

Sclera stores the JDBC configuration in its [metadata store](../intro/technical.md#metadata-store), and reconnects to the database at the start of every subsequent Sclera session.

**Connection Properties**

Username and password, if needed, can be specified as follows:

    > ADD LOCATION oraloc AS ORACLE("orahost:1521/oradb", "user=orascott", "password=oratiger");

You can also specify additional properties for customizing the JDBC connection. For instance, to specify the [`internal_logon` property](http://docs.oracle.com/cd/B28359_01/java.111/b31224/urls.htm#CHDDFIJJ):

    > ADD LOCATION oraloc AS ORACLE("orahost:1521/oradb", "user=orascott", "password=oratiger", "internal_logon=SYSDBA");

Any of the properties listed in the [Oracle JDBC documentation](http://docs.oracle.com/cd/B28359_01/java.111/b31224/urls.htm#i1006362) can be specified.

Values of all properties (for instance, passwords) may not be mentioned in the command -- you can specify to enter a property value interactively. See the section on [interactive input](#interactive-input-of-property-values) for details.

<a class="anchor" name="sclera-oracle-setup"></a> **Setup**

Before using this component, you need to download the [Oracle Database Thin JDBC Driver](https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html), and include the path to the downloaded driver in the `CLASSPATH`.

### Connecting to MySQL

*To work with Sclera, MySQL should be configured in the [case-insensitive mode](http://dev.mysql.com/doc/refman/5.6/en/identifier-case-sensitivity.html).*

The following sets up the data location `myloc`, configured as a [JDBC connection](http://en.wikipedia.org/wiki/Java_Database_Connectivity) to the [MySQL](http://www.mysql.com) database `mydb` on host `myhost`:

    > ADD LOCATION myloc AS MYSQL("myhost/mydb");

Sclera requires that the database server is accessible over the network and accepts connection via JDBC.

If the the server is running on `localhost`, you can omit the host name:

    > ADD LOCATION myloc AS MYSQL("mydb");

The JDBC connection is set up using [MySQL Connector/J](http://dev.mysql.com/doc/refman/5.6/en/connector-j.html), which is downloaded during the [Sclera-MySQL Connector](components.md#sclera-mysql) installation.

(Note: Sclera assumes that MySQL runs in a [case-insensitive mode](http://dev.mysql.com/doc/refman/5.6/en/identifier-case-sensitivity.html).)

Sclera stores the JDBC configuration in its [metadata store](../intro/technical.md#metadata-store), and reconnects to the database at the start of every subsequent Sclera session.

**Connection Properties**

Username and password, if needed, can be specified as follows:

    > ADD LOCATION myloc AS MYSQL("myhost/mydb", "user=myscott", "password=mytiger");

You can also specify additional properties for customizing the JDBC connection. For instance, to use SSL connections:

    > ADD LOCATION myloc AS MYSQL("myhost/mydb", "user=myscott", "password=mytiger", "useSSL=true");

Any of the properties listed in the [MySQL Connector/J documentation](http://dev.mysql.com/doc/refman/5.6/en/connector-j-reference-configuration-properties.html#idm47663271142368) can be specified, except `useOldAliasMetadataBehavior`, which is internally set to `true` by Sclera.

Values of all properties (for instance, passwords) may not be mentioned in the command -- you can specify to enter a property value interactively. See the section on [interactive input](#interactive-input-of-property-values) for details.

### Connecting to PostgreSQL

The following sets up the data location `pgloc`, configured as a [JDBC connection](http://en.wikipedia.org/wiki/Java_Database_Connectivity) to the [PostgreSQL](http://www.postgresql.org) database `pgdb` on host `pghost`:

    > ADD LOCATION pgloc AS POSTGRESQL("pghost/pgdb");

Sclera requires that the database server is accessible over the network and accepts connection via JDBC.

If the the server is running on `localhost`, you can omit the host name:

    > ADD LOCATION pgloc AS POSTGRESQL("pgdb");

The JDBC connection is set up using the [PostgreSQL JDBC Driver](http://jdbc.postgresql.org), which is downloaded during the [Sclera-PostgreSQL Connector](components.md#sclera-postgresql) installation.

Sclera stores the JDBC configuration in its [metadata store](../intro/technical.md#metadata-store), and reconnects to the database at the start of every subsequent Sclera session.

**Connection Properties**

Username and password, if needed, can be specified as follows:

    > ADD LOCATION pgloc AS POSTGRESQL("pghost/pgdb", "user=pgscott", "password=pgtiger");

You can also specify additional properties for customizing the JDBC connection. For instance, to use SSL connections:

    > ADD LOCATION pgloc AS POSTGRESQL("pghost/pgdb", "user=pgscott", "password=pgtiger", "ssl=true");

Any of the properties listed in the [PostgreSQL JDBC Driver documentation](http://jdbc.postgresql.org/documentation/head/connect.html#connection-parameters) can be specified as above.

Values of all properties (for instance, passwords) may not be mentioned in the command -- you can specify to enter a property value interactively. See the section on [interactive input](#interactive-input-of-property-values) for details.

### Connecting to Heroku Postgres

The following sets up the data location `heroku`, configured as a [JDBC connection](http://en.wikipedia.org/wiki/Java_Database_Connectivity) to a [Heroku Postgres](https://www.heroku.com/postgres) database:

    > ADD LOCATION heroku AS HEROKU_POSTGRESQL("postgres://<username>:<password>@<host>:<port>/<dbname>");

The parameter is the `DATABASE_URL` in the format provided by Heroku -- see [Heroku's documentation](https://devcenter.heroku.com/articles/connecting-to-heroku-postgres-databases-from-outside-of-heroku) for the details.

The JDBC connection is set up using the [PostgreSQL JDBC Driver](http://jdbc.postgresql.org).

Sclera stores the JDBC configuration in its [metadata store](../intro/technical.md#metadata-store), and reconnects to the database at the start of every subsequent Sclera session.

*Note: Heroku Postgres connectivity was introduced in Sclera 3.0 and is currently in beta.*

## Interactive Input of Property Values

Sometimes, you may not want to specify a property value as a part of the statement. For instance, it might be desirable to not disclose the password in the statements for MySQL and PostgreSQL above.

In such cases, you can specify the character '`?`' as the property value instead. While processing the statement, Sclera will prompt for the actual property value and accept the input in a secure manner.

For example:

    > ADD LOCATION pgloc AS POSTGRESQL("pghost/pgdb", "user=pgscott", "password=?");
      password: *******

Here, the `password=?` generates a prompt that enables you to enter the password interactively.

## Read-Write versus Read-Only Mode

By default, Sclera uses the underlying database system in a read-write mode. This means that:

- The database system acts as a data source. However, if the data needs to be filtered, projected, aggregated, and/or joined with other tables within this database system, then the computation is pushed to this database system and only the result is read by Sclera.
- If the result as above needs to join with data coming from elsewhere, Sclera may decide to materialize the external data as a table within this database system, and perform the join on this database system. This involves creating a table, inserting this external data into the table and then performing the computation.

If you do not want (or are not permitted to) create/update tables on a database system, connect to it as a `readonly` location:

    > ADD READONLY LOCATION myloc AS MYSQL("myhost/mydb");

If you want specify a default for all connections, please set the `sclera.connections.accessmode` configuration parameter to `readwrite` or `readonly`. In the initial configuration, the value is `readwrite`.

## Removing Connected Platforms

The following command removes a location referenced by `location_name`:

    REMOVE LOCATION location_name

Sclera requires that all tables under a location be removed or deleted before removing a location (see [the next section](#importing-database-tables)). 

## Importing Database Tables

After a database system has been connected, you can pick tables from the database system and add to Sclera.

By default, adding a table involves reading (or computing) the metadata -- the set of columns, their data types, the key constraints, etc. -- of the table and storing in the metadata store, so that you can [manage the data and frame queries on the same](../sclerasql/sqlregular.md).

The command to add tables has the following syntax:

    ADD TABLE location_name.table_name [ (
      column_name data_type [ column_constraint [ ... ] ] [, ...]
      [ , table_constraint [, ...] ]
    ) ]

This command adds to Sclera a new table named `table_name` from the database system connected as location `location_name`. You can optionally specify the table schema and constraints explicitly. The details of the parameters used in the schema specifications are similar to that in the [`CREATE TABLE` statement](../sclerasql/sqlregular.md#creating-empty-tables); please see the associated discussion on [column names and types](../sclerasql/sqlregular.md#columns-and-their-types) and [column constraints](../sclerasql/sqlregular.md#column-constraints).

For instance, the following add a new table `mytable` from location `loc` and lets Sclera determine the table's schema:

    > ADD TABLE loc.mytable;

However, the following mentions the schema as well, obviating the need for Sclera to determine the metadata:

    > ADD TABLE loc.mytable(a int primary key, b int);

The above statement specifies the table as having two integer columns, `a` and `b`, with `a` being the primary key. The columns `a` and `b` must be present in the underlying table `mytable` at the source database system, with compatible data types. The underlying table can have other columns and constraints, but they are not visible to Sclera now.

When the table metadata is specified explicitly, the actual table metadata is not queried, and Sclera does not verify that the columns are actually present, or the specified constraints (e.g. the primary key constraint in the example above) actually hold. However, this is useful when getting the table metadata from the underlying database system is expensive (this is just a one-time computation, though).

The [Sclera Command-Line Shell](../interface/shell.md#exploring-metadata) provides commands to list the set of imported tables under a location, tables in the location available for import, and so on.

## Removing Database Tables
When you no longer need a table `table_name` in location `location_name`, you can remove it as follows:

    REMOVE TABLE [location_name.]table_name

The `location_name` can be omitted if the `table_name` is unique across locations.

This command only removes the metadata for the table from Sclera's [metadata store](../intro/technical.md#metadata-store). The actual table in the underlying data store is not deleted.
