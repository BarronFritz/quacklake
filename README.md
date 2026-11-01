# Quack Lake

Fast, Free, Local, and Easy Datalake

Requirements: DuckDB 2.0.0+

This guide will show you how to setup
- Ducklake open table format metadata database backed by duckdb database file.
- Local client-server using duckdb Quack protocol

# Easy Mode

```bash
# one-liner
duckdb {optional_metadata_file} -serve -cmd "ATTACH 'ducklake:quack:localhost' as lake (DATA_PATH 'quick_lake_files/'); USE lake;"
```

# Medium Mode

```bash
# open duckdb connected to metadata file
duckdb quick_lake.metadata
# create a temporary password
quacklake> D  CREATE SECRET quack_pass (TYPE quack, TOKEN 'quack_pa55w0rd', SCOPE 'quack:localhost');
# start server (uses password scoped to quack:localhost)
quacklake> D  CALL quack_serve('quack:localhost');
# connect ducklake (uses password scoped to quack:localhost)
quacklake> D  ATTACH 'ducklake:quack:localhost' as lake; USE lake;
```


# Hard Mode

## Terminal Session #1

### Starting Quack Server

Start duckdb with 

```bash
duckdb quacklake.metadata.duckdb
```

Create a secrete password (token) for the server
```bash
CREATE PERSISTENT SECRET quacklake_quack_token (
	TYPE quack,
	TOKEN 'quack_pa55w0rd',
	SCOPE 'quack:localhost'
);
```

```
quacklake> D CALL quack_serve('quack:localhost', TOKEN 'quack_password');
```


Start the server protocol (uses the quack_token password created above)
```CALL quack_serve('quack:localhost')```


## Connect to quack server and attach ducklake instance created above

Start duckdb
``` duckdb ```

Create a secret to store ducklake parameters
``` 
CREATE PERSISTENT SECRET quacklake_ducklake (
	TYPE ducklake,
	METADATA_PATH '',
	DATA_PATH '...shared data file path...',
	METADATA_PARAMETERS MAP {
		'TYPE': 'quack',
		'SECRET': 'quacklake_quack_token',
		'PATH': 'quack:localhost'
	}
);
```

Attach ducklake open table format to quack server.
Table metadata is saved on the quack servers file backed metadata.

```
ATTACH 'ducklake:quacklake_ducklake' as lake;
```