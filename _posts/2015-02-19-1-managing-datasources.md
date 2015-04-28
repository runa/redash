--- 
title: Managing Data Sources 
layout: post
category: configuration 
permalink: /configuration/datasources.html
---

re:dash supports several types of data sources (see below the full list) and their management is done with the CLI (`manage.py`):

### Create new data source

#### Version 0.6 and newer

```bash
$ bin/run ./manage.py ds new -n {name} -t {type} -o {options}
```

If you omit any of the options (-n, -t, -o) it will show a prompt asking for it. Options is a JSON string with the connection parameters.
Unless you're doing some sort of automation, it's probably easier to leave it empty and fill out the prompt.

##### Example
```bash
$ bin/run ./manage.py ds new -n "ds-name" -t mysql -o '{"user":"redash","passwd":"foobarbaz","host":"10.125.18.148","db":"ds_db"}'

```

#### Version 0.5 and older

```bash
$ bin/run ./manage.py ds new {name} {type} {options}
```

Where `{name}` is the name of the new data source, `{type}` is one of the supported types and `{options}` is the connection string. For example:

```bash
$ bin/run ./manage.py ds new "Postgres" "pg" "dbname=database"
```

See below for the different supported data sources types and the relevant options string format.

### Listing existing data sources
```bash
$ bin/run ./manage.py ds list
```

## Supported data sources

### PostgreSQL / Redshift

* **Type**: pg
* **Options**:
	* User (user)
	* Password (password)
	* Host (host)
	* Port (port)
	* Database name (dbname) (mandatory)
* **Options string format (for v0.5 and older)**: "user= password= host= port=5439 dbname="

### MySQL

* **Type**: mysql
* **Options**:
	* User (user)
	* Password (passwd)
	* Host (host)
	* Port (port)
	* Database name (db) (mandatory)
* **Options string format (for v0.5 and older)**: "Server=localhost;User=;Pwd=;Database="

Note that you need to install the MySQLDb package as it is not included in the `requirements.txt` file.

### Graphite

* **Type**: graphite
* **Options**:
	* Url (url) (mandatory)
	* User (username)
	* Password (password)
	* Verify SSL ceritficate (verify)
* **Options string format**: '{"url": "https://graphite.yourcompany.com", "auth": ["user", "password"], "verify": true}'

### Google BigQuery

* **Type**: bigquery
* **Options**:
	* Service Account (serviceAccount) (mandatory)
	* Project ID (projectId) (mandatory)
	* Private Key filename (privateKey) (mandatory)
* **Options string format (for v0.5 and older)**: {"serviceAccount" : "43242343247-fjdfakljr3r2@developer.gserviceaccount.com", "privateKey" : "/somewhere/23fjkfjdsfj21312-privatekey.p12", "projectId" : "myproject-123" }

Notes:

1. To obtain BigQuery credentials follow the guidelines at: https://developers.google.com/bigquery/authorization#service-accounts
2. You need to install the `google-api-python-client`, `oauth2client` and `pyopenssl` packages (PyOpenSSL requires `libffi-dev` and `libssl-dev` packages), as they are not included in the `requirements.txt` file.

### MongoDB

* **Type**: mongo
* **Options**:
	* Connection String (connectionString) (mandatory)
	* Database name (dbName)
	* Replica set name (replicaSetName)
* **Options string format (for v0.5 and older)**: { "connectionString" : "mongodb://user:password@localhost:27017/mydb", "dbName" : "mydb" }

For ReplicaSet databases use the following connection string:
* **Options string format**: { "connectionString" : "mongodb://user:pasword@server1:27017,server2:27017/mydb", "dbName" : "mydb", "replicaSetName" : "myreplicaSet" }

Notes:

1. You need to install `pymongo`, as it is not included in the `requirements.txt` file.


### URL

A URL based data source which requests URLs that conforms to the supported [results JSON format](https://github.com/EverythingMe/redash/wiki/re:dash-Data-Source-Results-JSON-Format). 

Very useful in situations where you want to expose the data without connecting directly to the database.

The query itself inside re:dash will simply contain the URL to be executed (i.e. http://myserver/path/myquery)

* **Type**: url
* **Options**:
	* Url (url)
* **Options string format (optional) (for v0.5 and older)**: http://myserver/path/

Notes:

1. All URLs must return the supported [results JSON format]({% post_url 2015-02-19-8-json-format %}).
2. If the Options string is set, only URLs that are part of the supplied path can be executed using this data source. Not setting the options path allows any URL to be executed as long as it returns the supported [results JSON format]({% post_url 2015-02-19-8-json-format %}).


### Script

Allows executing any executable script residing on the server as long as its standard output conforms to the supported [results JSON format]({% post_url 2015-02-19-8-json-format %}).

This integration is useful in situations where you need more than just a query and requires some processing to happen.

Once the path to scripts is configured in the datasource the query needs to contain the file name of the script as well as any command line parameters the script requires (i.e. myscript.py param1 param2 --param3=value)

* **Type**: script
* **Options**:
	* Scripts Path (path) (mandatory)
* **Options string format (for v0.5 and older)**: /path/to/scripts/

Notes:

1. You MUST set a path to execute the scripts, otherwise the data source will not work.
2. All scripts must be executable, otherwise results won't return
3. The script data source does not allow relative paths in the form of "../". You may use a relative sub path such as "./mydir/myscript".
4. All scripts must output to the standard output the supported [results JSON format]({% post_url 2015-02-19-8-json-format %}) and only that, otherwise the data source will not be able to load the data.
