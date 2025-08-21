# MYSQL 错误码和信息

MySQL programs have access to several types of error information when the server returns an error. For example, the mysql client program displays errors using the following format:

```sql
SELECT * FROM no_such_table;
-- ERROR 1146 (42S02): Table 'test.no_such_table' doesn't exist
```

The message displayed contains three types of information:

- A numeric error code (1146). This number is MySQL-specific and is not portable to other database systems.

- A five-character SQLSTATE value ('42S02'). The values are taken from ANSI SQL and ODBC and are more standardized. Not all MySQL error numbers have corresponding SQLSTATE values. In these cases, 'HY000' (general error) is used.

- A message string that provides a textual description of the error.

For error checking, use error codes, not error messages. Error messages do not change often, but it is possible. Also if the database administrator changes the language setting, that affects the language of error messages.

Error codes are stable across GA releases of a given MySQL series. Before a series reaches GA status, new codes may still be under development and subject to change.

Server error information comes from the following source files. For details about the way that error information is defined, see the MySQL Internals Manual.

- Error message information is listed in the share/errmsg-utf8.txt file. %d and %s represent numbers and strings, respectively, that are substituted into the Message values when they are displayed.

- The Error values listed in share/errmsg-utf8.txt are used to generate the definitions in the include/mysqld_error.h and include/mysqld_ername.h MySQL source files.

- The SQLSTATE values listed in share/errmsg-utf8.txt are used to generate the definitions in the include/sql_state.h MySQL source file.

Because updates are frequent, it is possible that those files will contain additional error information not listed here.

- Error: 1 SQLSTATE: HY000 (EE_CANTCREATEFILE)

Message: Can't create/write to file '%s' (Errcode: %d - %s)

- Error: 2 SQLSTATE: HY000 (EE_READ)

Message: Error reading file '%s' (Errcode: %d - %s)

- Error: 3 SQLSTATE: HY000 (EE_WRITE)

Message: Error writing file '%s' (Errcode: %d - %s)

- Error: 4 SQLSTATE: HY000 (EE_BADCLOSE)

Message: Error on close of '%s' (Errcode: %d - %s)

- Error: 5 SQLSTATE: HY000 (EE_OUTOFMEMORY)

Message: Out of memory (Needed %u bytes)

- Error: 6 SQLSTATE: HY000 (EE_DELETE)

Message: Error on delete of '%s' (Errcode: %d - %s)

- Error: 7 SQLSTATE: HY000 (EE_LINK)

Message: Error on rename of '%s' to '%s' (Errcode: %d - %s)

- Error: 9 SQLSTATE: HY000 (EE_EOFERR)

Message: Unexpected EOF found when reading file '%s' (Errcode: %d - %s)

- Error: 10 SQLSTATE: HY000 (EE_CANTLOCK)

Message: Can't lock file (Errcode: %d - %s)

- Error: 11 SQLSTATE: HY000 (EE_CANTUNLOCK)

Message: Can't unlock file (Errcode: %d - %s)

- Error: 12 SQLSTATE: HY000 (EE_DIR)

Message: Can't read dir of '%s' (Errcode: %d - %s)

- Error: 13 SQLSTATE: HY000 (EE_STAT)

Message: Can't get stat of '%s' (Errcode: %d - %s)

- Error: 14 SQLSTATE: HY000 (EE_CANT_CHSIZE)

Message: Can't change size of file (Errcode: %d - %s)

- Error: 15 SQLSTATE: HY000 (EE_CANT_OPEN_STREAM)

Message: Can't open stream from handle (Errcode: %d - %s)

- Error: 16 SQLSTATE: HY000 (EE_GETWD)

Message: Can't get working directory (Errcode: %d - %s)

- Error: 17 SQLSTATE: HY000 (EE_SETWD)

Message: Can't change dir to '%s' (Errcode: %d - %s)

- Error: 18 SQLSTATE: HY000 (EE_LINK_WARNING)

Message: Warning: '%s' had %d links

- Error: 19 SQLSTATE: HY000 (EE_OPEN_WARNING)

Message: Warning: %d files and %d streams is left open

- Error: 20 SQLSTATE: HY000 (EE_DISK_FULL)

Message: Disk is full writing '%s' (Errcode: %d - %s). Waiting for someone to free space...

- Error: 21 SQLSTATE: HY000 (EE_CANT_MKDIR)

Message: Can't create directory '%s' (Errcode: %d - %s)

- Error: 22 SQLSTATE: HY000 (EE_UNKNOWN_CHARSET)

Message: Character set '%s' is not a compiled character set and is not specified in the '%s' file

- Error: 23 SQLSTATE: HY000 (EE_OUT_OF_FILERESOURCES)

Message: Out of resources when opening file '%s' (Errcode: %d - %s)

- Error: 24 SQLSTATE: HY000 (EE_CANT_READLINK)

Message: Can't read value for symlink '%s' (Error %d - %s)

- Error: 25 SQLSTATE: HY000 (EE_CANT_SYMLINK)

Message: Can't create symlink '%s' pointing at '%s' (Error %d - %s)

- Error: 26 SQLSTATE: HY000 (EE_REALPATH)

Message: Error on realpath() on '%s' (Error %d - %s)

- Error: 27 SQLSTATE: HY000 (EE_SYNC)

Message: Can't sync file '%s' to disk (Errcode: %d - %s)

- Error: 28 SQLSTATE: HY000 (EE_UNKNOWN_COLLATION)

Message: Collation '%s' is not a compiled collation and is not specified in the '%s' file

- Error: 29 SQLSTATE: HY000 (EE_FILENOTFOUND)

Message: File '%s' not found (Errcode: %d - %s)

- Error: 30 SQLSTATE: HY000 (EE_FILE_NOT_CLOSED)

Message: File '%s' (fileno: %d) was not closed

- Error: 31 SQLSTATE: HY000 (EE_CHANGE_OWNERSHIP)

Message: Can't change ownership of the file '%s' (Errcode: %d - %s)

- Error: 32 SQLSTATE: HY000 (EE_CHANGE_PERMISSIONS)

Message: Can't change permissions of the file '%s' (Errcode: %d - %s)

- Error: 33 SQLSTATE: HY000 (EE_CANT_SEEK)

Message: Can't seek in file '%s' (Errcode: %d - %s)

- Error: 34 SQLSTATE: HY000 (EE_CAPACITY_EXCEEDED)

Message: Memory capacity exceeded (capacity %llu bytes)

EE_CAPACITY_EXCEEDED was added in 5.7.9.

- Error: 1000 SQLSTATE: HY000 (ER_HASHCHK)

Message: hashchk

Unused.

- Error: 1001 SQLSTATE: HY000 (ER_NISAMCHK)

Message: isamchk

Unused.

- Error: 1002 SQLSTATE: HY000 (ER_NO)

Message: NO

Used in the construction of other messages.

- Error: 1003 SQLSTATE: HY000 (ER_YES)

Message: YES

Used in the construction of other messages.

Extended EXPLAIN format generates Note messages. ER_YES is used in the Code column for these messages in subsequent SHOW WARNINGS output.

- Error: 1004 SQLSTATE: HY000 (ER_CANT_CREATE_FILE)

Message: Can't create file '%s' (errno: %d - %s)

Occurs for failure to create or copy a file needed for some operation.

Possible causes: Permissions problem for source file; destination file already exists but is not writeable.

- Error: 1005 SQLSTATE: HY000 (ER_CANT_CREATE_TABLE)

Message: Can't create table '%s' (errno: %d)

InnoDB reports this error when a table cannot be created. If the error message refers to error 150, table creation failed because a foreign key constraint was not correctly formed. If the error message refers to error −1, table creation probably failed because the table includes a column name that matched the name of an internal InnoDB table.

- Error: 1006 SQLSTATE: HY000 (ER_CANT_CREATE_DB)

Message: Can't create database '%s' (errno: %d)

- Error: 1007 SQLSTATE: HY000 (ER_DB_CREATE_EXISTS)

Message: Can't create database '%s'; database exists

An attempt to create a database failed because the database already exists.

Drop the database first if you really want to replace an existing database, or add an IF NOT EXISTS clause to the CREATE DATABASE statement if to retain an existing database without having the statement produce an error.

- Error: 1008 SQLSTATE: HY000 (ER_DB_DROP_EXISTS)

Message: Can't drop database '%s'; database doesn't exist

- Error: 1009 SQLSTATE: HY000 (ER_DB_DROP_DELETE)

Message: Error dropping database (can't delete '%s', errno: %d)

- Error: 1010 SQLSTATE: HY000 (ER_DB_DROP_RMDIR)

Message: Error dropping database (can't rmdir '%s', errno: %d)

- Error: 1011 SQLSTATE: HY000 (ER_CANT_DELETE_FILE)

Message: Error on delete of '%s' (errno: %d - %s)

- Error: 1012 SQLSTATE: HY000 (ER_CANT_FIND_SYSTEM_REC)

Message: Can't read record in system table

Returned by InnoDB for attempts to access InnoDB INFORMATION_SCHEMA tables when InnoDB is unavailable.

- Error: 1013 SQLSTATE: HY000 (ER_CANT_GET_STAT)

Message: Can't get status of '%s' (errno: %d - %s)

- Error: 1014 SQLSTATE: HY000 (ER_CANT_GET_WD)

Message: Can't get working directory (errno: %d - %s)

- Error: 1015 SQLSTATE: HY000 (ER_CANT_LOCK)

Message: Can't lock file (errno: %d - %s)

- Error: 1016 SQLSTATE: HY000 (ER_CANT_OPEN_FILE)

Message: Can't open file: '%s' (errno: %d - %s)

InnoDB reports this error when the table from the InnoDB data files cannot be found, even though the .frm file for the table exists. See Section 14.21.3, “Troubleshooting InnoDB Data Dictionary Operations”.

- Error: 1017 SQLSTATE: HY000 (ER_FILE_NOT_FOUND)

Message: Can't find file: '%s' (errno: %d - %s)

- Error: 1018 SQLSTATE: HY000 (ER_CANT_READ_DIR)

Message: Can't read dir of '%s' (errno: %d - %s)

- Error: 1019 SQLSTATE: HY000 (ER_CANT_SET_WD)

Message: Can't change dir to '%s' (errno: %d - %s)

- Error: 1020 SQLSTATE: HY000 (ER_CHECKREAD)

Message: Record has changed since last read in table '%s'

- Error: 1021 SQLSTATE: HY000 (ER_DISK_FULL)

Message: Disk full (%s); waiting for someone to free some space... (errno: %d - %s)

- Error: 1022 SQLSTATE: 23000 (ER_DUP_KEY)

Message: Can't write; duplicate key in table '%s'

- Error: 1023 SQLSTATE: HY000 (ER_ERROR_ON_CLOSE)

Message: Error on close of '%s' (errno: %d - %s)

- Error: 1024 SQLSTATE: HY000 (ER_ERROR_ON_READ)

Message: Error reading file '%s' (errno: %d - %s)

- Error: 1025 SQLSTATE: HY000 (ER_ERROR_ON_RENAME)

Message: Error on rename of '%s' to '%s' (errno: %d - %s)

- Error: 1026 SQLSTATE: HY000 (ER_ERROR_ON_WRITE)

Message: Error writing file '%s' (errno: %d - %s)

- Error: 1027 SQLSTATE: HY000 (ER_FILE_USED)

Message: '%s' is locked against change

- Error: 1028 SQLSTATE: HY000 (ER_FILSORT_ABORT)

Message: Sort aborted

- Error: 1029 SQLSTATE: HY000 (ER_FORM_NOT_FOUND)

Message: View '%s' doesn't exist for '%s'

- Error: 1030 SQLSTATE: HY000 (ER_GET_ERRNO)

Message: Got error %d from storage engine

Check the %d value to see what the OS error means. For example, 28 indicates that you have run out of disk space.

- Error: 1031 SQLSTATE: HY000 (ER_ILLEGAL_HA)

Message: Table storage engine for '%s' doesn't have this option

- Error: 1032 SQLSTATE: HY000 (ER_KEY_NOT_FOUND)

Message: Can't find record in '%s'

- Error: 1033 SQLSTATE: HY000 (ER_NOT_FORM_FILE)

Message: Incorrect information in file: '%s'

- Error: 1034 SQLSTATE: HY000 (ER_NOT_KEYFILE)

Message: Incorrect key file for table '%s'; try to repair it

- Error: 1035 SQLSTATE: HY000 (ER_OLD_KEYFILE)

Message: Old key file for table '%s'; repair it!

- Error: 1036 SQLSTATE: HY000 (ER_OPEN_AS_READONLY)

Message: Table '%s' is read only

- Error: 1037 SQLSTATE: HY001 (ER_OUTOFMEMORY)

Message: Out of memory; restart server and try again (needed %d bytes)

- Error: 1038 SQLSTATE: HY001 (ER_OUT_OF_SORTMEMORY)

Message: Out of sort memory, consider increasing server sort buffer size

- Error: 1039 SQLSTATE: HY000 (ER_UNEXPECTED_EOF)

Message: Unexpected EOF found when reading file '%s' (errno: %d - %s)

- Error: 1040 SQLSTATE: 08004 (ER_CON_COUNT_ERROR)

Message: Too many connections

- Error: 1041 SQLSTATE: HY000 (ER_OUT_OF_RESOURCES)

Message: Out of memory; check if mysqld or some other process uses all available memory; if not, you may have to use 'ulimit' to allow mysqld to use more memory or you can add more swap space

- Error: 1042 SQLSTATE: 08S01 (ER_BAD_HOST_ERROR)

Message: Can't get hostname for your address

- Error: 1043 SQLSTATE: 08S01 (ER_HANDSHAKE_ERROR)

Message: Bad handshake

- Error: 1044 SQLSTATE: 42000 (ER_DBACCESS_DENIED_ERROR)

Message: Access denied for user '%s'@'%s' to database '%s'

- Error: 1045 SQLSTATE: 28000 (ER_ACCESS_DENIED_ERROR)

Message: Access denied for user '%s'@'%s' (using password: %s)

- Error: 1046 SQLSTATE: 3D000 (ER_NO_DB_ERROR)

Message: No database selected

- Error: 1047 SQLSTATE: 08S01 (ER_UNKNOWN_COM_ERROR)

Message: Unknown command

- Error: 1048 SQLSTATE: 23000 (ER_BAD_NULL_ERROR)

Message: Column '%s' cannot be null

- Error: 1049 SQLSTATE: 42000 (ER_BAD_DB_ERROR)

Message: Unknown database '%s'

- Error: 1050 SQLSTATE: 42S01 (ER_TABLE_EXISTS_ERROR)

Message: Table '%s' already exists

- Error: 1051 SQLSTATE: 42S02 (ER_BAD_TABLE_ERROR)

Message: Unknown table '%s'

- Error: 1052 SQLSTATE: 23000 (ER_NON_UNIQ_ERROR)

Message: Column '%s' in %s is ambiguous

```sql
%s = column name
%s = location of column (for example, "field list")
```

Likely cause: A column appears in a query without appropriate qualification, such as in a select list or ON clause.

Examples:

```sql
SELECT i FROM t INNER JOIN t AS t2;
-- ERROR 1052 (23000): Column 'i' in field list is ambiguous

SELECT * FROM t LEFT JOIN t AS t2 ON i = i;
-- ERROR 1052 (23000): Column 'i' in on clause is ambiguous
Resolution:
```
- Qualify the column with the appropriate table name:

```sql
SELECT t2.i FROM t INNER JOIN t AS t2;
```

- Modify the query to avoid the need for qualification:

```sql
SELECT * FROM t LEFT JOIN t AS t2 USING (i);
```

- Error: 1053 SQLSTATE: 08S01 (ER_SERVER_SHUTDOWN)

Message: Server shutdown in progress

- Error: 1054 SQLSTATE: 42S22 (ER_BAD_FIELD_ERROR)

Message: Unknown column '%s' in '%s'

- Error: 1055 SQLSTATE: 42000 (ER_WRONG_FIELD_WITH_GROUP)

Message: '%s' isn't in GROUP BY

- Error: 1056 SQLSTATE: 42000 (ER_WRONG_GROUP_FIELD)

Message: Can't group on '%s'

- Error: 1057 SQLSTATE: 42000 (ER_WRONG_SUM_SELECT)

Message: Statement has sum functions and columns in same statement

- Error: 1058 SQLSTATE: 21S01 (ER_WRONG_VALUE_COUNT)

Message: Column count doesn't match value count

- Error: 1059 SQLSTATE: 42000 (ER_TOO_LONG_IDENT)

Message: Identifier name '%s' is too long

- Error: 1060 SQLSTATE: 42S21 (ER_DUP_FIELDNAME)

Message: Duplicate column name '%s'

- Error: 1061 SQLSTATE: 42000 (ER_DUP_KEYNAME)

Message: Duplicate key name '%s'

- Error: 1062 SQLSTATE: 23000 (ER_DUP_ENTRY)

Message: Duplicate entry '%s' for key %d

The message returned with this error uses the format string for ER_DUP_ENTRY_WITH_KEY_NAME.

- Error: 1063 SQLSTATE: 42000 (ER_WRONG_FIELD_SPEC)

Message: Incorrect column specifier for column '%s'

- Error: 1064 SQLSTATE: 42000 (ER_PARSE_ERROR)

Message: %s near '%s' at line %d

- Error: 1065 SQLSTATE: 42000 (ER_EMPTY_QUERY)

Message: Query was empty

- Error: 1066 SQLSTATE: 42000 (ER_NONUNIQ_TABLE)

Message: Not unique table/alias: '%s'

- Error: 1067 SQLSTATE: 42000 (ER_INVALID_DEFAULT)

Message: Invalid default value for '%s'

- Error: 1068 SQLSTATE: 42000 (ER_MULTIPLE_PRI_KEY)

Message: Multiple primary key defined

- Error: 1069 SQLSTATE: 42000 (ER_TOO_MANY_KEYS)

Message: Too many keys specified; max %d keys allowed

- Error: 1070 SQLSTATE: 42000 (ER_TOO_MANY_KEY_PARTS)

Message: Too many key parts specified; max %d parts allowed

- Error: 1071 SQLSTATE: 42000 (ER_TOO_LONG_KEY)

Message: Specified key was too long; max key length is %d bytes

- Error: 1072 SQLSTATE: 42000 (ER_KEY_COLUMN_DOES_NOT_EXITS)

Message: Key column '%s' doesn't exist in table

- Error: 1073 SQLSTATE: 42000 (ER_BLOB_USED_AS_KEY)

Message: BLOB column '%s' can't be used in key specification with the used table type

- Error: 1074 SQLSTATE: 42000 (ER_TOO_BIG_FIELDLENGTH)

Message: Column length too big for column '%s' (max = %lu); use BLOB or TEXT instead

- Error: 1075 SQLSTATE: 42000 (ER_WRONG_AUTO_KEY)

Message: Incorrect table definition; there can be only one auto column and it must be defined as a key

- Error: 1076 SQLSTATE: HY000 (ER_READY)

Message: %s: ready for connections. Version: '%s' socket: '%s' port: %d

- Error: 1077 SQLSTATE: HY000 (ER_NORMAL_SHUTDOWN)

Message: %s: Normal shutdown

- Error: 1078 SQLSTATE: HY000 (ER_GOT_SIGNAL)

Message: %s: Got signal %d. Aborting!

- Error: 1079 SQLSTATE: HY000 (ER_SHUTDOWN_COMPLETE)

Message: %s: Shutdown complete

- Error: 1080 SQLSTATE: 08S01 (ER_FORCING_CLOSE)

Message: %s: Forcing close of thread %ld user: '%s'

- Error: 1081 SQLSTATE: 08S01 (ER_IPSOCK_ERROR)

Message: Can't create IP socket

- Error: 1082 SQLSTATE: 42S12 (ER_NO_SUCH_INDEX)

Message: Table '%s' has no index like the one used in CREATE INDEX; recreate the table

- Error: 1083 SQLSTATE: 42000 (ER_WRONG_FIELD_TERMINATORS)

Message: Field separator argument is not what is expected; check the manual

- Error: 1084 SQLSTATE: 42000 (ER_BLOBS_AND_NO_TERMINATED)

Message: You can't use fixed rowlength with BLOBs; please use 'fields terminated by'

- Error: 1085 SQLSTATE: HY000 (ER_TEXTFILE_NOT_READABLE)

Message: The file '%s' must be in the database directory or be readable by all

- Error: 1086 SQLSTATE: HY000 (ER_FILE_EXISTS_ERROR)

Message: File '%s' already exists

- Error: 1087 SQLSTATE: HY000 (ER_LOAD_INFO)

Message: Records: %ld Deleted: %ld Skipped: %ld Warnings: %ld

- Error: 1088 SQLSTATE: HY000 (ER_ALTER_INFO)

Message: Records: %ld Duplicates: %ld

- Error: 1089 SQLSTATE: HY000 (ER_WRONG_SUB_KEY)

Message: Incorrect prefix key; the used key part isn't a string, the used length is longer than the key part, or the storage engine doesn't support unique prefix keys

- Error: 1090 SQLSTATE: 42000 (ER_CANT_REMOVE_ALL_FIELDS)

Message: You can't delete all columns with ALTER TABLE; use DROP TABLE instead

- Error: 1091 SQLSTATE: 42000 (ER_CANT_DROP_FIELD_OR_KEY)

Message: Can't DROP '%s'; check that column/key exists

- Error: 1092 SQLSTATE: HY000 (ER_INSERT_INFO)

Message: Records: %ld Duplicates: %ld Warnings: %ld

- Error: 1093 SQLSTATE: HY000 (ER_UPDATE_TABLE_USED)

Message: You can't specify target table '%s' for update in FROM clause

This error occurs for attempts to select from and modify the same table within a single statement. If the select attempt occurs within a derived table, you can avoid this error by setting the derived_merge flag of the optimizer_switch system variable to force the subquery to be materialized into a temporary table, which effectively causes it to be a different table from the one modified. See Section 8.2.2.3, “Optimizing Derived Tables and View References”.

- Error: 1094 SQLSTATE: HY000 (ER_NO_SUCH_THREAD)

Message: Unknown thread id: %lu

- Error: 1095 SQLSTATE: HY000 (ER_KILL_DENIED_ERROR)

Message: You are not owner of thread %lu

- Error: 1096 SQLSTATE: HY000 (ER_NO_TABLES_USED)

Message: No tables used

- Error: 1097 SQLSTATE: HY000 (ER_TOO_BIG_SET)

Message: Too many strings for column %s and SET

- Error: 1098 SQLSTATE: HY000 (ER_NO_UNIQUE_LOGFILE)

Message: Can't generate a unique log-filename %s.(1-999)

- Error: 1099 SQLSTATE: HY000 (ER_TABLE_NOT_LOCKED_FOR_WRITE)

Message: Table '%s' was locked with a READ lock and can't be updated

- Error: 1100 SQLSTATE: HY000 (ER_TABLE_NOT_LOCKED)

Message: Table '%s' was not locked with LOCK TABLES

- Error: 1101 SQLSTATE: 42000 (ER_BLOB_CANT_HAVE_DEFAULT)

Message: BLOB, TEXT, GEOMETRY or JSON column '%s' can't have a default value

- Error: 1102 SQLSTATE: 42000 (ER_WRONG_DB_NAME)

Message: Incorrect database name '%s'

- Error: 1103 SQLSTATE: 42000 (ER_WRONG_TABLE_NAME)

Message: Incorrect table name '%s'

- Error: 1104 SQLSTATE: 42000 (ER_TOO_BIG_SELECT)

Message: The SELECT would examine more than MAX_JOIN_SIZE rows; check your WHERE and use SET SQL_BIG_SELECTS=1 or SET MAX_JOIN_SIZE=# if the SELECT is okay

- Error: 1105 SQLSTATE: HY000 (ER_UNKNOWN_ERROR)

Message: Unknown error

- Error: 1106 SQLSTATE: 42000 (ER_UNKNOWN_PROCEDURE)

Message: Unknown procedure '%s'

- Error: 1107 SQLSTATE: 42000 (ER_WRONG_PARAMCOUNT_TO_PROCEDURE)

Message: Incorrect parameter count to procedure '%s'

- Error: 1108 SQLSTATE: HY000 (ER_WRONG_PARAMETERS_TO_PROCEDURE)

Message: Incorrect parameters to procedure '%s'

- Error: 1109 SQLSTATE: 42S02 (ER_UNKNOWN_TABLE)

Message: Unknown table '%s' in %s

- Error: 1110 SQLSTATE: 42000 (ER_FIELD_SPECIFIED_TWICE)

Message: Column '%s' specified twice

- Error: 1111 SQLSTATE: HY000 (ER_INVALID_GROUP_FUNC_USE)

Message: Invalid use of group function

- Error: 1112 SQLSTATE: 42000 (ER_UNSUPPORTED_EXTENSION)

Message: Table '%s' uses an extension that doesn't exist in this MySQL version

- Error: 1113 SQLSTATE: 42000 (ER_TABLE_MUST_HAVE_COLUMNS)

Message: A table must have at least 1 column

- Error: 1114 SQLSTATE: HY000 (ER_RECORD_FILE_FULL)

Message: The table '%s' is full

InnoDB reports this error when the system tablespace runs out of free space. Reconfigure the system tablespace to add a new data file.

- Error: 1115 SQLSTATE: 42000 (ER_UNKNOWN_CHARACTER_SET)

Message: Unknown character set: '%s'

- Error: 1116 SQLSTATE: HY000 (ER_TOO_MANY_TABLES)

Message: Too many tables; MySQL can only use %d tables in a join

- Error: 1117 SQLSTATE: HY000 (ER_TOO_MANY_FIELDS)

Message: Too many columns

- Error: 1118 SQLSTATE: 42000 (ER_TOO_BIG_ROWSIZE)

Message: Row size too large. The maximum row size for the used table type, not counting BLOBs, is %ld. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs

- Error: 1119 SQLSTATE: HY000 (ER_STACK_OVERRUN)

Message: Thread stack overrun: Used: %ld of a %ld stack. Use 'mysqld --thread_stack=#' to specify a bigger stack if needed

- Error: 1120 SQLSTATE: 42000 (ER_WRONG_OUTER_JOIN)

Message: Cross dependency found in OUTER JOIN; examine your ON conditions

- Error: 1121 SQLSTATE: 42000 (ER_NULL_COLUMN_IN_INDEX)

Message: Table handler doesn't support NULL in given index. Please change column '%s' to be NOT NULL or use another handler

- Error: 1122 SQLSTATE: HY000 (ER_CANT_FIND_UDF)

Message: Can't load function '%s'

- Error: 1123 SQLSTATE: HY000 (ER_CANT_INITIALIZE_UDF)

Message: Can't initialize function '%s'; %s

- Error: 1124 SQLSTATE: HY000 (ER_UDF_NO_PATHS)

Message: No paths allowed for shared library

- Error: 1125 SQLSTATE: HY000 (ER_UDF_EXISTS)

Message: Function '%s' already exists

- Error: 1126 SQLSTATE: HY000 (ER_CANT_OPEN_LIBRARY)

Message: Can't open shared library '%s' (errno: %d %s)

- Error: 1127 SQLSTATE: HY000 (ER_CANT_FIND_DL_ENTRY)

Message: Can't find symbol '%s' in library

- Error: 1128 SQLSTATE: HY000 (ER_FUNCTION_NOT_DEFINED)

Message: Function '%s' is not defined

- Error: 1129 SQLSTATE: HY000 (ER_HOST_IS_BLOCKED)

Message: Host '%s' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'

- Error: 1130 SQLSTATE: HY000 (ER_HOST_NOT_PRIVILEGED)

Message: Host '%s' is not allowed to connect to this MySQL server

- Error: 1131 SQLSTATE: 42000 (ER_PASSWORD_ANONYMOUS_USER)

Message: You are using MySQL as an anonymous user and anonymous users are not allowed to change passwords

- Error: 1132 SQLSTATE: 42000 (ER_PASSWORD_NOT_ALLOWED)

Message: You must have privileges to update tables in the mysql database to be able to change passwords for others

- Error: 1133 SQLSTATE: 42000 (ER_PASSWORD_NO_MATCH)

Message: Can't find any matching row in the user table

- Error: 1134 SQLSTATE: HY000 (ER_UPDATE_INFO)

Message: Rows matched: %ld Changed: %ld Warnings: %ld

- Error: 1135 SQLSTATE: HY000 (ER_CANT_CREATE_THREAD)

Message: Can't create a new thread (errno %d); if you are not out of available memory, you can consult the manual for a possible OS-dependent bug

- Error: 1136 SQLSTATE: 21S01 (ER_WRONG_VALUE_COUNT_ON_ROW)

Message: Column count doesn't match value count at row %ld

- Error: 1137 SQLSTATE: HY000 (ER_CANT_REOPEN_TABLE)

Message: Can't reopen table: '%s'

- Error: 1138 SQLSTATE: 22004 (ER_INVALID_USE_OF_NULL)

Message: Invalid use of NULL value

- Error: 1139 SQLSTATE: 42000 (ER_REGEXP_ERROR)

Message: Got error '%s' from regexp

- Error: 1140 SQLSTATE: 42000 (ER_MIX_OF_GROUP_FUNC_AND_FIELDS)

Message: Mixing of GROUP columns (MIN(),MAX(),COUNT(),...) with no GROUP columns is illegal if there is no GROUP BY clause

- Error: 1141 SQLSTATE: 42000 (ER_NONEXISTING_GRANT)

Message: There is no such grant defined for user '%s' on host '%s'

- Error: 1142 SQLSTATE: 42000 (ER_TABLEACCESS_DENIED_ERROR)

Message: %s command denied to user '%s'@'%s' for table '%s'

- Error: 1143 SQLSTATE: 42000 (ER_COLUMNACCESS_DENIED_ERROR)

Message: %s command denied to user '%s'@'%s' for column '%s' in table '%s'

- Error: 1144 SQLSTATE: 42000 (ER_ILLEGAL_GRANT_FOR_TABLE)

Message: Illegal GRANT/REVOKE command; please consult the manual to see which privileges can be used

- Error: 1145 SQLSTATE: 42000 (ER_GRANT_WRONG_HOST_OR_USER)

Message: The host or user argument to GRANT is too long

- Error: 1146 SQLSTATE: 42S02 (ER_NO_SUCH_TABLE)

Message: Table '%s.%s' doesn't exist

- Error: 1147 SQLSTATE: 42000 (ER_NONEXISTING_TABLE_GRANT)

Message: There is no such grant defined for user '%s' on host '%s' on table '%s'

- Error: 1148 SQLSTATE: 42000 (ER_NOT_ALLOWED_COMMAND)

Message: The used command is not allowed with this MySQL version

- Error: 1149 SQLSTATE: 42000 (ER_SYNTAX_ERROR)

Message: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use

- Error: 1150 SQLSTATE: HY000 (ER_UNUSED1)

Message: Delayed insert thread couldn't get requested lock for table %s

- Error: 1151 SQLSTATE: HY000 (ER_UNUSED2)

Message: Too many delayed threads in use

- Error: 1152 SQLSTATE: 08S01 (ER_ABORTING_CONNECTION)

Message: Aborted connection %ld to db: '%s' user: '%s' (%s)

- Error: 1153 SQLSTATE: 08S01 (ER_NET_PACKET_TOO_LARGE)

Message: Got a packet bigger than 'max_allowed_packet' bytes

- Error: 1154 SQLSTATE: 08S01 (ER_NET_READ_ERROR_FROM_PIPE)

Message: Got a read error from the connection pipe

- Error: 1155 SQLSTATE: 08S01 (ER_NET_FCNTL_ERROR)

Message: Got an error from fcntl()

- Error: 1156 SQLSTATE: 08S01 (ER_NET_PACKETS_OUT_OF_ORDER)

Message: Got packets out of order

- Error: 1157 SQLSTATE: 08S01 (ER_NET_UNCOMPRESS_ERROR)

Message: Couldn't uncompress communication packet

- Error: 1158 SQLSTATE: 08S01 (ER_NET_READ_ERROR)

Message: Got an error reading communication packets

- Error: 1159 SQLSTATE: 08S01 (ER_NET_READ_INTERRUPTED)

Message: Got timeout reading communication packets

- Error: 1160 SQLSTATE: 08S01 (ER_NET_ERROR_ON_WRITE)

Message: Got an error writing communication packets

- Error: 1161 SQLSTATE: 08S01 (ER_NET_WRITE_INTERRUPTED)

Message: Got timeout writing communication packets

- Error: 1162 SQLSTATE: 42000 (ER_TOO_LONG_STRING)

Message: Result string is longer than 'max_allowed_packet' bytes

- Error: 1163 SQLSTATE: 42000 (ER_TABLE_CANT_HANDLE_BLOB)

Message: The used table type doesn't support BLOB/TEXT columns

- Error: 1164 SQLSTATE: 42000 (ER_TABLE_CANT_HANDLE_AUTO_INCREMENT)

Message: The used table type doesn't support AUTO_INCREMENT columns

- Error: 1165 SQLSTATE: HY000 (ER_UNUSED3)

Message: INSERT DELAYED can't be used with table '%s' because it is locked with LOCK TABLES

- Error: 1166 SQLSTATE: 42000 (ER_WRONG_COLUMN_NAME)

Message: Incorrect column name '%s'

- Error: 1167 SQLSTATE: 42000 (ER_WRONG_KEY_COLUMN)

Message: The used storage engine can't index column '%s'

- Error: 1168 SQLSTATE: HY000 (ER_WRONG_MRG_TABLE)

Message: Unable to open underlying table which is differently defined or of non-MyISAM type or doesn't exist

- Error: 1169 SQLSTATE: 23000 (ER_DUP_UNIQUE)

Message: Can't write, because of unique constraint, to table '%s'

- Error: 1170 SQLSTATE: 42000 (ER_BLOB_KEY_WITHOUT_LENGTH)

Message: BLOB/TEXT column '%s' used in key specification without a key length

- Error: 1171 SQLSTATE: 42000 (ER_PRIMARY_CANT_HAVE_NULL)

Message: All parts of a PRIMARY KEY must be NOT NULL; if you need NULL in a key, use UNIQUE instead

- Error: 1172 SQLSTATE: 42000 (ER_TOO_MANY_ROWS)

Message: Result consisted of more than one row

- Error: 1173 SQLSTATE: 42000 (ER_REQUIRES_PRIMARY_KEY)

Message: This table type requires a primary key

- Error: 1174 SQLSTATE: HY000 (ER_NO_RAID_COMPILED)

Message: This version of MySQL is not compiled with RAID support

- Error: 1175 SQLSTATE: HY000 (ER_UPDATE_WITHOUT_KEY_IN_SAFE_MODE)

Message: You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column

- Error: 1176 SQLSTATE: 42000 (ER_KEY_DOES_NOT_EXITS)

Message: Key '%s' doesn't exist in table '%s'

- Error: 1177 SQLSTATE: 42000 (ER_CHECK_NO_SUCH_TABLE)

Message: Can't open table

- Error: 1178 SQLSTATE: 42000 (ER_CHECK_NOT_IMPLEMENTED)

Message: The storage engine for the table doesn't support %s

- Error: 1179 SQLSTATE: 25000 (ER_CANT_DO_THIS_DURING_AN_TRANSACTION)

Message: You are not allowed to execute this command in a transaction

- Error: 1180 SQLSTATE: HY000 (ER_ERROR_DURING_COMMIT)

Message: Got error %d during COMMIT

- Error: 1181 SQLSTATE: HY000 (ER_ERROR_DURING_ROLLBACK)

Message: Got error %d during ROLLBACK

- Error: 1182 SQLSTATE: HY000 (ER_ERROR_DURING_FLUSH_LOGS)

Message: Got error %d during FLUSH_LOGS

- Error: 1183 SQLSTATE: HY000 (ER_ERROR_DURING_CHECKPOINT)

Message: Got error %d during CHECKPOINT

- Error: 1184 SQLSTATE: 08S01 (ER_NEW_ABORTING_CONNECTION)

Message: Aborted connection %u to db: '%s' user: '%s' host: '%s' (%s)

- Error: 1185 SQLSTATE: HY000 (ER_DUMP_NOT_IMPLEMENTED)

Message: The storage engine for the table does not support binary table dump

- Error: 1186 SQLSTATE: HY000 (ER_FLUSH_MASTER_BINLOG_CLOSED)

Message: Binlog closed, cannot RESET MASTER

- Error: 1187 SQLSTATE: HY000 (ER_INDEX_REBUILD)

Message: Failed rebuilding the index of dumped table '%s'

- Error: 1188 SQLSTATE: HY000 (ER_MASTER)

Message: Error from master: '%s'

- Error: 1189 SQLSTATE: 08S01 (ER_MASTER_NET_READ)

Message: Net error reading from master

- Error: 1190 SQLSTATE: 08S01 (ER_MASTER_NET_WRITE)

Message: Net error writing to master

- Error: 1191 SQLSTATE: HY000 (ER_FT_MATCHING_KEY_NOT_FOUND)

Message: Can't find FULLTEXT index matching the column list

- Error: 1192 SQLSTATE: HY000 (ER_LOCK_OR_ACTIVE_TRANSACTION)

Message: Can't execute the given command because you have active locked tables or an active transaction

- Error: 1193 SQLSTATE: HY000 (ER_UNKNOWN_SYSTEM_VARIABLE)

Message: Unknown system variable '%s'

- Error: 1194 SQLSTATE: HY000 (ER_CRASHED_ON_USAGE)

Message: Table '%s' is marked as crashed and should be repaired

- Error: 1195 SQLSTATE: HY000 (ER_CRASHED_ON_REPAIR)

Message: Table '%s' is marked as crashed and last (automatic?) repair failed

- Error: 1196 SQLSTATE: HY000 (ER_WARNING_NOT_COMPLETE_ROLLBACK)

Message: Some non-transactional changed tables couldn't be rolled back

- Error: 1197 SQLSTATE: HY000 (ER_TRANS_CACHE_FULL)

Message: Multi-statement transaction required more than 'max_binlog_cache_size' bytes of storage; increase this mysqld variable and try again

- Error: 1198 SQLSTATE: HY000 (ER_SLAVE_MUST_STOP)

Message: This operation cannot be performed with a running slave; run STOP SLAVE first

- Error: 1199 SQLSTATE: HY000 (ER_SLAVE_NOT_RUNNING)

Message: This operation requires a running slave; configure slave and do START SLAVE

- Error: 1200 SQLSTATE: HY000 (ER_BAD_SLAVE)

Message: The server is not configured as slave; fix in config file or with CHANGE MASTER TO

- Error: 1201 SQLSTATE: HY000 (ER_MASTER_INFO)

Message: Could not initialize master info structure; more error messages can be found in the MySQL error log

- Error: 1202 SQLSTATE: HY000 (ER_SLAVE_THREAD)

Message: Could not create slave thread; check system resources

- Error: 1203 SQLSTATE: 42000 (ER_TOO_MANY_USER_CONNECTIONS)

Message: User %s already has more than 'max_user_connections' active connections

- Error: 1204 SQLSTATE: HY000 (ER_SET_CONSTANTS_ONLY)

Message: You may only use constant expressions with SET

- Error: 1205 SQLSTATE: HY000 (ER_LOCK_WAIT_TIMEOUT)

Message: Lock wait timeout exceeded; try restarting transaction

InnoDB reports this error when lock wait timeout expires. The statement that waited too long was rolled back (not the entire transaction). You can increase the value of the innodb_lock_wait_timeout configuration option if SQL statements should wait longer for other transactions to complete, or decrease it if too many long-running transactions are causing locking problems and reducing concurrency on a busy system.

- Error: 1206 SQLSTATE: HY000 (ER_LOCK_TABLE_FULL)

Message: The total number of locks exceeds the lock table size

InnoDB reports this error when the total number of locks exceeds the amount of memory devoted to managing locks. To avoid this error, increase the value of innodb_buffer_pool_size. Within an individual application, a workaround may be to break a large operation into smaller pieces. For example, if the error occurs for a large INSERT, perform several smaller INSERT operations.

- Error: 1207 SQLSTATE: 25000 (ER_READ_ONLY_TRANSACTION)

Message: Update locks cannot be acquired during a READ UNCOMMITTED transaction

- Error: 1208 SQLSTATE: HY000 (ER_DROP_DB_WITH_READ_LOCK)

Message: DROP DATABASE not allowed while thread is holding global read lock

- Error: 1209 SQLSTATE: HY000 (ER_CREATE_DB_WITH_READ_LOCK)

Message: CREATE DATABASE not allowed while thread is holding global read lock

- Error: 1210 SQLSTATE: HY000 (ER_WRONG_ARGUMENTS)

Message: Incorrect arguments to %s

- Error: 1211 SQLSTATE: 42000 (ER_NO_PERMISSION_TO_CREATE_USER)

Message: '%s'@'%s' is not allowed to create new users

- Error: 1212 SQLSTATE: HY000 (ER_UNION_TABLES_IN_DIFFERENT_DIR)

Message: Incorrect table definition; all MERGE tables must be in the same database

- Error: 1213 SQLSTATE: 40001 (ER_LOCK_DEADLOCK)

Message: Deadlock found when trying to get lock; try restarting transaction

InnoDB reports this error when a transaction encounters a deadlock and is automatically rolled back so that your application can take corrective action. To recover from this error, run all the operations in this transaction again. A deadlock occurs when requests for locks arrive in inconsistent order between transactions. The transaction that was rolled back released all its locks, and the other transaction can now get all the locks it requested. Thus, when you re-run the transaction that was rolled back, it might have to wait for other transactions to complete, but typically the deadlock does not recur. If you encounter frequent deadlocks, make the sequence of locking operations (LOCK TABLES, SELECT ... FOR UPDATE, and so on) consistent between the different transactions or applications that experience the issue. See Section 14.5.5, “Deadlocks in InnoDB” for details.

- Error: 1214 SQLSTATE: HY000 (ER_TABLE_CANT_HANDLE_FT)

Message: The used table type doesn't support FULLTEXT indexes

- Error: 1215 SQLSTATE: HY000 (ER_CANNOT_ADD_FOREIGN)

Message: Cannot add foreign key constraint

- Error: 1216 SQLSTATE: 23000 (ER_NO_REFERENCED_ROW)

Message: Cannot add or update a child row: a foreign key constraint fails

InnoDB reports this error when you try to add a row but there is no parent row, and a foreign key constraint fails. Add the parent row first.

- Error: 1217 SQLSTATE: 23000 (ER_ROW_IS_REFERENCED)

Message: Cannot delete or update a parent row: a foreign key constraint fails

InnoDB reports this error when you try to delete a parent row that has children, and a foreign key constraint fails. Delete the children first.

- Error: 1218 SQLSTATE: 08S01 (ER_CONNECT_TO_MASTER)

Message: Error connecting to master: %s

- Error: 1219 SQLSTATE: HY000 (ER_QUERY_ON_MASTER)

Message: Error running query on master: %s

- Error: 1220 SQLSTATE: HY000 (ER_ERROR_WHEN_EXECUTING_COMMAND)

Message: Error when executing command %s: %s

- Error: 1221 SQLSTATE: HY000 (ER_WRONG_USAGE)

Message: Incorrect usage of %s and %s

- Error: 1222 SQLSTATE: 21000 (ER_WRONG_NUMBER_OF_COLUMNS_IN_SELECT)

Message: The used SELECT statements have a different number of columns

- Error: 1223 SQLSTATE: HY000 (ER_CANT_UPDATE_WITH_READLOCK)

Message: Can't execute the query because you have a conflicting read lock

- Error: 1224 SQLSTATE: HY000 (ER_MIXING_NOT_ALLOWED)

Message: Mixing of transactional and non-transactional tables is disabled

- Error: 1225 SQLSTATE: HY000 (ER_DUP_ARGUMENT)

Message: Option '%s' used twice in statement

- Error: 1226 SQLSTATE: 42000 (ER_USER_LIMIT_REACHED)

Message: User '%s' has exceeded the '%s' resource (current value: %ld)

- Error: 1227 SQLSTATE: 42000 (ER_SPECIFIC_ACCESS_DENIED_ERROR)

Message: Access denied; you need (at least one of) the %s privilege(s) for this operation

- Error: 1228 SQLSTATE: HY000 (ER_LOCAL_VARIABLE)

Message: Variable '%s' is a SESSION variable and can't be used with SET GLOBAL

- Error: 1229 SQLSTATE: HY000 (ER_GLOBAL_VARIABLE)

Message: Variable '%s' is a GLOBAL variable and should be set with SET GLOBAL

- Error: 1230 SQLSTATE: 42000 (ER_NO_DEFAULT)

Message: Variable '%s' doesn't have a default value

- Error: 1231 SQLSTATE: 42000 (ER_WRONG_VALUE_FOR_VAR)

Message: Variable '%s' can't be set to the value of '%s'

- Error: 1232 SQLSTATE: 42000 (ER_WRONG_TYPE_FOR_VAR)

Message: Incorrect argument type to variable '%s'

- Error: 1233 SQLSTATE: HY000 (ER_VAR_CANT_BE_READ)

Message: Variable '%s' can only be set, not read

- Error: 1234 SQLSTATE: 42000 (ER_CANT_USE_OPTION_HERE)

Message: Incorrect usage/placement of '%s'

- Error: 1235 SQLSTATE: 42000 (ER_NOT_SUPPORTED_YET)

Message: This version of MySQL doesn't yet support '%s'

- Error: 1236 SQLSTATE: HY000 (ER_MASTER_FATAL_ERROR_READING_BINLOG)

Message: Got fatal error %d from master when reading data from binary log: '%s'

- Error: 1237 SQLSTATE: HY000 (ER_SLAVE_IGNORED_TABLE)

Message: Slave SQL thread ignored the query because of replicate-*-table rules

- Error: 1238 SQLSTATE: HY000 (ER_INCORRECT_GLOBAL_LOCAL_VAR)

Message: Variable '%s' is a %s variable

- Error: 1239 SQLSTATE: 42000 (ER_WRONG_FK_DEF)

Message: Incorrect foreign key definition for '%s': %s

- Error: 1240 SQLSTATE: HY000 (ER_KEY_REF_DO_NOT_MATCH_TABLE_REF)

Message: Key reference and table reference don't match

- Error: 1241 SQLSTATE: 21000 (ER_OPERAND_COLUMNS)

Message: Operand should contain %d column(s)

- Error: 1242 SQLSTATE: 21000 (ER_SUBQUERY_NO_1_ROW)

Message: Subquery returns more than 1 row

- Error: 1243 SQLSTATE: HY000 (ER_UNKNOWN_STMT_HANDLER)

Message: Unknown prepared statement handler (%.*s) given to %s

- Error: 1244 SQLSTATE: HY000 (ER_CORRUPT_HELP_DB)

Message: Help database is corrupt or does not exist

- Error: 1245 SQLSTATE: HY000 (ER_CYCLIC_REFERENCE)

Message: Cyclic reference on subqueries

- Error: 1246 SQLSTATE: HY000 (ER_AUTO_CONVERT)

Message: Converting column '%s' from %s to %s

- Error: 1247 SQLSTATE: 42S22 (ER_ILLEGAL_REFERENCE)

Message: Reference '%s' not supported (%s)

- Error: 1248 SQLSTATE: 42000 (ER_DERIVED_MUST_HAVE_ALIAS)

Message: Every derived table must have its own alias

- Error: 1249 SQLSTATE: 01000 (ER_SELECT_REDUCED)

Message: Select %u was reduced during optimization

- Error: 1250 SQLSTATE: 42000 (ER_TABLENAME_NOT_ALLOWED_HERE)

Message: Table '%s' from one of the SELECTs cannot be used in %s

- Error: 1251 SQLSTATE: 08004 (ER_NOT_SUPPORTED_AUTH_MODE)

Message: Client does not support authentication protocol requested by server; consider upgrading MySQL client

- Error: 1252 SQLSTATE: 42000 (ER_SPATIAL_CANT_HAVE_NULL)

Message: All parts of a SPATIAL index must be NOT NULL

- Error: 1253 SQLSTATE: 42000 (ER_COLLATION_CHARSET_MISMATCH)

Message: COLLATION '%s' is not valid for CHARACTER SET '%s'

- Error: 1254 SQLSTATE: HY000 (ER_SLAVE_WAS_RUNNING)

Message: Slave is already running

- Error: 1255 SQLSTATE: HY000 (ER_SLAVE_WAS_NOT_RUNNING)

Message: Slave already has been stopped

- Error: 1256 SQLSTATE: HY000 (ER_TOO_BIG_FOR_UNCOMPRESS)

Message: Uncompressed data size too large; the maximum size is %d (probably, length of uncompressed data was corrupted)

- Error: 1257 SQLSTATE: HY000 (ER_ZLIB_Z_MEM_ERROR)

Message: ZLIB: Not enough memory

- Error: 1258 SQLSTATE: HY000 (ER_ZLIB_Z_BUF_ERROR)

Message: ZLIB: Not enough room in the output buffer (probably, length of uncompressed data was corrupted)

- Error: 1259 SQLSTATE: HY000 (ER_ZLIB_Z_DATA_ERROR)

Message: ZLIB: Input data corrupted

- Error: 1260 SQLSTATE: HY000 (ER_CUT_VALUE_GROUP_CONCAT)

Message: Row %u was cut by GROUP_CONCAT()

- Error: 1261 SQLSTATE: 01000 (ER_WARN_TOO_FEW_RECORDS)

Message: Row %ld doesn't contain data for all columns

- Error: 1262 SQLSTATE: 01000 (ER_WARN_TOO_MANY_RECORDS)

Message: Row %ld was truncated; it contained more data than there were input columns

- Error: 1263 SQLSTATE: 22004 (ER_WARN_NULL_TO_NOTNULL)

Message: Column set to default value; NULL supplied to NOT NULL column '%s' at row %ld

- Error: 1264 SQLSTATE: 22003 (ER_WARN_DATA_OUT_OF_RANGE)

Message: Out of range value for column '%s' at row %ld

- Error: 1265 SQLSTATE: 01000 (WARN_DATA_TRUNCATED)

Message: Data truncated for column '%s' at row %ld

- Error: 1266 SQLSTATE: HY000 (ER_WARN_USING_OTHER_HANDLER)

Message: Using storage engine %s for table '%s'

- Error: 1267 SQLSTATE: HY000 (ER_CANT_AGGREGATE_2COLLATIONS)

Message: Illegal mix of collations (%s,%s) and (%s,%s) for operation '%s'

- Error: 1268 SQLSTATE: HY000 (ER_DROP_USER)

Message: Cannot drop one or more of the requested users

- Error: 1269 SQLSTATE: HY000 (ER_REVOKE_GRANTS)

Message: Can't revoke all privileges for one or more of the requested users

- Error: 1270 SQLSTATE: HY000 (ER_CANT_AGGREGATE_3COLLATIONS)

Message: Illegal mix of collations (%s,%s), (%s,%s), (%s,%s) for operation '%s'

- Error: 1271 SQLSTATE: HY000 (ER_CANT_AGGREGATE_NCOLLATIONS)

Message: Illegal mix of collations for operation '%s'

- Error: 1272 SQLSTATE: HY000 (ER_VARIABLE_IS_NOT_STRUCT)

Message: Variable '%s' is not a variable component (can't be used as XXXX.variable_name)

- Error: 1273 SQLSTATE: HY000 (ER_UNKNOWN_COLLATION)

Message: Unknown collation: '%s'

- Error: 1274 SQLSTATE: HY000 (ER_SLAVE_IGNORED_SSL_PARAMS)

Message: SSL parameters in CHANGE MASTER are ignored because this MySQL slave was compiled without SSL support; they can be used later if MySQL slave with SSL is started

- Error: 1275 SQLSTATE: HY000 (ER_SERVER_IS_IN_SECURE_AUTH_MODE)

Message: Server is running in --secure-auth mode, but '%s'@'%s' has a password in the old format; please change the password to the new format

- Error: 1276 SQLSTATE: HY000 (ER_WARN_FIELD_RESOLVED)

Message: Field or reference '%s%s%s%s%s' of SELECT #%d was resolved in SELECT #%d

- Error: 1277 SQLSTATE: HY000 (ER_BAD_SLAVE_UNTIL_COND)

Message: Incorrect parameter or combination of parameters for START SLAVE UNTIL

- Error: 1278 SQLSTATE: HY000 (ER_MISSING_SKIP_SLAVE)

Message: It is recommended to use --skip-slave-start when doing step-by-step replication with START SLAVE UNTIL; otherwise, you will get problems if you get an unexpected slave's mysqld restart

- Error: 1279 SQLSTATE: HY000 (ER_UNTIL_COND_IGNORED)

Message: SQL thread is not to be started so UNTIL options are ignored

- Error: 1280 SQLSTATE: 42000 (ER_WRONG_NAME_FOR_INDEX)

Message: Incorrect index name '%s'

- Error: 1281 SQLSTATE: 42000 (ER_WRONG_NAME_FOR_CATALOG)

Message: Incorrect catalog name '%s'

- Error: 1282 SQLSTATE: HY000 (ER_WARN_QC_RESIZE)

Message: Query cache failed to set size %lu; new query cache size is %lu

- Error: 1283 SQLSTATE: HY000 (ER_BAD_FT_COLUMN)

Message: Column '%s' cannot be part of FULLTEXT index

- Error: 1284 SQLSTATE: HY000 (ER_UNKNOWN_KEY_CACHE)

Message: Unknown key cache '%s'

- Error: 1285 SQLSTATE: HY000 (ER_WARN_HOSTNAME_WONT_WORK)

Message: MySQL is started in --skip-name-resolve mode; you must restart it without this switch for this grant to work

- Error: 1286 SQLSTATE: 42000 (ER_UNKNOWN_STORAGE_ENGINE)

Message: Unknown storage engine '%s'

- Error: 1287 SQLSTATE: HY000 (ER_WARN_DEPRECATED_SYNTAX)

Message: '%s' is deprecated and will be removed in a future release. Please use %s instead

- Error: 1288 SQLSTATE: HY000 (ER_NON_UPDATABLE_TABLE)

Message: The target table %s of the %s is not updatable

- Error: 1289 SQLSTATE: HY000 (ER_FEATURE_DISABLED)

Message: The '%s' feature is disabled; you need MySQL built with '%s' to have it working

- Error: 1290 SQLSTATE: HY000 (ER_OPTION_PREVENTS_STATEMENT)

Message: The MySQL server is running with the %s option so it cannot execute this statement

- Error: 1291 SQLSTATE: HY000 (ER_DUPLICATED_VALUE_IN_TYPE)

Message: Column '%s' has duplicated value '%s' in %s

- Error: 1292 SQLSTATE: 22007 (ER_TRUNCATED_WRONG_VALUE)

Message: Truncated incorrect %s value: '%s'

- Error: 1293 SQLSTATE: HY000 (ER_TOO_MUCH_AUTO_TIMESTAMP_COLS)

Message: Incorrect table definition; there can be only one TIMESTAMP column with CURRENT_TIMESTAMP in DEFAULT or ON UPDATE clause

- Error: 1294 SQLSTATE: HY000 (ER_INVALID_ON_UPDATE)

Message: Invalid ON UPDATE clause for '%s' column

- Error: 1295 SQLSTATE: HY000 (ER_UNSUPPORTED_PS)

Message: This command is not supported in the prepared statement protocol yet

- Error: 1296 SQLSTATE: HY000 (ER_GET_ERRMSG)

Message: Got error %d '%s' from %s

- Error: 1297 SQLSTATE: HY000 (ER_GET_TEMPORARY_ERRMSG)

Message: Got temporary error %d '%s' from %s

- Error: 1298 SQLSTATE: HY000 (ER_UNKNOWN_TIME_ZONE)

Message: Unknown or incorrect time zone: '%s'

- Error: 1299 SQLSTATE: HY000 (ER_WARN_INVALID_TIMESTAMP)

Message: Invalid TIMESTAMP value in column '%s' at row %ld

- Error: 1300 SQLSTATE: HY000 (ER_INVALID_CHARACTER_STRING)

Message: Invalid %s character string: '%s'

- Error: 1301 SQLSTATE: HY000 (ER_WARN_ALLOWED_PACKET_OVERFLOWED)

Message: Result of %s() was larger than max_allowed_packet (%ld) - truncated

- Error: 1302 SQLSTATE: HY000 (ER_CONFLICTING_DECLARATIONS)

Message: Conflicting declarations: '%s%s' and '%s%s'

- Error: 1303 SQLSTATE: 2F003 (ER_SP_NO_RECURSIVE_CREATE)

Message: Can't create a %s from within another stored routine

- Error: 1304 SQLSTATE: 42000 (ER_SP_ALREADY_EXISTS)

Message: %s %s already exists

- Error: 1305 SQLSTATE: 42000 (ER_SP_DOES_NOT_EXIST)

Message: %s %s does not exist

- Error: 1306 SQLSTATE: HY000 (ER_SP_DROP_FAILED)

Message: Failed to DROP %s %s

- Error: 1307 SQLSTATE: HY000 (ER_SP_STORE_FAILED)

Message: Failed to CREATE %s %s

- Error: 1308 SQLSTATE: 42000 (ER_SP_LILABEL_MISMATCH)

Message: %s with no matching label: %s

- Error: 1309 SQLSTATE: 42000 (ER_SP_LABEL_REDEFINE)

Message: Redefining label %s

- Error: 1310 SQLSTATE: 42000 (ER_SP_LABEL_MISMATCH)

Message: End-label %s without match

- Error: 1311 SQLSTATE: 01000 (ER_SP_UNINIT_VAR)

Message: Referring to uninitialized variable %s

- Error: 1312 SQLSTATE: 0A000 (ER_SP_BADSELECT)

Message: PROCEDURE %s can't return a result set in the given context

- Error: 1313 SQLSTATE: 42000 (ER_SP_BADRETURN)

Message: RETURN is only allowed in a FUNCTION

- Error: 1314 SQLSTATE: 0A000 (ER_SP_BADSTATEMENT)

Message: %s is not allowed in stored procedures

- Error: 1315 SQLSTATE: 42000 (ER_UPDATE_LOG_DEPRECATED_IGNORED)

Message: The update log is deprecated and replaced by the binary log; SET SQL_LOG_UPDATE has been ignored.

- Error: 1316 SQLSTATE: 42000 (ER_UPDATE_LOG_DEPRECATED_TRANSLATED)

Message: The update log is deprecated and replaced by the binary log; SET SQL_LOG_UPDATE has been translated to SET SQL_LOG_BIN.

- Error: 1317 SQLSTATE: 70100 (ER_QUERY_INTERRUPTED)

Message: Query execution was interrupted

- Error: 1318 SQLSTATE: 42000 (ER_SP_WRONG_NO_OF_ARGS)

Message: Incorrect number of arguments for %s %s; expected %u, got %u

- Error: 1319 SQLSTATE: 42000 (ER_SP_COND_MISMATCH)

Message: Undefined CONDITION: %s

- Error: 1320 SQLSTATE: 42000 (ER_SP_NORETURN)

Message: No RETURN found in FUNCTION %s

- Error: 1321 SQLSTATE: 2F005 (ER_SP_NORETURNEND)

Message: FUNCTION %s ended without RETURN

- Error: 1322 SQLSTATE: 42000 (ER_SP_BAD_CURSOR_QUERY)

Message: Cursor statement must be a SELECT

- Error: 1323 SQLSTATE: 42000 (ER_SP_BAD_CURSOR_SELECT)

Message: Cursor SELECT must not have INTO

- Error: 1324 SQLSTATE: 42000 (ER_SP_CURSOR_MISMATCH)

Message: Undefined CURSOR: %s

- Error: 1325 SQLSTATE: 24000 (ER_SP_CURSOR_ALREADY_OPEN)

Message: Cursor is already open

- Error: 1326 SQLSTATE: 24000 (ER_SP_CURSOR_NOT_OPEN)

Message: Cursor is not open

- Error: 1327 SQLSTATE: 42000 (ER_SP_UNDECLARED_VAR)

Message: Undeclared variable: %s

- Error: 1328 SQLSTATE: HY000 (ER_SP_WRONG_NO_OF_FETCH_ARGS)

Message: Incorrect number of FETCH variables

- Error: 1329 SQLSTATE: 02000 (ER_SP_FETCH_NO_DATA)

Message: No data - zero rows fetched, selected, or processed

- Error: 1330 SQLSTATE: 42000 (ER_SP_DUP_PARAM)

Message: Duplicate parameter: %s

- Error: 1331 SQLSTATE: 42000 (ER_SP_DUP_VAR)

Message: Duplicate variable: %s

- Error: 1332 SQLSTATE: 42000 (ER_SP_DUP_COND)

Message: Duplicate condition: %s

- Error: 1333 SQLSTATE: 42000 (ER_SP_DUP_CURS)

Message: Duplicate cursor: %s

- Error: 1334 SQLSTATE: HY000 (ER_SP_CANT_ALTER)

Message: Failed to ALTER %s %s

- Error: 1335 SQLSTATE: 0A000 (ER_SP_SUBSELECT_NYI)

Message: Subquery value not supported

- Error: 1336 SQLSTATE: 0A000 (ER_STMT_NOT_ALLOWED_IN_SF_OR_TRG)

Message: %s is not allowed in stored function or trigger

- Error: 1337 SQLSTATE: 42000 (ER_SP_VARCOND_AFTER_CURSHNDLR)

Message: Variable or condition declaration after cursor or handler declaration

- Error: 1338 SQLSTATE: 42000 (ER_SP_CURSOR_AFTER_HANDLER)

Message: Cursor declaration after handler declaration

- Error: 1339 SQLSTATE: 20000 (ER_SP_CASE_NOT_FOUND)

Message: Case not found for CASE statement

- Error: 1340 SQLSTATE: HY000 (ER_FPARSER_TOO_BIG_FILE)

Message: Configuration file '%s' is too big

- Error: 1341 SQLSTATE: HY000 (ER_FPARSER_BAD_HEADER)

Message: Malformed file type header in file '%s'

- Error: 1342 SQLSTATE: HY000 (ER_FPARSER_EOF_IN_COMMENT)

Message: Unexpected end of file while parsing comment '%s'

- Error: 1343 SQLSTATE: HY000 (ER_FPARSER_ERROR_IN_PARAMETER)

Message: Error while parsing parameter '%s' (line: '%s')

- Error: 1344 SQLSTATE: HY000 (ER_FPARSER_EOF_IN_UNKNOWN_PARAMETER)

Message: Unexpected end of file while skipping unknown parameter '%s'

- Error: 1345 SQLSTATE: HY000 (ER_VIEW_NO_EXPLAIN)

Message: EXPLAIN/SHOW can not be issued; lacking privileges for underlying table

- Error: 1346 SQLSTATE: HY000 (ER_FRM_UNKNOWN_TYPE)

Message: File '%s' has unknown type '%s' in its header

- Error: 1347 SQLSTATE: HY000 (ER_WRONG_OBJECT)

Message: '%s.%s' is not %s

The named object is incorrect for the type of operation attempted on it. It must be an object of the named type.

- Error: 1348 SQLSTATE: HY000 (ER_NONUPDATEABLE_COLUMN)

Message: Column '%s' is not updatable

- Error: 1349 SQLSTATE: HY000 (ER_VIEW_SELECT_DERIVED)

Message: View's SELECT contains a subquery in the FROM clause

ER_VIEW_SELECT_DERIVED was removed after 5.7.6.

- Error: 1349 SQLSTATE: HY000 (ER_VIEW_SELECT_DERIVED_UNUSED)

Message: View's SELECT contains a subquery in the FROM clause

ER_VIEW_SELECT_DERIVED_UNUSED was added in 5.7.7.

- Error: 1350 SQLSTATE: HY000 (ER_VIEW_SELECT_CLAUSE)

Message: View's SELECT contains a '%s' clause

- Error: 1351 SQLSTATE: HY000 (ER_VIEW_SELECT_VARIABLE)

Message: View's SELECT contains a variable or parameter

- Error: 1352 SQLSTATE: HY000 (ER_VIEW_SELECT_TMPTABLE)

Message: View's SELECT refers to a temporary table '%s'

- Error: 1353 SQLSTATE: HY000 (ER_VIEW_WRONG_LIST)

Message: View's SELECT and view's field list have different column counts

- Error: 1354 SQLSTATE: HY000 (ER_WARN_VIEW_MERGE)

Message: View merge algorithm can't be used here for now (assumed undefined algorithm)

- Error: 1355 SQLSTATE: HY000 (ER_WARN_VIEW_WITHOUT_KEY)

Message: View being updated does not have complete key of underlying table in it

- Error: 1356 SQLSTATE: HY000 (ER_VIEW_INVALID)

Message: View '%s.%s' references invalid table(s) or column(s) or function(s) or definer/invoker of view lack rights to use them

- Error: 1357 SQLSTATE: HY000 (ER_SP_NO_DROP_SP)

Message: Can't drop or alter a %s from within another stored routine

- Error: 1358 SQLSTATE: HY000 (ER_SP_GOTO_IN_HNDLR)

Message: GOTO is not allowed in a stored procedure handler

- Error: 1359 SQLSTATE: HY000 (ER_TRG_ALREADY_EXISTS)

Message: Trigger already exists

- Error: 1360 SQLSTATE: HY000 (ER_TRG_DOES_NOT_EXIST)

Message: Trigger does not exist

- Error: 1361 SQLSTATE: HY000 (ER_TRG_ON_VIEW_OR_TEMP_TABLE)

Message: Trigger's '%s' is view or temporary table

- Error: 1362 SQLSTATE: HY000 (ER_TRG_CANT_CHANGE_ROW)

Message: Updating of %s row is not allowed in %strigger

- Error: 1363 SQLSTATE: HY000 (ER_TRG_NO_SUCH_ROW_IN_TRG)

Message: There is no %s row in %s trigger

- Error: 1364 SQLSTATE: HY000 (ER_NO_DEFAULT_FOR_FIELD)

Message: Field '%s' doesn't have a default value

- Error: 1365 SQLSTATE: 22012 (ER_DIVISION_BY_ZERO)

Message: Division by 0

- Error: 1366 SQLSTATE: HY000 (ER_TRUNCATED_WRONG_VALUE_FOR_FIELD)

Message: Incorrect %s value: '%s' for column '%s' at row %ld

- Error: 1367 SQLSTATE: 22007 (ER_ILLEGAL_VALUE_FOR_TYPE)

Message: Illegal %s '%s' value found during parsing

- Error: 1368 SQLSTATE: HY000 (ER_VIEW_NONUPD_CHECK)

Message: CHECK OPTION on non-updatable view '%s.%s'

- Error: 1369 SQLSTATE: HY000 (ER_VIEW_CHECK_FAILED)

Message: CHECK OPTION failed '%s.%s'

- Error: 1370 SQLSTATE: 42000 (ER_PROCACCESS_DENIED_ERROR)

Message: %s command denied to user '%s'@'%s' for routine '%s'

- Error: 1371 SQLSTATE: HY000 (ER_RELAY_LOG_FAIL)

Message: Failed purging old relay logs: %s

- Error: 1372 SQLSTATE: HY000 (ER_PASSWD_LENGTH)

Message: Password hash should be a %d-digit hexadecimal number

- Error: 1373 SQLSTATE: HY000 (ER_UNKNOWN_TARGET_BINLOG)

Message: Target log not found in binlog index

- Error: 1374 SQLSTATE: HY000 (ER_IO_ERR_LOG_INDEX_READ)

Message: I/O error reading log index file

- Error: 1375 SQLSTATE: HY000 (ER_BINLOG_PURGE_PROHIBITED)

Message: Server configuration does not permit binlog purge

- Error: 1376 SQLSTATE: HY000 (ER_FSEEK_FAIL)

Message: Failed on fseek()

- Error: 1377 SQLSTATE: HY000 (ER_BINLOG_PURGE_FATAL_ERR)

Message: Fatal error during log purge

- Error: 1378 SQLSTATE: HY000 (ER_LOG_IN_USE)

Message: A purgeable log is in use, will not purge

- Error: 1379 SQLSTATE: HY000 (ER_LOG_PURGE_UNKNOWN_ERR)

Message: Unknown error during log purge

- Error: 1380 SQLSTATE: HY000 (ER_RELAY_LOG_INIT)

Message: Failed initializing relay log position: %s

- Error: 1381 SQLSTATE: HY000 (ER_NO_BINARY_LOGGING)

Message: You are not using binary logging

- Error: 1382 SQLSTATE: HY000 (ER_RESERVED_SYNTAX)

Message: The '%s' syntax is reserved for purposes internal to the MySQL server

- Error: 1383 SQLSTATE: HY000 (ER_WSAS_FAILED)

Message: WSAStartup Failed

- Error: 1384 SQLSTATE: HY000 (ER_DIFF_GROUPS_PROC)

Message: Can't handle procedures with different groups yet

- Error: 1385 SQLSTATE: HY000 (ER_NO_GROUP_FOR_PROC)

Message: Select must have a group with this procedure

- Error: 1386 SQLSTATE: HY000 (ER_ORDER_WITH_PROC)

Message: Can't use ORDER clause with this procedure

- Error: 1387 SQLSTATE: HY000 (ER_LOGGING_PROHIBIT_CHANGING_OF)

Message: Binary logging and replication forbid changing the global server %s

- Error: 1388 SQLSTATE: HY000 (ER_NO_FILE_MAPPING)

Message: Can't map file: %s, errno: %d

- Error: 1389 SQLSTATE: HY000 (ER_WRONG_MAGIC)

Message: Wrong magic in %s

- Error: 1390 SQLSTATE: HY000 (ER_PS_MANY_PARAM)

Message: Prepared statement contains too many placeholders

- Error: 1391 SQLSTATE: HY000 (ER_KEY_PART_0)

Message: Key part '%s' length cannot be 0

- Error: 1392 SQLSTATE: HY000 (ER_VIEW_CHECKSUM)

Message: View text checksum failed

- Error: 1393 SQLSTATE: HY000 (ER_VIEW_MULTIUPDATE)

Message: Can not modify more than one base table through a join view '%s.%s'

- Error: 1394 SQLSTATE: HY000 (ER_VIEW_NO_INSERT_FIELD_LIST)

Message: Can not insert into join view '%s.%s' without fields list

- Error: 1395 SQLSTATE: HY000 (ER_VIEW_DELETE_MERGE_VIEW)

Message: Can not delete from join view '%s.%s'

- Error: 1396 SQLSTATE: HY000 (ER_CANNOT_USER)

Message: Operation %s failed for %s

- Error: 1397 SQLSTATE: XAE04 (ER_XAER_NOTA)

Message: XAER_NOTA: Unknown XID

- Error: 1398 SQLSTATE: XAE05 (ER_XAER_INVAL)

Message: XAER_INVAL: Invalid arguments (or unsupported command)

- Error: 1399 SQLSTATE: XAE07 (ER_XAER_RMFAIL)

Message: XAER_RMFAIL: The command cannot be executed when global transaction is in the %s state

- Error: 1400 SQLSTATE: XAE09 (ER_XAER_OUTSIDE)

Message: XAER_OUTSIDE: Some work is done outside global transaction

- Error: 1401 SQLSTATE: XAE03 (ER_XAER_RMERR)

Message: XAER_RMERR: Fatal error occurred in the transaction branch - check your data for consistency

- Error: 1402 SQLSTATE: XA100 (ER_XA_RBROLLBACK)

Message: XA_RBROLLBACK: Transaction branch was rolled back

- Error: 1403 SQLSTATE: 42000 (ER_NONEXISTING_PROC_GRANT)

Message: There is no such grant defined for user '%s' on host '%s' on routine '%s'

- Error: 1404 SQLSTATE: HY000 (ER_PROC_AUTO_GRANT_FAIL)

Message: Failed to grant EXECUTE and ALTER ROUTINE privileges

- Error: 1405 SQLSTATE: HY000 (ER_PROC_AUTO_REVOKE_FAIL)

Message: Failed to revoke all privileges to dropped routine

- Error: 1406 SQLSTATE: 22001 (ER_DATA_TOO_LONG)

Message: Data too long for column '%s' at row %ld

- Error: 1407 SQLSTATE: 42000 (ER_SP_BAD_SQLSTATE)

Message: Bad SQLSTATE: '%s'

- Error: 1408 SQLSTATE: HY000 (ER_STARTUP)

Message: %s: ready for connections. Version: '%s' socket: '%s' port: %d %s

- Error: 1409 SQLSTATE: HY000 (ER_LOAD_FROM_FIXED_SIZE_ROWS_TO_VAR)

Message: Can't load value from file with fixed size rows to variable

- Error: 1410 SQLSTATE: 42000 (ER_CANT_CREATE_USER_WITH_GRANT)

Message: You are not allowed to create a user with GRANT

- Error: 1411 SQLSTATE: HY000 (ER_WRONG_VALUE_FOR_TYPE)

Message: Incorrect %s value: '%s' for function %s

- Error: 1412 SQLSTATE: HY000 (ER_TABLE_DEF_CHANGED)

Message: Table definition has changed, please retry transaction

- Error: 1413 SQLSTATE: 42000 (ER_SP_DUP_HANDLER)

Message: Duplicate handler declared in the same block

- Error: 1414 SQLSTATE: 42000 (ER_SP_NOT_VAR_ARG)

Message: OUT or INOUT argument %d for routine %s is not a variable or NEW pseudo-variable in BEFORE trigger

- Error: 1415 SQLSTATE: 0A000 (ER_SP_NO_RETSET)

Message: Not allowed to return a result set from a %s

- Error: 1416 SQLSTATE: 22003 (ER_CANT_CREATE_GEOMETRY_OBJECT)

Message: Cannot get geometry object from data you send to the GEOMETRY field

- Error: 1417 SQLSTATE: HY000 (ER_FAILED_ROUTINE_BREAK_BINLOG)

Message: A routine failed and has neither NO SQL nor READS SQL DATA in its declaration and binary logging is enabled; if non-transactional tables were updated, the binary log will miss their changes

- Error: 1418 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_ROUTINE)

Message: This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)

- Error: 1419 SQLSTATE: HY000 (ER_BINLOG_CREATE_ROUTINE_NEED_SUPER)

Message: You do not have the SUPER privilege and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)

- Error: 1420 SQLSTATE: HY000 (ER_EXEC_STMT_WITH_OPEN_CURSOR)

Message: You can't execute a prepared statement which has an open cursor associated with it. Reset the statement to re-execute it.

- Error: 1421 SQLSTATE: HY000 (ER_STMT_HAS_NO_OPEN_CURSOR)

Message: The statement (%lu) has no open cursor.

- Error: 1422 SQLSTATE: HY000 (ER_COMMIT_NOT_ALLOWED_IN_SF_OR_TRG)

Message: Explicit or implicit commit is not allowed in stored function or trigger.

- Error: 1423 SQLSTATE: HY000 (ER_NO_DEFAULT_FOR_VIEW_FIELD)

Message: Field of view '%s.%s' underlying table doesn't have a default value

- Error: 1424 SQLSTATE: HY000 (ER_SP_NO_RECURSION)

Message: Recursive stored functions and triggers are not allowed.

- Error: 1425 SQLSTATE: 42000 (ER_TOO_BIG_SCALE)

Message: Too big scale %d specified for column '%s'. Maximum is %lu.

- Error: 1426 SQLSTATE: 42000 (ER_TOO_BIG_PRECISION)

Message: Too-big precision %d specified for '%s'. Maximum is %lu.

- Error: 1427 SQLSTATE: 42000 (ER_M_BIGGER_THAN_D)

Message: For float(M,D), double(M,D) or decimal(M,D), M must be >= D (column '%s').

- Error: 1428 SQLSTATE: HY000 (ER_WRONG_LOCK_OF_SYSTEM_TABLE)

Message: You can't combine write-locking of system tables with other tables or lock types

- Error: 1429 SQLSTATE: HY000 (ER_CONNECT_TO_FOREIGN_DATA_SOURCE)

Message: Unable to connect to foreign data source: %s

- Error: 1430 SQLSTATE: HY000 (ER_QUERY_ON_FOREIGN_DATA_SOURCE)

Message: There was a problem processing the query on the foreign data source. Data source - Error: %s

- Error: 1431 SQLSTATE: HY000 (ER_FOREIGN_DATA_SOURCE_DOESNT_EXIST)

Message: The foreign data source you are trying to reference does not exist. Data source - Error: %s

- Error: 1432 SQLSTATE: HY000 (ER_FOREIGN_DATA_STRING_INVALID_CANT_CREATE)

Message: Can't create federated table. The data source connection string '%s' is not in the correct format

- Error: 1433 SQLSTATE: HY000 (ER_FOREIGN_DATA_STRING_INVALID)

Message: The data source connection string '%s' is not in the correct format

- Error: 1434 SQLSTATE: HY000 (ER_CANT_CREATE_FEDERATED_TABLE)

Message: Can't create federated table. Foreign data src - Error: %s

- Error: 1435 SQLSTATE: HY000 (ER_TRG_IN_WRONG_SCHEMA)

Message: Trigger in wrong schema

- Error: 1436 SQLSTATE: HY000 (ER_STACK_OVERRUN_NEED_MORE)

Message: Thread stack overrun: %ld bytes used of a %ld byte stack, and %ld bytes needed. Use 'mysqld --thread_stack=#' to specify a bigger stack.

- Error: 1437 SQLSTATE: 42000 (ER_TOO_LONG_BODY)

Message: Routine body for '%s' is too long

- Error: 1438 SQLSTATE: HY000 (ER_WARN_CANT_DROP_DEFAULT_KEYCACHE)

Message: Cannot drop default keycache

- Error: 1439 SQLSTATE: 42000 (ER_TOO_BIG_DISPLAYWIDTH)

Message: Display width out of range for column '%s' (max = %lu)

- Error: 1440 SQLSTATE: XAE08 (ER_XAER_DUPID)

Message: XAER_DUPID: The XID already exists

- Error: 1441 SQLSTATE: 22008 (ER_DATETIME_FUNCTION_OVERFLOW)

Message: Datetime function: %s field overflow

- Error: 1442 SQLSTATE: HY000 (ER_CANT_UPDATE_USED_TABLE_IN_SF_OR_TRG)

Message: Can't update table '%s' in stored function/trigger because it is already used by statement which invoked this stored function/trigger.

- Error: 1443 SQLSTATE: HY000 (ER_VIEW_PREVENT_UPDATE)

Message: The definition of table '%s' prevents operation %s on table '%s'.

- Error: 1444 SQLSTATE: HY000 (ER_PS_NO_RECURSION)

Message: The prepared statement contains a stored routine call that refers to that same statement. It's not allowed to execute a prepared statement in such a recursive manner

- Error: 1445 SQLSTATE: HY000 (ER_SP_CANT_SET_AUTOCOMMIT)

Message: Not allowed to set autocommit from a stored function or trigger

- Error: 1446 SQLSTATE: HY000 (ER_MALFORMED_DEFINER)

Message: Definer is not fully qualified

- Error: 1447 SQLSTATE: HY000 (ER_VIEW_FRM_NO_USER)

Message: View '%s'.'%s' has no definer information (old table format). Current user is used as definer. Please recreate the view!

- Error: 1448 SQLSTATE: HY000 (ER_VIEW_OTHER_USER)

Message: You need the SUPER privilege for creation view with '%s'@'%s' definer

- Error: 1449 SQLSTATE: HY000 (ER_NO_SUCH_USER)

Message: The user specified as a definer ('%s'@'%s') does not exist

- Error: 1450 SQLSTATE: HY000 (ER_FORBID_SCHEMA_CHANGE)

Message: Changing schema from '%s' to '%s' is not allowed.

- Error: 1451 SQLSTATE: 23000 (ER_ROW_IS_REFERENCED_2)

Message: Cannot delete or update a parent row: a foreign key constraint fails (%s)

InnoDB reports this error when you try to delete a parent row that has children, and a foreign key constraint fails. Delete the children first.

- Error: 1452 SQLSTATE: 23000 (ER_NO_REFERENCED_ROW_2)

Message: Cannot add or update a child row: a foreign key constraint fails (%s)

InnoDB reports this error when you try to add a row but there is no parent row, and a foreign key constraint fails. Add the parent row first.

- Error: 1453 SQLSTATE: 42000 (ER_SP_BAD_VAR_SHADOW)

Message: Variable '%s' must be quoted with `...`, or renamed

- Error: 1454 SQLSTATE: HY000 (ER_TRG_NO_DEFINER)

Message: No definer attribute for trigger '%s'.'%s'. The trigger will be activated under the authorization of the invoker, which may have insufficient privileges. Please recreate the trigger.

- Error: 1455 SQLSTATE: HY000 (ER_OLD_FILE_FORMAT)

Message: '%s' has an old format, you should re-create the '%s' object(s)

- Error: 1456 SQLSTATE: HY000 (ER_SP_RECURSION_LIMIT)

Message: Recursive limit %d (as set by the max_sp_recursion_depth variable) was exceeded for routine %s

- Error: 1457 SQLSTATE: HY000 (ER_SP_PROC_TABLE_CORRUPT)

Message: Failed to load routine %s. The table mysql.proc is missing, corrupt, or contains bad data (internal code %d)

- Error: 1458 SQLSTATE: 42000 (ER_SP_WRONG_NAME)

Message: Incorrect routine name '%s'

- Error: 1459 SQLSTATE: HY000 (ER_TABLE_NEEDS_UPGRADE)

Message: Table upgrade required. Please do "REPAIR TABLE `%s`" or dump/reload to fix it!

- Error: 1460 SQLSTATE: 42000 (ER_SP_NO_AGGREGATE)

Message: AGGREGATE is not supported for stored functions

- Error: 1461 SQLSTATE: 42000 (ER_MAX_PREPARED_STMT_COUNT_REACHED)

Message: Can't create more than max_prepared_stmt_count statements (current value: %lu)

- Error: 1462 SQLSTATE: HY000 (ER_VIEW_RECURSIVE)

Message: `%s`.`%s` contains view recursion

- Error: 1463 SQLSTATE: 42000 (ER_NON_GROUPING_FIELD_USED)

Message: Non-grouping field '%s' is used in %s clause

- Error: 1464 SQLSTATE: HY000 (ER_TABLE_CANT_HANDLE_SPKEYS)

Message: The used table type doesn't support SPATIAL indexes

- Error: 1465 SQLSTATE: HY000 (ER_NO_TRIGGERS_ON_SYSTEM_SCHEMA)

Message: Triggers can not be created on system tables

- Error: 1466 SQLSTATE: HY000 (ER_REMOVED_SPACES)

Message: Leading spaces are removed from name '%s'

- Error: 1467 SQLSTATE: HY000 (ER_AUTOINC_READ_FAILED)

Message: Failed to read auto-increment value from storage engine

- Error: 1468 SQLSTATE: HY000 (ER_USERNAME)

Message: user name

- Error: 1469 SQLSTATE: HY000 (ER_HOSTNAME)

Message: host name

- Error: 1470 SQLSTATE: HY000 (ER_WRONG_STRING_LENGTH)

Message: String '%s' is too long for %s (should be no longer than %d)

- Error: 1471 SQLSTATE: HY000 (ER_NON_INSERTABLE_TABLE)

Message: The target table %s of the %s is not insertable-into

- Error: 1472 SQLSTATE: HY000 (ER_ADMIN_WRONG_MRG_TABLE)

Message: Table '%s' is differently defined or of non-MyISAM type or doesn't exist

- Error: 1473 SQLSTATE: HY000 (ER_TOO_HIGH_LEVEL_OF_NESTING_FOR_SELECT)

Message: Too high level of nesting for select

- Error: 1474 SQLSTATE: HY000 (ER_NAME_BECOMES_EMPTY)

Message: Name '%s' has become ''

- Error: 1475 SQLSTATE: HY000 (ER_AMBIGUOUS_FIELD_TERM)

Message: First character of the FIELDS TERMINATED string is ambiguous; please use non-optional and non-empty FIELDS ENCLOSED BY

- Error: 1476 SQLSTATE: HY000 (ER_FOREIGN_SERVER_EXISTS)

Message: The foreign server, %s, you are trying to create already exists.

- Error: 1477 SQLSTATE: HY000 (ER_FOREIGN_SERVER_DOESNT_EXIST)

Message: The foreign server name you are trying to reference does not exist. Data source - Error: %s

- Error: 1478 SQLSTATE: HY000 (ER_ILLEGAL_HA_CREATE_OPTION)

Message: Table storage engine '%s' does not support the create option '%s'

- Error: 1479 SQLSTATE: HY000 (ER_PARTITION_REQUIRES_VALUES_ERROR)

Message: Syntax - Error: %s PARTITIONING requires definition of VALUES %s for each partition

- Error: 1480 SQLSTATE: HY000 (ER_PARTITION_WRONG_VALUES_ERROR)

Message: Only %s PARTITIONING can use VALUES %s in partition definition

- Error: 1481 SQLSTATE: HY000 (ER_PARTITION_MAXVALUE_ERROR)

Message: MAXVALUE can only be used in last partition definition

- Error: 1482 SQLSTATE: HY000 (ER_PARTITION_SUBPARTITION_ERROR)

Message: Subpartitions can only be hash partitions and by key

- Error: 1483 SQLSTATE: HY000 (ER_PARTITION_SUBPART_MIX_ERROR)

Message: Must define subpartitions on all partitions if on one partition

- Error: 1484 SQLSTATE: HY000 (ER_PARTITION_WRONG_NO_PART_ERROR)

Message: Wrong number of partitions defined, mismatch with previous setting

- Error: 1485 SQLSTATE: HY000 (ER_PARTITION_WRONG_NO_SUBPART_ERROR)

Message: Wrong number of subpartitions defined, mismatch with previous setting

- Error: 1486 SQLSTATE: HY000 (ER_WRONG_EXPR_IN_PARTITION_FUNC_ERROR)

Message: Constant, random or timezone-dependent expressions in (sub)partitioning function are not allowed

- Error: 1487 SQLSTATE: HY000 (ER_NO_CONST_EXPR_IN_RANGE_OR_LIST_ERROR)

Message: Expression in RANGE/LIST VALUES must be constant

- Error: 1488 SQLSTATE: HY000 (ER_FIELD_NOT_FOUND_PART_ERROR)

Message: Field in list of fields for partition function not found in table

- Error: 1489 SQLSTATE: HY000 (ER_LIST_OF_FIELDS_ONLY_IN_HASH_ERROR)

Message: List of fields is only allowed in KEY partitions

- Error: 1490 SQLSTATE: HY000 (ER_INCONSISTENT_PARTITION_INFO_ERROR)

Message: The partition info in the frm file is not consistent with what can be written into the frm file

- Error: 1491 SQLSTATE: HY000 (ER_PARTITION_FUNC_NOT_ALLOWED_ERROR)

Message: The %s function returns the wrong type

- Error: 1492 SQLSTATE: HY000 (ER_PARTITIONS_MUST_BE_DEFINED_ERROR)

Message: For %s partitions each partition must be defined

- Error: 1493 SQLSTATE: HY000 (ER_RANGE_NOT_INCREASING_ERROR)

Message: VALUES LESS THAN value must be strictly increasing for each partition

- Error: 1494 SQLSTATE: HY000 (ER_INCONSISTENT_TYPE_OF_FUNCTIONS_ERROR)

Message: VALUES value must be of same type as partition function

- Error: 1495 SQLSTATE: HY000 (ER_MULTIPLE_DEF_CONST_IN_LIST_PART_ERROR)

Message: Multiple definition of same constant in list partitioning

- Error: 1496 SQLSTATE: HY000 (ER_PARTITION_ENTRY_ERROR)

Message: Partitioning can not be used stand-alone in query

- Error: 1497 SQLSTATE: HY000 (ER_MIX_HANDLER_ERROR)

Message: The mix of handlers in the partitions is not allowed in this version of MySQL

- Error: 1498 SQLSTATE: HY000 (ER_PARTITION_NOT_DEFINED_ERROR)

Message: For the partitioned engine it is necessary to define all %s

- Error: 1499 SQLSTATE: HY000 (ER_TOO_MANY_PARTITIONS_ERROR)

Message: Too many partitions (including subpartitions) were defined

- Error: 1500 SQLSTATE: HY000 (ER_SUBPARTITION_ERROR)

Message: It is only possible to mix RANGE/LIST partitioning with HASH/KEY partitioning for subpartitioning

- Error: 1501 SQLSTATE: HY000 (ER_CANT_CREATE_HANDLER_FILE)

Message: Failed to create specific handler file

- Error: 1502 SQLSTATE: HY000 (ER_BLOB_FIELD_IN_PART_FUNC_ERROR)

Message: A BLOB field is not allowed in partition function

- Error: 1503 SQLSTATE: HY000 (ER_UNIQUE_KEY_NEED_ALL_FIELDS_IN_PF)

Message: A %s must include all columns in the table's partitioning function

- Error: 1504 SQLSTATE: HY000 (ER_NO_PARTS_ERROR)

Message: Number of %s = 0 is not an allowed value

- Error: 1505 SQLSTATE: HY000 (ER_PARTITION_MGMT_ON_NONPARTITIONED)

Message: Partition management on a not partitioned table is not possible

- Error: 1506 SQLSTATE: HY000 (ER_FOREIGN_KEY_ON_PARTITIONED)

Message: Foreign keys are not yet supported in conjunction with partitioning

- Error: 1507 SQLSTATE: HY000 (ER_DROP_PARTITION_NON_EXISTENT)

Message: Error in list of partitions to %s

- Error: 1508 SQLSTATE: HY000 (ER_DROP_LAST_PARTITION)

Message: Cannot remove all partitions, use DROP TABLE instead

- Error: 1509 SQLSTATE: HY000 (ER_COALESCE_ONLY_ON_HASH_PARTITION)

Message: COALESCE PARTITION can only be used on HASH/KEY partitions

- Error: 1510 SQLSTATE: HY000 (ER_REORG_HASH_ONLY_ON_SAME_NO)

Message: REORGANIZE PARTITION can only be used to reorganize partitions not to change their numbers

- Error: 1511 SQLSTATE: HY000 (ER_REORG_NO_PARAM_ERROR)

Message: REORGANIZE PARTITION without parameters can only be used on auto-partitioned tables using HASH PARTITIONs

- Error: 1512 SQLSTATE: HY000 (ER_ONLY_ON_RANGE_LIST_PARTITION)

Message: %s PARTITION can only be used on RANGE/LIST partitions

- Error: 1513 SQLSTATE: HY000 (ER_ADD_PARTITION_SUBPART_ERROR)

Message: Trying to Add partition(s) with wrong number of subpartitions

- Error: 1514 SQLSTATE: HY000 (ER_ADD_PARTITION_NO_NEW_PARTITION)

Message: At least one partition must be added

- Error: 1515 SQLSTATE: HY000 (ER_COALESCE_PARTITION_NO_PARTITION)

Message: At least one partition must be coalesced

- Error: 1516 SQLSTATE: HY000 (ER_REORG_PARTITION_NOT_EXIST)

Message: More partitions to reorganize than there are partitions

- Error: 1517 SQLSTATE: HY000 (ER_SAME_NAME_PARTITION)

Message: Duplicate partition name %s

- Error: 1518 SQLSTATE: HY000 (ER_NO_BINLOG_ERROR)

Message: It is not allowed to shut off binlog on this command

- Error: 1519 SQLSTATE: HY000 (ER_CONSECUTIVE_REORG_PARTITIONS)

Message: When reorganizing a set of partitions they must be in consecutive order

- Error: 1520 SQLSTATE: HY000 (ER_REORG_OUTSIDE_RANGE)

Message: Reorganize of range partitions cannot change total ranges except for last partition where it can extend the range

- Error: 1521 SQLSTATE: HY000 (ER_PARTITION_FUNCTION_FAILURE)

Message: Partition function not supported in this version for this handler

- Error: 1522 SQLSTATE: HY000 (ER_PART_STATE_ERROR)

Message: Partition state cannot be defined from CREATE/ALTER TABLE

- Error: 1523 SQLSTATE: HY000 (ER_LIMITED_PART_RANGE)

Message: The %s handler only supports 32 bit integers in VALUES

- Error: 1524 SQLSTATE: HY000 (ER_PLUGIN_IS_NOT_LOADED)

Message: Plugin '%s' is not loaded

- Error: 1525 SQLSTATE: HY000 (ER_WRONG_VALUE)

Message: Incorrect %s value: '%s'

- Error: 1526 SQLSTATE: HY000 (ER_NO_PARTITION_FOR_GIVEN_VALUE)

Message: Table has no partition for value %s

- Error: 1527 SQLSTATE: HY000 (ER_FILEGROUP_OPTION_ONLY_ONCE)

Message: It is not allowed to specify %s more than once

- Error: 1528 SQLSTATE: HY000 (ER_CREATE_FILEGROUP_FAILED)

Message: Failed to create %s

- Error: 1529 SQLSTATE: HY000 (ER_DROP_FILEGROUP_FAILED)

Message: Failed to drop %s

- Error: 1530 SQLSTATE: HY000 (ER_TABLESPACE_AUTO_EXTEND_ERROR)

Message: The handler doesn't support autoextend of tablespaces

- Error: 1531 SQLSTATE: HY000 (ER_WRONG_SIZE_NUMBER)

Message: A size parameter was incorrectly specified, either number or on the form 10M

- Error: 1532 SQLSTATE: HY000 (ER_SIZE_OVERFLOW_ERROR)

Message: The size number was correct but we don't allow the digit part to be more than 2 billion

- Error: 1533 SQLSTATE: HY000 (ER_ALTER_FILEGROUP_FAILED)

Message: Failed to alter: %s

- Error: 1534 SQLSTATE: HY000 (ER_BINLOG_ROW_LOGGING_FAILED)

Message: Writing one row to the row-based binary log failed

- Error: 1535 SQLSTATE: HY000 (ER_BINLOG_ROW_WRONG_TABLE_DEF)

Message: Table definition on master and slave does not match: %s

- Error: 1536 SQLSTATE: HY000 (ER_BINLOG_ROW_RBR_TO_SBR)

Message: Slave running with --log-slave-updates must use row-based binary logging to be able to replicate row-based binary log events

- Error: 1537 SQLSTATE: HY000 (ER_EVENT_ALREADY_EXISTS)

Message: Event '%s' already exists

- Error: 1538 SQLSTATE: HY000 (ER_EVENT_STORE_FAILED)

Message: Failed to store event %s. Error code %d from storage engine.

- Error: 1539 SQLSTATE: HY000 (ER_EVENT_DOES_NOT_EXIST)

Message: Unknown event '%s'

- Error: 1540 SQLSTATE: HY000 (ER_EVENT_CANT_ALTER)

Message: Failed to alter event '%s'

- Error: 1541 SQLSTATE: HY000 (ER_EVENT_DROP_FAILED)

Message: Failed to drop %s

- Error: 1542 SQLSTATE: HY000 (ER_EVENT_INTERVAL_NOT_POSITIVE_OR_TOO_BIG)

Message: INTERVAL is either not positive or too big

- Error: 1543 SQLSTATE: HY000 (ER_EVENT_ENDS_BEFORE_STARTS)

Message: ENDS is either invalid or before STARTS

- Error: 1544 SQLSTATE: HY000 (ER_EVENT_EXEC_TIME_IN_THE_PAST)

Message: Event execution time is in the past. Event has been disabled

- Error: 1545 SQLSTATE: HY000 (ER_EVENT_OPEN_TABLE_FAILED)

Message: Failed to open mysql.event

- Error: 1546 SQLSTATE: HY000 (ER_EVENT_NEITHER_M_EXPR_NOR_M_AT)

Message: No datetime expression provided

- Error: 1547 SQLSTATE: HY000 (ER_OBSOLETE_COL_COUNT_DOESNT_MATCH_CORRUPTED)

Message: Column count of mysql.%s is wrong. Expected %d, found %d. The table is probably corrupted

- Error: 1548 SQLSTATE: HY000 (ER_OBSOLETE_CANNOT_LOAD_FROM_TABLE)

Message: Cannot load from mysql.%s. The table is probably corrupted

- Error: 1549 SQLSTATE: HY000 (ER_EVENT_CANNOT_DELETE)

Message: Failed to delete the event from mysql.event

- Error: 1550 SQLSTATE: HY000 (ER_EVENT_COMPILE_ERROR)

Message: Error during compilation of event's body

- Error: 1551 SQLSTATE: HY000 (ER_EVENT_SAME_NAME)

Message: Same old and new event name

- Error: 1552 SQLSTATE: HY000 (ER_EVENT_DATA_TOO_LONG)

Message: Data for column '%s' too long

- Error: 1553 SQLSTATE: HY000 (ER_DROP_INDEX_FK)

Message: Cannot drop index '%s': needed in a foreign key constraint

InnoDB reports this error when you attempt to drop the last index that can enforce a particular referential constraint.

For optimal performance with DML statements, InnoDB requires an index to exist on foreign key columns, so that UPDATE and DELETE operations on a parent table can easily check whether corresponding rows exist in the child table. MySQL creates or drops such indexes automatically when needed, as a side-effect of CREATE TABLE, CREATE INDEX, and ALTER TABLE statements.

When you drop an index, InnoDB checks if the index is used for checking a foreign key constraint. It is still OK to drop the index if there is another index that can be used to enforce the same constraint. InnoDB prevents you from dropping the last index that can enforce a particular referential constraint.

- Error: 1554 SQLSTATE: HY000 (ER_WARN_DEPRECATED_SYNTAX_WITH_VER)

Message: The syntax '%s' is deprecated and will be removed in MySQL %s. Please use %s instead

- Error: 1555 SQLSTATE: HY000 (ER_CANT_WRITE_LOCK_LOG_TABLE)

Message: You can't write-lock a log table. Only read access is possible

- Error: 1556 SQLSTATE: HY000 (ER_CANT_LOCK_LOG_TABLE)

Message: You can't use locks with log tables.

- Error: 1557 SQLSTATE: 23000 (ER_FOREIGN_DUPLICATE_KEY_OLD_UNUSED)

Message: Upholding foreign key constraints for table '%s', entry '%s', key %d would lead to a duplicate entry

- Error: 1558 SQLSTATE: HY000 (ER_COL_COUNT_DOESNT_MATCH_PLEASE_UPDATE)

Message: Column count of mysql.%s is wrong. Expected %d, found %d. Created with MySQL %d, now running %d. Please use mysql_upgrade to fix this error.

- Error: 1559 SQLSTATE: HY000 (ER_TEMP_TABLE_PREVENTS_SWITCH_OUT_OF_RBR)

Message: Cannot switch out of the row-based binary log format when the session has open temporary tables

- Error: 1560 SQLSTATE: HY000 (ER_STORED_FUNCTION_PREVENTS_SWITCH_BINLOG_FORMAT)

Message: Cannot change the binary logging format inside a stored function or trigger

- Error: 1561 SQLSTATE: HY000 (ER_NDB_CANT_SWITCH_BINLOG_FORMAT)

Message: The NDB cluster engine does not support changing the binlog format on the fly yet

- Error: 1562 SQLSTATE: HY000 (ER_PARTITION_NO_TEMPORARY)

Message: Cannot create temporary table with partitions

- Error: 1563 SQLSTATE: HY000 (ER_PARTITION_CONST_DOMAIN_ERROR)

Message: Partition constant is out of partition function domain

- Error: 1564 SQLSTATE: HY000 (ER_PARTITION_FUNCTION_IS_NOT_ALLOWED)

Message: This partition function is not allowed

- Error: 1565 SQLSTATE: HY000 (ER_DDL_LOG_ERROR)

Message: Error in DDL log

- Error: 1566 SQLSTATE: HY000 (ER_NULL_IN_VALUES_LESS_THAN)

Message: Not allowed to use NULL value in VALUES LESS THAN

- Error: 1567 SQLSTATE: HY000 (ER_WRONG_PARTITION_NAME)

Message: Incorrect partition name

- Error: 1568 SQLSTATE: 25001 (ER_CANT_CHANGE_TX_CHARACTERISTICS)

Message: Transaction characteristics can't be changed while a transaction is in progress

- Error: 1569 SQLSTATE: HY000 (ER_DUP_ENTRY_AUTOINCREMENT_CASE)

Message: ALTER TABLE causes auto_increment resequencing, resulting in duplicate entry '%s' for key '%s'

- Error: 1570 SQLSTATE: HY000 (ER_EVENT_MODIFY_QUEUE_ERROR)

Message: Internal scheduler error %d

- Error: 1571 SQLSTATE: HY000 (ER_EVENT_SET_VAR_ERROR)

Message: Error during starting/stopping of the scheduler. Error code %u

- Error: 1572 SQLSTATE: HY000 (ER_PARTITION_MERGE_ERROR)

Message: Engine cannot be used in partitioned tables

- Error: 1573 SQLSTATE: HY000 (ER_CANT_ACTIVATE_LOG)

Message: Cannot activate '%s' log

- Error: 1574 SQLSTATE: HY000 (ER_RBR_NOT_AVAILABLE)

Message: The server was not built with row-based replication

- Error: 1575 SQLSTATE: HY000 (ER_BASE64_DECODE_ERROR)

Message: Decoding of base64 string failed

- Error: 1576 SQLSTATE: HY000 (ER_EVENT_RECURSION_FORBIDDEN)

Message: Recursion of EVENT DDL statements is forbidden when body is present

- Error: 1577 SQLSTATE: HY000 (ER_EVENTS_DB_ERROR)

Message: Cannot proceed because system tables used by Event Scheduler were found damaged at server start

To address this issue, try running mysql_upgrade.

- Error: 1578 SQLSTATE: HY000 (ER_ONLY_INTEGERS_ALLOWED)

Message: Only integers allowed as number here

- Error: 1579 SQLSTATE: HY000 (ER_UNSUPORTED_LOG_ENGINE)

Message: This storage engine cannot be used for log tables"

- Error: 1580 SQLSTATE: HY000 (ER_BAD_LOG_STATEMENT)

Message: You cannot '%s' a log table if logging is enabled

- Error: 1581 SQLSTATE: HY000 (ER_CANT_RENAME_LOG_TABLE)

Message: Cannot rename '%s'. When logging enabled, rename to/from log table must rename two tables: the log table to an archive table and another table back to '%s'

- Error: 1582 SQLSTATE: 42000 (ER_WRONG_PARAMCOUNT_TO_NATIVE_FCT)

Message: Incorrect parameter count in the call to native function '%s'

- Error: 1583 SQLSTATE: 42000 (ER_WRONG_PARAMETERS_TO_NATIVE_FCT)

Message: Incorrect parameters in the call to native function '%s'

- Error: 1584 SQLSTATE: 42000 (ER_WRONG_PARAMETERS_TO_STORED_FCT)

Message: Incorrect parameters in the call to stored function %s

- Error: 1585 SQLSTATE: HY000 (ER_NATIVE_FCT_NAME_COLLISION)

Message: This function '%s' has the same name as a native function

- Error: 1586 SQLSTATE: 23000 (ER_DUP_ENTRY_WITH_KEY_NAME)

Message: Duplicate entry '%s' for key '%s'

The format string for this error is also used with ER_DUP_ENTRY.

- Error: 1587 SQLSTATE: HY000 (ER_BINLOG_PURGE_EMFILE)

Message: Too many files opened, please execute the command again

- Error: 1588 SQLSTATE: HY000 (ER_EVENT_CANNOT_CREATE_IN_THE_PAST)

Message: Event execution time is in the past and ON COMPLETION NOT PRESERVE is set. The event was dropped immediately after creation.

- Error: 1589 SQLSTATE: HY000 (ER_EVENT_CANNOT_ALTER_IN_THE_PAST)

Message: Event execution time is in the past and ON COMPLETION NOT PRESERVE is set. The event was not changed. Specify a time in the future.

- Error: 1590 SQLSTATE: HY000 (ER_SLAVE_INCIDENT)

Message: The incident %s occured on the master. Message: %s

- Error: 1591 SQLSTATE: HY000 (ER_NO_PARTITION_FOR_GIVEN_VALUE_SILENT)

Message: Table has no partition for some existing values

- Error: 1592 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_STATEMENT)

Message: Unsafe statement written to the binary log using statement format since BINLOG_FORMAT = STATEMENT. %s

- Error: 1593 SQLSTATE: HY000 (ER_SLAVE_FATAL_ERROR)

Message: Fatal - Error: %s

- Error: 1594 SQLSTATE: HY000 (ER_SLAVE_RELAY_LOG_READ_FAILURE)

Message: Relay log read failure: %s

- Error: 1595 SQLSTATE: HY000 (ER_SLAVE_RELAY_LOG_WRITE_FAILURE)

Message: Relay log write failure: %s

- Error: 1596 SQLSTATE: HY000 (ER_SLAVE_CREATE_EVENT_FAILURE)

Message: Failed to create %s

- Error: 1597 SQLSTATE: HY000 (ER_SLAVE_MASTER_COM_FAILURE)

Message: Master command %s failed: %s

- Error: 1598 SQLSTATE: HY000 (ER_BINLOG_LOGGING_IMPOSSIBLE)

Message: Binary logging not possible. Message: %s

- Error: 1599 SQLSTATE: HY000 (ER_VIEW_NO_CREATION_CTX)

Message: View `%s`.`%s` has no creation context

- Error: 1600 SQLSTATE: HY000 (ER_VIEW_INVALID_CREATION_CTX)

Message: Creation context of view `%s`.`%s' is invalid

- Error: 1601 SQLSTATE: HY000 (ER_SR_INVALID_CREATION_CTX)

Message: Creation context of stored routine `%s`.`%s` is invalid

- Error: 1602 SQLSTATE: HY000 (ER_TRG_CORRUPTED_FILE)

Message: Corrupted TRG file for table `%s`.`%s`

- Error: 1603 SQLSTATE: HY000 (ER_TRG_NO_CREATION_CTX)

Message: Triggers for table `%s`.`%s` have no creation context

- Error: 1604 SQLSTATE: HY000 (ER_TRG_INVALID_CREATION_CTX)

Message: Trigger creation context of table `%s`.`%s` is invalid

- Error: 1605 SQLSTATE: HY000 (ER_EVENT_INVALID_CREATION_CTX)

Message: Creation context of event `%s`.`%s` is invalid

- Error: 1606 SQLSTATE: HY000 (ER_TRG_CANT_OPEN_TABLE)

Message: Cannot open table for trigger `%s`.`%s`

- Error: 1607 SQLSTATE: HY000 (ER_CANT_CREATE_SROUTINE)

Message: Cannot create stored routine `%s`. Check warnings

- Error: 1608 SQLSTATE: HY000 (ER_NEVER_USED)

Message: Ambiguous slave modes combination. %s

- Error: 1609 SQLSTATE: HY000 (ER_NO_FORMAT_DESCRIPTION_EVENT_BEFORE_BINLOG_STATEMENT)

Message: The BINLOG statement of type `%s` was not preceded by a format description BINLOG statement.

- Error: 1610 SQLSTATE: HY000 (ER_SLAVE_CORRUPT_EVENT)

Message: Corrupted replication event was detected

- Error: 1611 SQLSTATE: HY000 (ER_LOAD_DATA_INVALID_COLUMN)

Message: Invalid column reference (%s) in LOAD DATA

ER_LOAD_DATA_INVALID_COLUMN was removed after 5.7.7.

- Error: 1611 SQLSTATE: HY000 (ER_LOAD_DATA_INVALID_COLUMN_UNUSED)

Message: Invalid column reference (%s) in LOAD DATA

ER_LOAD_DATA_INVALID_COLUMN_UNUSED was added in 5.7.8.

- Error: 1612 SQLSTATE: HY000 (ER_LOG_PURGE_NO_FILE)

Message: Being purged log %s was not found

- Error: 1613 SQLSTATE: XA106 (ER_XA_RBTIMEOUT)

Message: XA_RBTIMEOUT: Transaction branch was rolled back: took too long

- Error: 1614 SQLSTATE: XA102 (ER_XA_RBDEADLOCK)

Message: XA_RBDEADLOCK: Transaction branch was rolled back: deadlock was detected

- Error: 1615 SQLSTATE: HY000 (ER_NEED_REPREPARE)

Message: Prepared statement needs to be re-prepared

- Error: 1616 SQLSTATE: HY000 (ER_DELAYED_NOT_SUPPORTED)

Message: DELAYED option not supported for table '%s'

- Error: 1617 SQLSTATE: HY000 (WARN_NO_MASTER_INFO)

Message: The master info structure does not exist

- Error: 1618 SQLSTATE: HY000 (WARN_OPTION_IGNORED)

Message: <%s> option ignored

- Error: 1619 SQLSTATE: HY000 (WARN_PLUGIN_DELETE_BUILTIN)

Message: Built-in plugins cannot be deleted

WARN_PLUGIN_DELETE_BUILTIN was removed after 5.7.4.

- Error: 1619 SQLSTATE: HY000 (ER_PLUGIN_DELETE_BUILTIN)

Message: Built-in plugins cannot be deleted

ER_PLUGIN_DELETE_BUILTIN was added in 5.7.5.

- Error: 1620 SQLSTATE: HY000 (WARN_PLUGIN_BUSY)

Message: Plugin is busy and will be uninstalled on shutdown

- Error: 1621 SQLSTATE: HY000 (ER_VARIABLE_IS_READONLY)

Message: %s variable '%s' is read-only. Use SET %s to assign the value

- Error: 1622 SQLSTATE: HY000 (ER_WARN_ENGINE_TRANSACTION_ROLLBACK)

Message: Storage engine %s does not support rollback for this statement. Transaction rolled back and must be restarted

- Error: 1623 SQLSTATE: HY000 (ER_SLAVE_HEARTBEAT_FAILURE)

Message: Unexpected master's heartbeat data: %s

- Error: 1624 SQLSTATE: HY000 (ER_SLAVE_HEARTBEAT_VALUE_OUT_OF_RANGE)

Message: The requested value for the heartbeat period is either negative or exceeds the maximum allowed (%s seconds).

- Error: 1625 SQLSTATE: HY000 (ER_NDB_REPLICATION_SCHEMA_ERROR)

Message: Bad schema for mysql.ndb_replication table. Message: %s

- Error: 1626 SQLSTATE: HY000 (ER_CONFLICT_FN_PARSE_ERROR)

Message: Error in parsing conflict function. Message: %s

- Error: 1627 SQLSTATE: HY000 (ER_EXCEPTIONS_WRITE_ERROR)

Message: Write to exceptions table failed. Message: %s"

- Error: 1628 SQLSTATE: HY000 (ER_TOO_LONG_TABLE_COMMENT)

Message: Comment for table '%s' is too long (max = %lu)

- Error: 1629 SQLSTATE: HY000 (ER_TOO_LONG_FIELD_COMMENT)

Message: Comment for field '%s' is too long (max = %lu)

- Error: 1630 SQLSTATE: 42000 (ER_FUNC_INEXISTENT_NAME_COLLISION)

Message: FUNCTION %s does not exist. Check the 'Function Name Parsing and Resolution' section in the Reference Manual

- Error: 1631 SQLSTATE: HY000 (ER_DATABASE_NAME)

Message: Database

- Error: 1632 SQLSTATE: HY000 (ER_TABLE_NAME)

Message: Table

- Error: 1633 SQLSTATE: HY000 (ER_PARTITION_NAME)

Message: Partition

- Error: 1634 SQLSTATE: HY000 (ER_SUBPARTITION_NAME)

Message: Subpartition

- Error: 1635 SQLSTATE: HY000 (ER_TEMPORARY_NAME)

Message: Temporary

- Error: 1636 SQLSTATE: HY000 (ER_RENAMED_NAME)

Message: Renamed

- Error: 1637 SQLSTATE: HY000 (ER_TOO_MANY_CONCURRENT_TRXS)

Message: Too many active concurrent transactions

- Error: 1638 SQLSTATE: HY000 (WARN_NON_ASCII_SEPARATOR_NOT_IMPLEMENTED)

Message: Non-ASCII separator arguments are not fully supported

- Error: 1639 SQLSTATE: HY000 (ER_DEBUG_SYNC_TIMEOUT)

Message: debug sync point wait timed out

- Error: 1640 SQLSTATE: HY000 (ER_DEBUG_SYNC_HIT_LIMIT)

Message: debug sync point hit limit reached

- Error: 1641 SQLSTATE: 42000 (ER_DUP_SIGNAL_SET)

Message: Duplicate condition information item '%s'

- Error: 1642 SQLSTATE: 01000 (ER_SIGNAL_WARN)

Message: Unhandled user-defined warning condition

- Error: 1643 SQLSTATE: 02000 (ER_SIGNAL_NOT_FOUND)

Message: Unhandled user-defined not found condition

- Error: 1644 SQLSTATE: HY000 (ER_SIGNAL_EXCEPTION)

Message: Unhandled user-defined exception condition

- Error: 1645 SQLSTATE: 0K000 (ER_RESIGNAL_WITHOUT_ACTIVE_HANDLER)

Message: RESIGNAL when handler not active

- Error: 1646 SQLSTATE: HY000 (ER_SIGNAL_BAD_CONDITION_TYPE)

Message: SIGNAL/RESIGNAL can only use a CONDITION defined with SQLSTATE

- Error: 1647 SQLSTATE: HY000 (WARN_COND_ITEM_TRUNCATED)

Message: Data truncated for condition item '%s'

- Error: 1648 SQLSTATE: HY000 (ER_COND_ITEM_TOO_LONG)

Message: Data too long for condition item '%s'

- Error: 1649 SQLSTATE: HY000 (ER_UNKNOWN_LOCALE)

Message: Unknown locale: '%s'

- Error: 1650 SQLSTATE: HY000 (ER_SLAVE_IGNORE_SERVER_IDS)

Message: The requested server id %d clashes with the slave startup option --replicate-same-server-id

- Error: 1651 SQLSTATE: HY000 (ER_QUERY_CACHE_DISABLED)

Message: Query cache is disabled; restart the server with query_cache_type=1 to enable it

- Error: 1652 SQLSTATE: HY000 (ER_SAME_NAME_PARTITION_FIELD)

Message: Duplicate partition field name '%s'

- Error: 1653 SQLSTATE: HY000 (ER_PARTITION_COLUMN_LIST_ERROR)

Message: Inconsistency in usage of column lists for partitioning

- Error: 1654 SQLSTATE: HY000 (ER_WRONG_TYPE_COLUMN_VALUE_ERROR)

Message: Partition column values of incorrect type

- Error: 1655 SQLSTATE: HY000 (ER_TOO_MANY_PARTITION_FUNC_FIELDS_ERROR)

Message: Too many fields in '%s'

- Error: 1656 SQLSTATE: HY000 (ER_MAXVALUE_IN_VALUES_IN)

Message: Cannot use MAXVALUE as value in VALUES IN

- Error: 1657 SQLSTATE: HY000 (ER_TOO_MANY_VALUES_ERROR)

Message: Cannot have more than one value for this type of %s partitioning

- Error: 1658 SQLSTATE: HY000 (ER_ROW_SINGLE_PARTITION_FIELD_ERROR)

Message: Row expressions in VALUES IN only allowed for multi-field column partitioning

- Error: 1659 SQLSTATE: HY000 (ER_FIELD_TYPE_NOT_ALLOWED_AS_PARTITION_FIELD)

Message: Field '%s' is of a not allowed type for this type of partitioning

- Error: 1660 SQLSTATE: HY000 (ER_PARTITION_FIELDS_TOO_LONG)

Message: The total length of the partitioning fields is too large

- Error: 1661 SQLSTATE: HY000 (ER_BINLOG_ROW_ENGINE_AND_STMT_ENGINE)

Message: Cannot execute statement: impossible to write to binary log since both row-incapable engines and statement-incapable engines are involved.

- Error: 1662 SQLSTATE: HY000 (ER_BINLOG_ROW_MODE_AND_STMT_ENGINE)

Message: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = ROW and at least one table uses a storage engine limited to statement-based logging.

- Error: 1663 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_AND_STMT_ENGINE)

Message: Cannot execute statement: impossible to write to binary log since statement is unsafe, storage engine is limited to statement-based logging, and BINLOG_FORMAT = MIXED. %s

- Error: 1664 SQLSTATE: HY000 (ER_BINLOG_ROW_INJECTION_AND_STMT_ENGINE)

Message: Cannot execute statement: impossible to write to binary log since statement is in row format and at least one table uses a storage engine limited to statement-based logging.

- Error: 1665 SQLSTATE: HY000 (ER_BINLOG_STMT_MODE_AND_ROW_ENGINE)

Message: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging.%s

- Error: 1666 SQLSTATE: HY000 (ER_BINLOG_ROW_INJECTION_AND_STMT_MODE)

Message: Cannot execute statement: impossible to write to binary log since statement is in row format and BINLOG_FORMAT = STATEMENT.

- Error: 1667 SQLSTATE: HY000 (ER_BINLOG_MULTIPLE_ENGINES_AND_SELF_LOGGING_ENGINE)

Message: Cannot execute statement: impossible to write to binary log since more than one engine is involved and at least one engine is self-logging.

- Error: 1668 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_LIMIT)

Message: The statement is unsafe because it uses a LIMIT clause. This is unsafe because the set of rows included cannot be predicted.

- Error: 1669 SQLSTATE: HY000 (ER_UNUSED4)

Message: The statement is unsafe because it uses INSERT DELAYED. This is unsafe because the times when rows are inserted cannot be predicted.

- Error: 1670 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_SYSTEM_TABLE)

Message: The statement is unsafe because it uses the general log, slow query log, or performance_schema table(s). This is unsafe because system tables may differ on slaves.

- Error: 1671 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_AUTOINC_COLUMNS)

Message: Statement is unsafe because it invokes a trigger or a stored function that inserts into an AUTO_INCREMENT column. Inserted values cannot be logged correctly.

- Error: 1672 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_UDF)

Message: Statement is unsafe because it uses a UDF which may not return the same value on the slave.

- Error: 1673 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_SYSTEM_VARIABLE)

Message: Statement is unsafe because it uses a system variable that may have a different value on the slave.

- Error: 1674 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_SYSTEM_FUNCTION)

Message: Statement is unsafe because it uses a system function that may return a different value on the slave.

- Error: 1675 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_NONTRANS_AFTER_TRANS)

Message: Statement is unsafe because it accesses a non-transactional table after accessing a transactional table within the same transaction.

- Error: 1676 SQLSTATE: HY000 (ER_MESSAGE_AND_STATEMENT)

Message: %s Statement: %s

- Error: 1677 SQLSTATE: HY000 (ER_SLAVE_CONVERSION_FAILED)

Message: Column %d of table '%s.%s' cannot be converted from type '%s' to type '%s'

- Error: 1678 SQLSTATE: HY000 (ER_SLAVE_CANT_CREATE_CONVERSION)

Message: Can't create conversion table for table '%s.%s'

- Error: 1679 SQLSTATE: HY000 (ER_INSIDE_TRANSACTION_PREVENTS_SWITCH_BINLOG_FORMAT)

Message: Cannot modify @@session.binlog_format inside a transaction

- Error: 1680 SQLSTATE: HY000 (ER_PATH_LENGTH)

Message: The path specified for %s is too long.

- Error: 1681 SQLSTATE: HY000 (ER_WARN_DEPRECATED_SYNTAX_NO_REPLACEMENT)

Message: '%s' is deprecated and will be removed in a future release.

- Error: 1682 SQLSTATE: HY000 (ER_WRONG_NATIVE_TABLE_STRUCTURE)

Message: Native table '%s'.'%s' has the wrong structure

- Error: 1683 SQLSTATE: HY000 (ER_WRONG_PERFSCHEMA_USAGE)

Message: Invalid performance_schema usage.

- Error: 1684 SQLSTATE: HY000 (ER_WARN_I_S_SKIPPED_TABLE)

Message: Table '%s'.'%s' was skipped since its definition is being modified by concurrent DDL statement

- Error: 1685 SQLSTATE: HY000 (ER_INSIDE_TRANSACTION_PREVENTS_SWITCH_BINLOG_DIRECT)

Message: Cannot modify @@session.binlog_direct_non_transactional_updates inside a transaction

- Error: 1686 SQLSTATE: HY000 (ER_STORED_FUNCTION_PREVENTS_SWITCH_BINLOG_DIRECT)

Message: Cannot change the binlog direct flag inside a stored function or trigger

- Error: 1687 SQLSTATE: 42000 (ER_SPATIAL_MUST_HAVE_GEOM_COL)

Message: A SPATIAL index may only contain a geometrical type column

- Error: 1688 SQLSTATE: HY000 (ER_TOO_LONG_INDEX_COMMENT)

Message: Comment for index '%s' is too long (max = %lu)

- Error: 1689 SQLSTATE: HY000 (ER_LOCK_ABORTED)

Message: Wait on a lock was aborted due to a pending exclusive lock

- Error: 1690 SQLSTATE: 22003 (ER_DATA_OUT_OF_RANGE)

Message: %s value is out of range in '%s'

- Error: 1691 SQLSTATE: HY000 (ER_WRONG_SPVAR_TYPE_IN_LIMIT)

Message: A variable of a non-integer based type in LIMIT clause

- Error: 1692 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_MULTIPLE_ENGINES_AND_SELF_LOGGING_ENGINE)

Message: Mixing self-logging and non-self-logging engines in a statement is unsafe.

- Error: 1693 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_MIXED_STATEMENT)

Message: Statement accesses nontransactional table as well as transactional or temporary table, and writes to any of them.

- Error: 1694 SQLSTATE: HY000 (ER_INSIDE_TRANSACTION_PREVENTS_SWITCH_SQL_LOG_BIN)

Message: Cannot modify @@session.sql_log_bin inside a transaction

- Error: 1695 SQLSTATE: HY000 (ER_STORED_FUNCTION_PREVENTS_SWITCH_SQL_LOG_BIN)

Message: Cannot change the sql_log_bin inside a stored function or trigger

- Error: 1696 SQLSTATE: HY000 (ER_FAILED_READ_FROM_PAR_FILE)

Message: Failed to read from the .par file

- Error: 1697 SQLSTATE: HY000 (ER_VALUES_IS_NOT_INT_TYPE_ERROR)

Message: VALUES value for partition '%s' must have type INT

- Error: 1698 SQLSTATE: 28000 (ER_ACCESS_DENIED_NO_PASSWORD_ERROR)

Message: Access denied for user '%s'@'%s'

- Error: 1699 SQLSTATE: HY000 (ER_SET_PASSWORD_AUTH_PLUGIN)

Message: SET PASSWORD has no significance for users authenticating via plugins

- Error: 1700 SQLSTATE: HY000 (ER_GRANT_PLUGIN_USER_EXISTS)

Message: GRANT with IDENTIFIED WITH is illegal because the user %-.*s already exists

- Error: 1701 SQLSTATE: 42000 (ER_TRUNCATE_ILLEGAL_FK)

Message: Cannot truncate a table referenced in a foreign key constraint (%s)

- Error: 1702 SQLSTATE: HY000 (ER_PLUGIN_IS_PERMANENT)

Message: Plugin '%s' is force_plus_permanent and can not be unloaded

- Error: 1703 SQLSTATE: HY000 (ER_SLAVE_HEARTBEAT_VALUE_OUT_OF_RANGE_MIN)

Message: The requested value for the heartbeat period is less than 1 millisecond. The value is reset to 0, meaning that heartbeating will effectively be disabled.

- Error: 1704 SQLSTATE: HY000 (ER_SLAVE_HEARTBEAT_VALUE_OUT_OF_RANGE_MAX)

Message: The requested value for the heartbeat period exceeds the value of `slave_net_timeout' seconds. A sensible value for the period should be less than the timeout.

- Error: 1705 SQLSTATE: HY000 (ER_STMT_CACHE_FULL)

Message: Multi-row statements required more than 'max_binlog_stmt_cache_size' bytes of storage; increase this mysqld variable and try again

- Error: 1706 SQLSTATE: HY000 (ER_MULTI_UPDATE_KEY_CONFLICT)

Message: Primary key/partition key update is not allowed since the table is updated both as '%s' and '%s'.

- Error: 1707 SQLSTATE: HY000 (ER_TABLE_NEEDS_REBUILD)

Message: Table rebuild required. Please do "ALTER TABLE `%s` FORCE" or dump/reload to fix it!

- Error: 1708 SQLSTATE: HY000 (WARN_OPTION_BELOW_LIMIT)

Message: The value of '%s' should be no less than the value of '%s'

- Error: 1709 SQLSTATE: HY000 (ER_INDEX_COLUMN_TOO_LONG)

Message: Index column size too large. The maximum column size is %lu bytes.

- Error: 1710 SQLSTATE: HY000 (ER_ERROR_IN_TRIGGER_BODY)

Message: Trigger '%s' has an error in its body: '%s'

- Error: 1711 SQLSTATE: HY000 (ER_ERROR_IN_UNKNOWN_TRIGGER_BODY)

Message: Unknown trigger has an error in its body: '%s'

- Error: 1712 SQLSTATE: HY000 (ER_INDEX_CORRUPT)

Message: Index %s is corrupted

- Error: 1713 SQLSTATE: HY000 (ER_UNDO_RECORD_TOO_BIG)

Message: Undo log record is too big.

- Error: 1714 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_INSERT_IGNORE_SELECT)

Message: INSERT IGNORE... SELECT is unsafe because the order in which rows are retrieved by the SELECT determines which (if any) rows are ignored. This order cannot be predicted and may differ on master and the slave.

- Error: 1715 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_INSERT_SELECT_UPDATE)

Message: INSERT... SELECT... ON DUPLICATE KEY UPDATE is unsafe because the order in which rows are retrieved by the SELECT determines which (if any) rows are updated. This order cannot be predicted and may differ on master and the slave.

- Error: 1716 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_REPLACE_SELECT)

Message: REPLACE... SELECT is unsafe because the order in which rows are retrieved by the SELECT determines which (if any) rows are replaced. This order cannot be predicted and may differ on master and the slave.

- Error: 1717 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_CREATE_IGNORE_SELECT)

Message: CREATE... IGNORE SELECT is unsafe because the order in which rows are retrieved by the SELECT determines which (if any) rows are ignored. This order cannot be predicted and may differ on master and the slave.

- Error: 1718 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_CREATE_REPLACE_SELECT)

Message: CREATE... REPLACE SELECT is unsafe because the order in which rows are retrieved by the SELECT determines which (if any) rows are replaced. This order cannot be predicted and may differ on master and the slave.

- Error: 1719 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_UPDATE_IGNORE)

Message: UPDATE IGNORE is unsafe because the order in which rows are updated determines which (if any) rows are ignored. This order cannot be predicted and may differ on master and the slave.

- Error: 1720 SQLSTATE: HY000 (ER_PLUGIN_NO_UNINSTALL)

Message: Plugin '%s' is marked as not dynamically uninstallable. You have to stop the server to uninstall it.

- Error: 1721 SQLSTATE: HY000 (ER_PLUGIN_NO_INSTALL)

Message: Plugin '%s' is marked as not dynamically installable. You have to stop the server to install it.

- Error: 1722 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_WRITE_AUTOINC_SELECT)

Message: Statements writing to a table with an auto-increment column after selecting from another table are unsafe because the order in which rows are retrieved determines what (if any) rows will be written. This order cannot be predicted and may differ on master and the slave.

- Error: 1723 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_CREATE_SELECT_AUTOINC)

Message: CREATE TABLE... SELECT... on a table with an auto-increment column is unsafe because the order in which rows are retrieved by the SELECT determines which (if any) rows are inserted. This order cannot be predicted and may differ on master and the slave.

- Error: 1724 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_INSERT_TWO_KEYS)

Message: INSERT... ON DUPLICATE KEY UPDATE on a table with more than one UNIQUE KEY is unsafe

- Error: 1725 SQLSTATE: HY000 (ER_TABLE_IN_FK_CHECK)

Message: Table is being used in foreign key check.

- Error: 1726 SQLSTATE: HY000 (ER_UNSUPPORTED_ENGINE)

Message: Storage engine '%s' does not support system tables. [%s.%s]

- Error: 1727 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_AUTOINC_NOT_FIRST)

Message: INSERT into autoincrement field which is not the first part in the composed primary key is unsafe.

- Error: 1728 SQLSTATE: HY000 (ER_CANNOT_LOAD_FROM_TABLE_V2)

Message: Cannot load from %s.%s. The table is probably corrupted

- Error: 1729 SQLSTATE: HY000 (ER_MASTER_DELAY_VALUE_OUT_OF_RANGE)

Message: The requested value %s for the master delay exceeds the maximum %u

- Error: 1730 SQLSTATE: HY000 (ER_ONLY_FD_AND_RBR_EVENTS_ALLOWED_IN_BINLOG_STATEMENT)

Message: Only Format_description_log_event and row events are allowed in BINLOG statements (but %s was provided)

- Error: 1731 SQLSTATE: HY000 (ER_PARTITION_EXCHANGE_DIFFERENT_OPTION)

Message: Non matching attribute '%s' between partition and table

- Error: 1732 SQLSTATE: HY000 (ER_PARTITION_EXCHANGE_PART_TABLE)

Message: Table to exchange with partition is partitioned: '%s'

- Error: 1733 SQLSTATE: HY000 (ER_PARTITION_EXCHANGE_TEMP_TABLE)

Message: Table to exchange with partition is temporary: '%s'

- Error: 1734 SQLSTATE: HY000 (ER_PARTITION_INSTEAD_OF_SUBPARTITION)

Message: Subpartitioned table, use subpartition instead of partition

- Error: 1735 SQLSTATE: HY000 (ER_UNKNOWN_PARTITION)

Message: Unknown partition '%s' in table '%s'

- Error: 1736 SQLSTATE: HY000 (ER_TABLES_DIFFERENT_METADATA)

Message: Tables have different definitions

- Error: 1737 SQLSTATE: HY000 (ER_ROW_DOES_NOT_MATCH_PARTITION)

Message: Found a row that does not match the partition

- Error: 1738 SQLSTATE: HY000 (ER_BINLOG_CACHE_SIZE_GREATER_THAN_MAX)

Message: Option binlog_cache_size (%lu) is greater than max_binlog_cache_size (%lu); setting binlog_cache_size equal to max_binlog_cache_size.

- Error: 1739 SQLSTATE: HY000 (ER_WARN_INDEX_NOT_APPLICABLE)

Message: Cannot use %s access on index '%s' due to type or collation conversion on field '%s'

- Error: 1740 SQLSTATE: HY000 (ER_PARTITION_EXCHANGE_FOREIGN_KEY)

Message: Table to exchange with partition has foreign key references: '%s'

- Error: 1741 SQLSTATE: HY000 (ER_NO_SUCH_KEY_VALUE)

Message: Key value '%s' was not found in table '%s.%s'

- Error: 1742 SQLSTATE: HY000 (ER_RPL_INFO_DATA_TOO_LONG)

Message: Data for column '%s' too long

- Error: 1743 SQLSTATE: HY000 (ER_NETWORK_READ_EVENT_CHECKSUM_FAILURE)

Message: Replication event checksum verification failed while reading from network.

- Error: 1744 SQLSTATE: HY000 (ER_BINLOG_READ_EVENT_CHECKSUM_FAILURE)

Message: Replication event checksum verification failed while reading from a log file.

- Error: 1745 SQLSTATE: HY000 (ER_BINLOG_STMT_CACHE_SIZE_GREATER_THAN_MAX)

Message: Option binlog_stmt_cache_size (%lu) is greater than max_binlog_stmt_cache_size (%lu); setting binlog_stmt_cache_size equal to max_binlog_stmt_cache_size.

- Error: 1746 SQLSTATE: HY000 (ER_CANT_UPDATE_TABLE_IN_CREATE_TABLE_SELECT)

Message: Can't update table '%s' while '%s' is being created.

- Error: 1747 SQLSTATE: HY000 (ER_PARTITION_CLAUSE_ON_NONPARTITIONED)

Message: PARTITION () clause on non partitioned table

- Error: 1748 SQLSTATE: HY000 (ER_ROW_DOES_NOT_MATCH_GIVEN_PARTITION_SET)

Message: Found a row not matching the given partition set

- Error: 1749 SQLSTATE: HY000 (ER_NO_SUCH_PARTITION__UNUSED)

Message: partition '%s' doesn't exist

- Error: 1750 SQLSTATE: HY000 (ER_CHANGE_RPL_INFO_REPOSITORY_FAILURE)

Message: Failure while changing the type of replication repository: %s.

- Error: 1751 SQLSTATE: HY000 (ER_WARNING_NOT_COMPLETE_ROLLBACK_WITH_CREATED_TEMP_TABLE)

Message: The creation of some temporary tables could not be rolled back.

- Error: 1752 SQLSTATE: HY000 (ER_WARNING_NOT_COMPLETE_ROLLBACK_WITH_DROPPED_TEMP_TABLE)

Message: Some temporary tables were dropped, but these operations could not be rolled back.

- Error: 1753 SQLSTATE: HY000 (ER_MTS_FEATURE_IS_NOT_SUPPORTED)

Message: %s is not supported in multi-threaded slave mode. %s

- Error: 1754 SQLSTATE: HY000 (ER_MTS_UPDATED_DBS_GREATER_MAX)

Message: The number of modified databases exceeds the maximum %d; the database names will not be included in the replication event metadata.

- Error: 1755 SQLSTATE: HY000 (ER_MTS_CANT_PARALLEL)

Message: Cannot execute the current event group in the parallel mode. Encountered event %s, relay-log name %s, position %s which prevents execution of this event group in parallel mode. Reason: %s.

- Error: 1756 SQLSTATE: HY000 (ER_MTS_INCONSISTENT_DATA)

Message: %s

- Error: 1757 SQLSTATE: HY000 (ER_FULLTEXT_NOT_SUPPORTED_WITH_PARTITIONING)

Message: FULLTEXT index is not supported for partitioned tables.

- Error: 1758 SQLSTATE: 35000 (ER_DA_INVALID_CONDITION_NUMBER)

Message: Invalid condition number

- Error: 1759 SQLSTATE: HY000 (ER_INSECURE_PLAIN_TEXT)

Message: Sending passwords in plain text without SSL/TLS is extremely insecure.

- Error: 1760 SQLSTATE: HY000 (ER_INSECURE_CHANGE_MASTER)

Message: Storing MySQL user name or password information in the master info repository is not secure and is therefore not recommended. Please consider using the USER and PASSWORD connection options for START SLAVE; see the 'START SLAVE Syntax' in the MySQL Manual for more information.

- Error: 1761 SQLSTATE: 23000 (ER_FOREIGN_DUPLICATE_KEY_WITH_CHILD_INFO)

Message: Foreign key constraint for table '%s', record '%s' would lead to a duplicate entry in table '%s', key '%s'

- Error: 1762 SQLSTATE: 23000 (ER_FOREIGN_DUPLICATE_KEY_WITHOUT_CHILD_INFO)

Message: Foreign key constraint for table '%s', record '%s' would lead to a duplicate entry in a child table

- Error: 1763 SQLSTATE: HY000 (ER_SQLTHREAD_WITH_SECURE_SLAVE)

Message: Setting authentication options is not possible when only the Slave SQL Thread is being started.

- Error: 1764 SQLSTATE: HY000 (ER_TABLE_HAS_NO_FT)

Message: The table does not have FULLTEXT index to support this query

- Error: 1765 SQLSTATE: HY000 (ER_VARIABLE_NOT_SETTABLE_IN_SF_OR_TRIGGER)

Message: The system variable %s cannot be set in stored functions or triggers.

- Error: 1766 SQLSTATE: HY000 (ER_VARIABLE_NOT_SETTABLE_IN_TRANSACTION)

Message: The system variable %s cannot be set when there is an ongoing transaction.

- Error: 1767 SQLSTATE: HY000 (ER_GTID_NEXT_IS_NOT_IN_GTID_NEXT_LIST)

Message: The system variable @@SESSION.GTID_NEXT has the value %s, which is not listed in @@SESSION.GTID_NEXT_LIST.

- Error: 1768 SQLSTATE: HY000 (ER_CANT_CHANGE_GTID_NEXT_IN_TRANSACTION_WHEN_GTID_NEXT_LIST_IS_NULL)

Message: The system variable @@SESSION.GTID_NEXT cannot change inside a transaction.

ER_CANT_CHANGE_GTID_NEXT_IN_TRANSACTION_WHEN_GTID_NEXT_LIST_IS_NULL was removed after 5.7.5.

- Error: 1768 SQLSTATE: HY000 (ER_CANT_CHANGE_GTID_NEXT_IN_TRANSACTION)

Message: The system variable @@SESSION.GTID_NEXT cannot change inside a transaction.

ER_CANT_CHANGE_GTID_NEXT_IN_TRANSACTION was added in 5.7.6.

- Error: 1769 SQLSTATE: HY000 (ER_SET_STATEMENT_CANNOT_INVOKE_FUNCTION)

Message: The statement 'SET %s' cannot invoke a stored function.

- Error: 1770 SQLSTATE: HY000 (ER_GTID_NEXT_CANT_BE_AUTOMATIC_IF_GTID_NEXT_LIST_IS_NON_NULL)

Message: The system variable @@SESSION.GTID_NEXT cannot be 'AUTOMATIC' when @@SESSION.GTID_NEXT_LIST is non-NULL.

- Error: 1771 SQLSTATE: HY000 (ER_SKIPPING_LOGGED_TRANSACTION)

Message: Skipping transaction %s because it has already been executed and logged.

- Error: 1772 SQLSTATE: HY000 (ER_MALFORMED_GTID_SET_SPECIFICATION)

Message: Malformed GTID set specification '%s'.

- Error: 1773 SQLSTATE: HY000 (ER_MALFORMED_GTID_SET_ENCODING)

Message: Malformed GTID set encoding.

- Error: 1774 SQLSTATE: HY000 (ER_MALFORMED_GTID_SPECIFICATION)

Message: Malformed GTID specification '%s'.

- Error: 1775 SQLSTATE: HY000 (ER_GNO_EXHAUSTED)

Message: Impossible to generate Global Transaction Identifier: the integer component reached the maximal value. Restart the server with a new server_uuid.

- Error: 1776 SQLSTATE: HY000 (ER_BAD_SLAVE_AUTO_POSITION)

Message: Parameters MASTER_LOG_FILE, MASTER_LOG_POS, RELAY_LOG_FILE and RELAY_LOG_POS cannot be set when MASTER_AUTO_POSITION is active.

- Error: 1777 SQLSTATE: HY000 (ER_AUTO_POSITION_REQUIRES_GTID_MODE_ON)

Message: CHANGE MASTER TO MASTER_AUTO_POSITION = 1 can only be executed when @@GLOBAL.GTID_MODE = ON.

ER_AUTO_POSITION_REQUIRES_GTID_MODE_ON was removed after 5.7.5.

- Error: 1777 SQLSTATE: HY000 (ER_AUTO_POSITION_REQUIRES_GTID_MODE_NOT_OFF)

Message: CHANGE MASTER TO MASTER_AUTO_POSITION = 1 cannot be executed because @@GLOBAL.GTID_MODE = OFF.

ER_AUTO_POSITION_REQUIRES_GTID_MODE_NOT_OFF was added in 5.7.6.

- Error: 1778 SQLSTATE: HY000 (ER_CANT_DO_IMPLICIT_COMMIT_IN_TRX_WHEN_GTID_NEXT_IS_SET)

Message: Cannot execute statements with implicit commit inside a transaction when @@SESSION.GTID_NEXT == 'UUID:NUMBER'.

- Error: 1779 SQLSTATE: HY000 (ER_GTID_MODE_2_OR_3_REQUIRES_ENFORCE_GTID_CONSISTENCY_ON)

Message: @@GLOBAL.GTID_MODE = ON or UPGRADE_STEP_2 requires @@GLOBAL.ENFORCE_GTID_CONSISTENCY = 1.

ER_GTID_MODE_2_OR_3_REQUIRES_ENFORCE_GTID_CONSISTENCY_ON was removed after 5.7.5.

- Error: 1779 SQLSTATE: HY000 (ER_GTID_MODE_ON_REQUIRES_ENFORCE_GTID_CONSISTENCY_ON)

Message: GTID_MODE = ON requires ENFORCE_GTID_CONSISTENCY = ON.

ER_GTID_MODE_ON_REQUIRES_ENFORCE_GTID_CONSISTENCY_ON was added in 5.7.6.

- Error: 1780 SQLSTATE: HY000 (ER_GTID_MODE_REQUIRES_BINLOG)

Message: @@GLOBAL.GTID_MODE = ON or ON_PERMISSIVE or OFF_PERMISSIVE requires --log-bin and --log-slave-updates.

- Error: 1781 SQLSTATE: HY000 (ER_CANT_SET_GTID_NEXT_TO_GTID_WHEN_GTID_MODE_IS_OFF)

Message: @@SESSION.GTID_NEXT cannot be set to UUID:NUMBER when @@GLOBAL.GTID_MODE = OFF.

- Error: 1782 SQLSTATE: HY000 (ER_CANT_SET_GTID_NEXT_TO_ANONYMOUS_WHEN_GTID_MODE_IS_ON)

Message: @@SESSION.GTID_NEXT cannot be set to ANONYMOUS when @@GLOBAL.GTID_MODE = ON.

- Error: 1783 SQLSTATE: HY000 (ER_CANT_SET_GTID_NEXT_LIST_TO_NON_NULL_WHEN_GTID_MODE_IS_OFF)

Message: @@SESSION.GTID_NEXT_LIST cannot be set to a non-NULL value when @@GLOBAL.GTID_MODE = OFF.

- Error: 1784 SQLSTATE: HY000 (ER_FOUND_GTID_EVENT_WHEN_GTID_MODE_IS_OFF)

Message: Found a Gtid_log_event or Previous_gtids_log_event when @@GLOBAL.GTID_MODE = OFF.

ER_FOUND_GTID_EVENT_WHEN_GTID_MODE_IS_OFF was removed after 5.7.5.

- Error: 1784 SQLSTATE: HY000 (ER_FOUND_GTID_EVENT_WHEN_GTID_MODE_IS_OFF__UNUSED)

Message: Found a Gtid_log_event when @@GLOBAL.GTID_MODE = OFF.

ER_FOUND_GTID_EVENT_WHEN_GTID_MODE_IS_OFF__UNUSED was added in 5.7.6.

- Error: 1785 SQLSTATE: HY000 (ER_GTID_UNSAFE_NON_TRANSACTIONAL_TABLE)

Message: Statement violates GTID consistency: Updates to non-transactional tables can only be done in either autocommitted statements or single-statement transactions, and never in the same statement as updates to transactional tables.

- Error: 1786 SQLSTATE: HY000 (ER_GTID_UNSAFE_CREATE_SELECT)

Message: Statement violates GTID consistency: CREATE TABLE ... SELECT.

- Error: 1787 SQLSTATE: HY000 (ER_GTID_UNSAFE_CREATE_DROP_TEMPORARY_TABLE_IN_TRANSACTION)

Message: Statement violates GTID consistency: CREATE TEMPORARY TABLE and DROP TEMPORARY TABLE can only be executed outside transactional context. These statements are also not allowed in a function or trigger because functions and triggers are also considered to be multi-statement transactions.

- Error: 1788 SQLSTATE: HY000 (ER_GTID_MODE_CAN_ONLY_CHANGE_ONE_STEP_AT_A_TIME)

Message: The value of @@GLOBAL.GTID_MODE can only be changed one step at a time: OFF <-> OFF_PERMISSIVE <-> ON_PERMISSIVE <-> ON. Also note that this value must be stepped up or down simultaneously on all servers. See the Manual for instructions.

- Error: 1789 SQLSTATE: HY000 (ER_MASTER_HAS_PURGED_REQUIRED_GTIDS)

Message: The slave is connecting using CHANGE MASTER TO MASTER_AUTO_POSITION = 1, but the master has purged binary logs containing GTIDs that the slave requires.

- Error: 1790 SQLSTATE: HY000 (ER_CANT_SET_GTID_NEXT_WHEN_OWNING_GTID)

Message: @@SESSION.GTID_NEXT cannot be changed by a client that owns a GTID. The client owns %s. Ownership is released on COMMIT or ROLLBACK.

- Error: 1791 SQLSTATE: HY000 (ER_UNKNOWN_EXPLAIN_FORMAT)

Message: Unknown EXPLAIN format name: '%s'

- Error: 1792 SQLSTATE: 25006 (ER_CANT_EXECUTE_IN_READ_ONLY_TRANSACTION)

Message: Cannot execute statement in a READ ONLY transaction.

- Error: 1793 SQLSTATE: HY000 (ER_TOO_LONG_TABLE_PARTITION_COMMENT)

Message: Comment for table partition '%s' is too long (max = %lu)

- Error: 1794 SQLSTATE: HY000 (ER_SLAVE_CONFIGURATION)

Message: Slave is not configured or failed to initialize properly. You must at least set --server-id to enable either a master or a slave. Additional error messages can be found in the MySQL error log.

- Error: 1795 SQLSTATE: HY000 (ER_INNODB_FT_LIMIT)

Message: InnoDB presently supports one FULLTEXT index creation at a time

- Error: 1796 SQLSTATE: HY000 (ER_INNODB_NO_FT_TEMP_TABLE)

Message: Cannot create FULLTEXT index on temporary InnoDB table

- Error: 1797 SQLSTATE: HY000 (ER_INNODB_FT_WRONG_DOCID_COLUMN)

Message: Column '%s' is of wrong type for an InnoDB FULLTEXT index

- Error: 1798 SQLSTATE: HY000 (ER_INNODB_FT_WRONG_DOCID_INDEX)

Message: Index '%s' is of wrong type for an InnoDB FULLTEXT index

- Error: 1799 SQLSTATE: HY000 (ER_INNODB_ONLINE_LOG_TOO_BIG)

Message: Creating index '%s' required more than 'innodb_online_alter_log_max_size' bytes of modification log. Please try again.

- Error: 1800 SQLSTATE: HY000 (ER_UNKNOWN_ALTER_ALGORITHM)

Message: Unknown ALGORITHM '%s'

- Error: 1801 SQLSTATE: HY000 (ER_UNKNOWN_ALTER_LOCK)

Message: Unknown LOCK type '%s'

- Error: 1802 SQLSTATE: HY000 (ER_MTS_CHANGE_MASTER_CANT_RUN_WITH_GAPS)

Message: CHANGE MASTER cannot be executed when the slave was stopped with an error or killed in MTS mode. Consider using RESET SLAVE or START SLAVE UNTIL.

- Error: 1803 SQLSTATE: HY000 (ER_MTS_RECOVERY_FAILURE)

Message: Cannot recover after SLAVE errored out in parallel execution mode. Additional error messages can be found in the MySQL error log.

- Error: 1804 SQLSTATE: HY000 (ER_MTS_RESET_WORKERS)

Message: Cannot clean up worker info tables. Additional error messages can be found in the MySQL error log.

- Error: 1805 SQLSTATE: HY000 (ER_COL_COUNT_DOESNT_MATCH_CORRUPTED_V2)

Message: Column count of %s.%s is wrong. Expected %d, found %d. The table is probably corrupted

- Error: 1806 SQLSTATE: HY000 (ER_SLAVE_SILENT_RETRY_TRANSACTION)

Message: Slave must silently retry current transaction

- Error: 1807 SQLSTATE: HY000 (ER_DISCARD_FK_CHECKS_RUNNING)

Message: There is a foreign key check running on table '%s'. Cannot discard the table.

- Error: 1808 SQLSTATE: HY000 (ER_TABLE_SCHEMA_MISMATCH)

Message: Schema mismatch (%s)

- Error: 1809 SQLSTATE: HY000 (ER_TABLE_IN_SYSTEM_TABLESPACE)

Message: Table '%s' in system tablespace

- Error: 1810 SQLSTATE: HY000 (ER_IO_READ_ERROR)

Message: IO Read - Error: (%lu, %s) %s

- Error: 1811 SQLSTATE: HY000 (ER_IO_WRITE_ERROR)

Message: IO Write - Error: (%lu, %s) %s

- Error: 1812 SQLSTATE: HY000 (ER_TABLESPACE_MISSING)

Message: Tablespace is missing for table %s.

- Error: 1813 SQLSTATE: HY000 (ER_TABLESPACE_EXISTS)

Message: Tablespace '%s' exists.

- Error: 1814 SQLSTATE: HY000 (ER_TABLESPACE_DISCARDED)

Message: Tablespace has been discarded for table '%s'

- Error: 1815 SQLSTATE: HY000 (ER_INTERNAL_ERROR)

Message: Internal - Error: %s

- Error: 1816 SQLSTATE: HY000 (ER_INNODB_IMPORT_ERROR)

Message: ALTER TABLE %s IMPORT TABLESPACE failed with error %lu : '%s'

- Error: 1817 SQLSTATE: HY000 (ER_INNODB_INDEX_CORRUPT)

Message: Index corrupt: %s

- Error: 1818 SQLSTATE: HY000 (ER_INVALID_YEAR_COLUMN_LENGTH)

Message: Supports only YEAR or YEAR(4) column.

- Error: 1819 SQLSTATE: HY000 (ER_NOT_VALID_PASSWORD)

Message: Your password does not satisfy the current policy requirements

- Error: 1820 SQLSTATE: HY000 (ER_MUST_CHANGE_PASSWORD)

Message: You must reset your password using ALTER USER statement before executing this statement.

- Error: 1821 SQLSTATE: HY000 (ER_FK_NO_INDEX_CHILD)

Message: Failed to add the foreign key constaint. Missing index for constraint '%s' in the foreign table '%s'

- Error: 1822 SQLSTATE: HY000 (ER_FK_NO_INDEX_PARENT)

Message: Failed to add the foreign key constaint. Missing index for constraint '%s' in the referenced table '%s'

- Error: 1823 SQLSTATE: HY000 (ER_FK_FAIL_ADD_SYSTEM)

Message: Failed to add the foreign key constraint '%s' to system tables

- Error: 1824 SQLSTATE: HY000 (ER_FK_CANNOT_OPEN_PARENT)

Message: Failed to open the referenced table '%s'

- Error: 1825 SQLSTATE: HY000 (ER_FK_INCORRECT_OPTION)

Message: Failed to add the foreign key constraint on table '%s'. Incorrect options in FOREIGN KEY constraint '%s'

- Error: 1826 SQLSTATE: HY000 (ER_FK_DUP_NAME)

Message: Duplicate foreign key constraint name '%s'

- Error: 1827 SQLSTATE: HY000 (ER_PASSWORD_FORMAT)

Message: The password hash doesn't have the expected format. Check if the correct password algorithm is being used with the PASSWORD() function.

- Error: 1828 SQLSTATE: HY000 (ER_FK_COLUMN_CANNOT_DROP)

Message: Cannot drop column '%s': needed in a foreign key constraint '%s'

- Error: 1829 SQLSTATE: HY000 (ER_FK_COLUMN_CANNOT_DROP_CHILD)

Message: Cannot drop column '%s': needed in a foreign key constraint '%s' of table '%s'

- Error: 1830 SQLSTATE: HY000 (ER_FK_COLUMN_NOT_NULL)

Message: Column '%s' cannot be NOT NULL: needed in a foreign key constraint '%s' SET NULL

- Error: 1831 SQLSTATE: HY000 (ER_DUP_INDEX)

Message: Duplicate index '%s' defined on the table '%s.%s'. This is deprecated and will be disallowed in a future release.

- Error: 1832 SQLSTATE: HY000 (ER_FK_COLUMN_CANNOT_CHANGE)

Message: Cannot change column '%s': used in a foreign key constraint '%s'

- Error: 1833 SQLSTATE: HY000 (ER_FK_COLUMN_CANNOT_CHANGE_CHILD)

Message: Cannot change column '%s': used in a foreign key constraint '%s' of table '%s'

- Error: 1834 SQLSTATE: HY000 (ER_FK_CANNOT_DELETE_PARENT)

Message: Cannot delete rows from table which is parent in a foreign key constraint '%s' of table '%s'

ER_FK_CANNOT_DELETE_PARENT was removed after 5.7.3.

- Error: 1834 SQLSTATE: HY000 (ER_UNUSED5)

Message: Cannot delete rows from table which is parent in a foreign key constraint '%s' of table '%s'

ER_UNUSED5 was added in 5.7.4.

- Error: 1835 SQLSTATE: HY000 (ER_MALFORMED_PACKET)

Message: Malformed communication packet.

- Error: 1836 SQLSTATE: HY000 (ER_READ_ONLY_MODE)

Message: Running in read-only mode

- Error: 1837 SQLSTATE: HY000 (ER_GTID_NEXT_TYPE_UNDEFINED_GROUP)

Message: When @@SESSION.GTID_NEXT is set to a GTID, you must explicitly set it to a different value after a COMMIT or ROLLBACK. Please check GTID_NEXT variable manual page for detailed explanation. Current @@SESSION.GTID_NEXT is '%s'.

- Error: 1838 SQLSTATE: HY000 (ER_VARIABLE_NOT_SETTABLE_IN_SP)

Message: The system variable %s cannot be set in stored procedures.

- Error: 1839 SQLSTATE: HY000 (ER_CANT_SET_GTID_PURGED_WHEN_GTID_MODE_IS_OFF)

Message: @@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_MODE = ON.

- Error: 1840 SQLSTATE: HY000 (ER_CANT_SET_GTID_PURGED_WHEN_GTID_EXECUTED_IS_NOT_EMPTY)

Message: @@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_EXECUTED is empty.

- Error: 1841 SQLSTATE: HY000 (ER_CANT_SET_GTID_PURGED_WHEN_OWNED_GTIDS_IS_NOT_EMPTY)

Message: @@GLOBAL.GTID_PURGED can only be set when there are no ongoing transactions (not even in other clients).

- Error: 1842 SQLSTATE: HY000 (ER_GTID_PURGED_WAS_CHANGED)

Message: @@GLOBAL.GTID_PURGED was changed from '%s' to '%s'.

- Error: 1843 SQLSTATE: HY000 (ER_GTID_EXECUTED_WAS_CHANGED)

Message: @@GLOBAL.GTID_EXECUTED was changed from '%s' to '%s'.

- Error: 1844 SQLSTATE: HY000 (ER_BINLOG_STMT_MODE_AND_NO_REPL_TABLES)

Message: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT, and both replicated and non replicated tables are written to.

- Error: 1845 SQLSTATE: 0A000 (ER_ALTER_OPERATION_NOT_SUPPORTED)

Message: %s is not supported for this operation. Try %s.

ER_ALTER_OPERATION_NOT_SUPPORTED was added in 5.7.1.

- Error: 1846 SQLSTATE: 0A000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON)

Message: %s is not supported. Reason: %s. Try %s.

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON was added in 5.7.1.

- Error: 1847 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_COPY)

Message: COPY algorithm requires a lock

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_COPY was added in 5.7.1.

- Error: 1848 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_PARTITION)

Message: Partition specific operations do not yet support LOCK/ALGORITHM

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_PARTITION was added in 5.7.1.

- Error: 1849 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_FK_RENAME)

Message: Columns participating in a foreign key are renamed

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_FK_RENAME was added in 5.7.1.

- Error: 1850 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_COLUMN_TYPE)

Message: Cannot change column type INPLACE

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_COLUMN_TYPE was added in 5.7.1.

- Error: 1851 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_FK_CHECK)

Message: Adding foreign keys needs foreign_key_checks=OFF

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_FK_CHECK was added in 5.7.1.

- Error: 1852 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_IGNORE)

Message: Creating unique indexes with IGNORE requires COPY algorithm to remove duplicate rows

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_IGNORE was added in 5.7.1, removed after 5.7.3.

- Error: 1852 SQLSTATE: HY000 (ER_UNUSED6)

Message: Creating unique indexes with IGNORE requires COPY algorithm to remove duplicate rows

ER_UNUSED6 was added in 5.7.4.

- Error: 1853 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_NOPK)

Message: Dropping a primary key is not allowed without also adding a new primary key

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_NOPK was added in 5.7.1.

- Error: 1854 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_AUTOINC)

Message: Adding an auto-increment column requires a lock

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_AUTOINC was added in 5.7.1.

- Error: 1855 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_HIDDEN_FTS)

Message: Cannot replace hidden FTS_DOC_ID with a user-visible one

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_HIDDEN_FTS was added in 5.7.1.

- Error: 1856 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_CHANGE_FTS)

Message: Cannot drop or rename FTS_DOC_ID

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_CHANGE_FTS was added in 5.7.1.

- Error: 1857 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_FTS)

Message: Fulltext index creation requires a lock

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_FTS was added in 5.7.1.

- Error: 1858 SQLSTATE: HY000 (ER_SQL_SLAVE_SKIP_COUNTER_NOT_SETTABLE_IN_GTID_MODE)

Message: sql_slave_skip_counter can not be set when the server is running with @@GLOBAL.GTID_MODE = ON. Instead, for each transaction that you want to skip, generate an empty transaction with the same GTID as the transaction

ER_SQL_SLAVE_SKIP_COUNTER_NOT_SETTABLE_IN_GTID_MODE was added in 5.7.1.

- Error: 1859 SQLSTATE: 23000 (ER_DUP_UNKNOWN_IN_INDEX)

Message: Duplicate entry for key '%s'

ER_DUP_UNKNOWN_IN_INDEX was added in 5.7.1.

- Error: 1860 SQLSTATE: HY000 (ER_IDENT_CAUSES_TOO_LONG_PATH)

Message: Long database name and identifier for object resulted in path length exceeding %d characters. Path: '%s'.

ER_IDENT_CAUSES_TOO_LONG_PATH was added in 5.7.1.

- Error: 1861 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_NOT_NULL)

Message: cannot silently convert NULL values, as required in this SQL_MODE

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_NOT_NULL was added in 5.7.1.

- Error: 1862 SQLSTATE: HY000 (ER_MUST_CHANGE_PASSWORD_LOGIN)

Message: Your password has expired. To log in you must change it using a client that supports expired passwords.

ER_MUST_CHANGE_PASSWORD_LOGIN was added in 5.7.1.

- Error: 1863 SQLSTATE: HY000 (ER_ROW_IN_WRONG_PARTITION)

Message: Found a row in wrong partition %s

ER_ROW_IN_WRONG_PARTITION was added in 5.7.1.

- Error: 1864 SQLSTATE: HY000 (ER_MTS_EVENT_BIGGER_PENDING_JOBS_SIZE_MAX)

Message: Cannot schedule event %s, relay-log name %s, position %s to Worker thread because its size %lu exceeds %lu of slave_pending_jobs_size_max.

ER_MTS_EVENT_BIGGER_PENDING_JOBS_SIZE_MAX was added in 5.7.2.

- Error: 1865 SQLSTATE: HY000 (ER_INNODB_NO_FT_USES_PARSER)

Message: Cannot CREATE FULLTEXT INDEX WITH PARSER on InnoDB table

ER_INNODB_NO_FT_USES_PARSER was added in 5.7.2.

- Error: 1866 SQLSTATE: HY000 (ER_BINLOG_LOGICAL_CORRUPTION)

Message: The binary log file '%s' is logically corrupted: %s

ER_BINLOG_LOGICAL_CORRUPTION was added in 5.7.2.

- Error: 1867 SQLSTATE: HY000 (ER_WARN_PURGE_LOG_IN_USE)

Message: file %s was not purged because it was being read by %d thread(s), purged only %d out of %d files.

ER_WARN_PURGE_LOG_IN_USE was added in 5.7.2.

- Error: 1868 SQLSTATE: HY000 (ER_WARN_PURGE_LOG_IS_ACTIVE)

Message: file %s was not purged because it is the active log file.

ER_WARN_PURGE_LOG_IS_ACTIVE was added in 5.7.2.

- Error: 1869 SQLSTATE: HY000 (ER_AUTO_INCREMENT_CONFLICT)

Message: Auto-increment value in UPDATE conflicts with internally generated values

ER_AUTO_INCREMENT_CONFLICT was added in 5.7.2.

- Error: 1870 SQLSTATE: HY000 (WARN_ON_BLOCKHOLE_IN_RBR)

Message: Row events are not logged for %s statements that modify BLACKHOLE tables in row format. Table(s): '%s'

WARN_ON_BLOCKHOLE_IN_RBR was added in 5.7.2.

- Error: 1871 SQLSTATE: HY000 (ER_SLAVE_MI_INIT_REPOSITORY)

Message: Slave failed to initialize master info structure from the repository

ER_SLAVE_MI_INIT_REPOSITORY was added in 5.7.2.

- Error: 1872 SQLSTATE: HY000 (ER_SLAVE_RLI_INIT_REPOSITORY)

Message: Slave failed to initialize relay log info structure from the repository

ER_SLAVE_RLI_INIT_REPOSITORY was added in 5.7.2.

- Error: 1873 SQLSTATE: 28000 (ER_ACCESS_DENIED_CHANGE_USER_ERROR)

Message: Access denied trying to change to user '%s'@'%s' (using password: %s). Disconnecting.

ER_ACCESS_DENIED_CHANGE_USER_ERROR was added in 5.7.2.

- Error: 1874 SQLSTATE: HY000 (ER_INNODB_READ_ONLY)

Message: InnoDB is in read only mode.

ER_INNODB_READ_ONLY was added in 5.7.2.

- Error: 1875 SQLSTATE: HY000 (ER_STOP_SLAVE_SQL_THREAD_TIMEOUT)

Message: STOP SLAVE command execution is incomplete: Slave SQL thread got the stop signal, thread is busy, SQL thread will stop once the current task is complete.

ER_STOP_SLAVE_SQL_THREAD_TIMEOUT was added in 5.7.2.

- Error: 1876 SQLSTATE: HY000 (ER_STOP_SLAVE_IO_THREAD_TIMEOUT)

Message: STOP SLAVE command execution is incomplete: Slave IO thread got the stop signal, thread is busy, IO thread will stop once the current task is complete.

ER_STOP_SLAVE_IO_THREAD_TIMEOUT was added in 5.7.2.

- Error: 1877 SQLSTATE: HY000 (ER_TABLE_CORRUPT)

Message: Operation cannot be performed. The table '%s.%s' is missing, corrupt or contains bad data.

ER_TABLE_CORRUPT was added in 5.7.2.

- Error: 1878 SQLSTATE: HY000 (ER_TEMP_FILE_WRITE_FAILURE)

Message: Temporary file write failure.

ER_TEMP_FILE_WRITE_FAILURE was added in 5.7.3.

- Error: 1879 SQLSTATE: HY000 (ER_INNODB_FT_AUX_NOT_HEX_ID)

Message: Upgrade index name failed, please use create index(alter table) algorithm copy to rebuild index.

ER_INNODB_FT_AUX_NOT_HEX_ID was added in 5.7.4.

- Error: 1880 SQLSTATE: HY000 (ER_OLD_TEMPORALS_UPGRADED)

Message: TIME/TIMESTAMP/DATETIME columns of old format have been upgraded to the new format.

ER_OLD_TEMPORALS_UPGRADED was added in 5.7.4.

- Error: 1881 SQLSTATE: HY000 (ER_INNODB_FORCED_RECOVERY)

Message: Operation not allowed when innodb_forced_recovery > 0.

ER_INNODB_FORCED_RECOVERY was added in 5.7.4.

- Error: 1882 SQLSTATE: HY000 (ER_AES_INVALID_IV)

Message: The initialization vector supplied to %s is too short. Must be at least %d bytes long

ER_AES_INVALID_IV was added in 5.7.4.

- Error: 1883 SQLSTATE: HY000 (ER_PLUGIN_CANNOT_BE_UNINSTALLED)

Message: Plugin '%s' cannot be uninstalled now. %s

ER_PLUGIN_CANNOT_BE_UNINSTALLED was added in 5.7.5.

- Error: 1884 SQLSTATE: HY000 (ER_GTID_UNSAFE_BINLOG_SPLITTABLE_STATEMENT_AND_GTID_GROUP)

Message: Cannot execute statement because it needs to be written to the binary log as multiple statements, and this is not allowed when @@SESSION.GTID_NEXT == 'UUID:NUMBER'.

ER_GTID_UNSAFE_BINLOG_SPLITTABLE_STATEMENT_AND_GTID_GROUP was added in 5.7.5.

- Error: 1885 SQLSTATE: HY000 (ER_SLAVE_HAS_MORE_GTIDS_THAN_MASTER)

Message: Slave has more GTIDs than the master has, using the master's SERVER_UUID. This may indicate that the end of the binary log was truncated or that the last binary log file was lost, e.g., after a power or disk failure when sync_binlog != 1. The master may or may not have rolled back transactions that were already replicated to the slave. Suggest to replicate any transactions that master has rolled back from slave to master, and/or commit empty transactions on master to account for transactions that have been committed on master but are not included in GTID_EXECUTED.

ER_SLAVE_HAS_MORE_GTIDS_THAN_MASTER was added in 5.7.6.

- Error: 1886 SQLSTATE: HY000 (ER_MISSING_KEY)

Message: The table '%s.%s' does not have the necessary key(s) defined on it. Please check the table definition and create index(s) accordingly.

ER_MISSING_KEY was added in 5.7.22.

- Error: 1906 SQLSTATE: HY000 (ER_SLAVE_IO_THREAD_MUST_STOP)

Message: This operation cannot be performed with a running slave io thread; run STOP SLAVE IO_THREAD first.

ER_SLAVE_IO_THREAD_MUST_STOP was added in 5.7.4, removed after 5.7.5.

- Error: 3000 SQLSTATE: HY000 (ER_FILE_CORRUPT)

Message: File %s is corrupted

- Error: 3001 SQLSTATE: HY000 (ER_ERROR_ON_MASTER)

Message: Query partially completed on the master (error on master: %d) and was aborted. There is a chance that your master is inconsistent at this point. If you are sure that your master is ok, run this query manually on the slave and then restart the slave with SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1; START SLAVE;. Query:'%s'

- Error: 3002 SQLSTATE: HY000 (ER_INCONSISTENT_ERROR)

Message: Query caused different errors on master and slave. Error on master: message (format)='%s' error code=%d; Error on slave:actual message='%s', error code=%d. Default database:'%s'. Query:'%s'

- Error: 3003 SQLSTATE: HY000 (ER_STORAGE_ENGINE_NOT_LOADED)

Message: Storage engine for table '%s'.'%s' is not loaded.

- Error: 3004 SQLSTATE: 0Z002 (ER_GET_STACKED_DA_WITHOUT_ACTIVE_HANDLER)

Message: GET STACKED DIAGNOSTICS when handler not active

- Error: 3005 SQLSTATE: HY000 (ER_WARN_LEGACY_SYNTAX_CONVERTED)

Message: %s is no longer supported. The statement was converted to %s.

- Error: 3006 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_FULLTEXT_PLUGIN)

Message: Statement is unsafe because it uses a fulltext parser plugin which may not return the same value on the slave.

ER_BINLOG_UNSAFE_FULLTEXT_PLUGIN was added in 5.7.1.

- Error: 3007 SQLSTATE: HY000 (ER_CANNOT_DISCARD_TEMPORARY_TABLE)

Message: Cannot DISCARD/IMPORT tablespace associated with temporary table

ER_CANNOT_DISCARD_TEMPORARY_TABLE was added in 5.7.1.

- Error: 3008 SQLSTATE: HY000 (ER_FK_DEPTH_EXCEEDED)

Message: Foreign key cascade delete/update exceeds max depth of %d.

ER_FK_DEPTH_EXCEEDED was added in 5.7.2.

- Error: 3009 SQLSTATE: HY000 (ER_COL_COUNT_DOESNT_MATCH_PLEASE_UPDATE_V2)

Message: Column count of %s.%s is wrong. Expected %d, found %d. Created with MySQL %d, now running %d. Please use mysql_upgrade to fix this error.

ER_COL_COUNT_DOESNT_MATCH_PLEASE_UPDATE_V2 was added in 5.7.2.

- Error: 3010 SQLSTATE: HY000 (ER_WARN_TRIGGER_DOESNT_HAVE_CREATED)

Message: Trigger %s.%s.%s does not have CREATED attribute.

ER_WARN_TRIGGER_DOESNT_HAVE_CREATED was added in 5.7.2.

- Error: 3011 SQLSTATE: HY000 (ER_REFERENCED_TRG_DOES_NOT_EXIST)

Message: Referenced trigger '%s' for the given action time and event type does not exist.

ER_REFERENCED_TRG_DOES_NOT_EXIST was added in 5.7.2.

- Error: 3012 SQLSTATE: HY000 (ER_EXPLAIN_NOT_SUPPORTED)

Message: EXPLAIN FOR CONNECTION command is supported only for SELECT/UPDATE/INSERT/DELETE/REPLACE

ER_EXPLAIN_NOT_SUPPORTED was added in 5.7.2.

- Error: 3013 SQLSTATE: HY000 (ER_INVALID_FIELD_SIZE)

Message: Invalid size for column '%s'.

ER_INVALID_FIELD_SIZE was added in 5.7.2.

- Error: 3014 SQLSTATE: HY000 (ER_MISSING_HA_CREATE_OPTION)

Message: Table storage engine '%s' found required create option missing

ER_MISSING_HA_CREATE_OPTION was added in 5.7.2.

- Error: 3015 SQLSTATE: HY000 (ER_ENGINE_OUT_OF_MEMORY)

Message: Out of memory in storage engine '%s'.

ER_ENGINE_OUT_OF_MEMORY was added in 5.7.3.

- Error: 3016 SQLSTATE: HY000 (ER_PASSWORD_EXPIRE_ANONYMOUS_USER)

Message: The password for anonymous user cannot be expired.

ER_PASSWORD_EXPIRE_ANONYMOUS_USER was added in 5.7.3.

- Error: 3017 SQLSTATE: HY000 (ER_SLAVE_SQL_THREAD_MUST_STOP)

Message: This operation cannot be performed with a running slave sql thread; run STOP SLAVE SQL_THREAD first

ER_SLAVE_SQL_THREAD_MUST_STOP was added in 5.7.3.

- Error: 3018 SQLSTATE: HY000 (ER_NO_FT_MATERIALIZED_SUBQUERY)

Message: Cannot create FULLTEXT index on materialized subquery

ER_NO_FT_MATERIALIZED_SUBQUERY was added in 5.7.4.

- Error: 3019 SQLSTATE: HY000 (ER_INNODB_UNDO_LOG_FULL)

Message: Undo Log - Error: %s

ER_INNODB_UNDO_LOG_FULL was added in 5.7.4.

- Error: 3020 SQLSTATE: 2201E (ER_INVALID_ARGUMENT_FOR_LOGARITHM)

Message: Invalid argument for logarithm

ER_INVALID_ARGUMENT_FOR_LOGARITHM was added in 5.7.4.

- Error: 3021 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_IO_THREAD_MUST_STOP)

Message: This operation cannot be performed with a running slave io thread; run STOP SLAVE IO_THREAD FOR CHANNEL '%s' first.

ER_SLAVE_CHANNEL_IO_THREAD_MUST_STOP was added in 5.7.6.

- Error: 3022 SQLSTATE: HY000 (ER_WARN_OPEN_TEMP_TABLES_MUST_BE_ZERO)

Message: This operation may not be safe when the slave has temporary tables. The tables will be kept open until the server restarts or until the tables are deleted by any replicated DROP statement. Suggest to wait until slave_open_temp_tables = 0.

ER_WARN_OPEN_TEMP_TABLES_MUST_BE_ZERO was added in 5.7.4.

- Error: 3023 SQLSTATE: HY000 (ER_WARN_ONLY_MASTER_LOG_FILE_NO_POS)

Message: CHANGE MASTER TO with a MASTER_LOG_FILE clause but no MASTER_LOG_POS clause may not be safe. The old position value may not be valid for the new binary log file.

ER_WARN_ONLY_MASTER_LOG_FILE_NO_POS was added in 5.7.4.

- Error: 3024 SQLSTATE: HY000 (ER_QUERY_TIMEOUT)

Message: Query execution was interrupted, maximum statement execution time exceeded

ER_QUERY_TIMEOUT was added in 5.7.4.

- Error: 3025 SQLSTATE: HY000 (ER_NON_RO_SELECT_DISABLE_TIMER)

Message: Select is not a read only statement, disabling timer

ER_NON_RO_SELECT_DISABLE_TIMER was added in 5.7.4.

- Error: 3026 SQLSTATE: HY000 (ER_DUP_LIST_ENTRY)

Message: Duplicate entry '%s'.

ER_DUP_LIST_ENTRY was added in 5.7.4.

- Error: 3027 SQLSTATE: HY000 (ER_SQL_MODE_NO_EFFECT)

Message: '%s' mode no longer has any effect. Use STRICT_ALL_TABLES or STRICT_TRANS_TABLES instead.

ER_SQL_MODE_NO_EFFECT was added in 5.7.4.

- Error: 3028 SQLSTATE: HY000 (ER_AGGREGATE_ORDER_FOR_UNION)

Message: Expression #%u of ORDER BY contains aggregate function and applies to a UNION

ER_AGGREGATE_ORDER_FOR_UNION was added in 5.7.5.

- Error: 3029 SQLSTATE: HY000 (ER_AGGREGATE_ORDER_NON_AGG_QUERY)

Message: Expression #%u of ORDER BY contains aggregate function and applies to the result of a non-aggregated query

ER_AGGREGATE_ORDER_NON_AGG_QUERY was added in 5.7.5.

- Error: 3030 SQLSTATE: HY000 (ER_SLAVE_WORKER_STOPPED_PREVIOUS_THD_ERROR)

Message: Slave worker has stopped after at least one previous worker encountered an error when slave-preserve-commit-order was enabled. To preserve commit order, the last transaction executed by this thread has not been committed. When restarting the slave after fixing any failed threads, you should fix this worker as well.

ER_SLAVE_WORKER_STOPPED_PREVIOUS_THD_ERROR was added in 5.7.5.

- Error: 3031 SQLSTATE: HY000 (ER_DONT_SUPPORT_SLAVE_PRESERVE_COMMIT_ORDER)

Message: slave_preserve_commit_order is not supported %s.

ER_DONT_SUPPORT_SLAVE_PRESERVE_COMMIT_ORDER was added in 5.7.5.

- Error: 3032 SQLSTATE: HY000 (ER_SERVER_OFFLINE_MODE)

Message: The server is currently in offline mode

ER_SERVER_OFFLINE_MODE was added in 5.7.5.

- Error: 3033 SQLSTATE: HY000 (ER_GIS_DIFFERENT_SRIDS)

Message: Binary geometry function %s given two geometries of different srids: %u and %u, which should have been identical.

Geometry values passed as arguments to spatial functions must have the same SRID value.

ER_GIS_DIFFERENT_SRIDS was added in 5.7.5.

- Error: 3034 SQLSTATE: HY000 (ER_GIS_UNSUPPORTED_ARGUMENT)

Message: Calling geometry function %s with unsupported types of arguments.

A spatial function was called with a combination of argument types that the function does not support.

ER_GIS_UNSUPPORTED_ARGUMENT was added in 5.7.5.

- Error: 3035 SQLSTATE: HY000 (ER_GIS_UNKNOWN_ERROR)

Message: Unknown GIS error occured in function %s.

ER_GIS_UNKNOWN_ERROR was added in 5.7.5.

- Error: 3036 SQLSTATE: HY000 (ER_GIS_UNKNOWN_EXCEPTION)

Message: Unknown exception caught in GIS function %s.

ER_GIS_UNKNOWN_EXCEPTION was added in 5.7.5.

- Error: 3037 SQLSTATE: 22023 (ER_GIS_INVALID_DATA)

Message: Invalid GIS data provided to function %s.

A spatial function was called with an argument not recognized as a valid geometry value.

ER_GIS_INVALID_DATA was added in 5.7.5.

- Error: 3038 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_EMPTY_INPUT_EXCEPTION)

Message: The geometry has no data in function %s.

ER_BOOST_GEOMETRY_EMPTY_INPUT_EXCEPTION was added in 5.7.5.

- Error: 3039 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_CENTROID_EXCEPTION)

Message: Unable to calculate centroid because geometry is empty in function %s.

ER_BOOST_GEOMETRY_CENTROID_EXCEPTION was added in 5.7.5.

- Error: 3040 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_OVERLAY_INVALID_INPUT_EXCEPTION)

Message: Geometry overlay calculation - Error: geometry data is invalid in function %s.

ER_BOOST_GEOMETRY_OVERLAY_INVALID_INPUT_EXCEPTION was added in 5.7.5.

- Error: 3041 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_TURN_INFO_EXCEPTION)

Message: Geometry turn info calculation - Error: geometry data is invalid in function %s.

ER_BOOST_GEOMETRY_TURN_INFO_EXCEPTION was added in 5.7.5.

- Error: 3042 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_SELF_INTERSECTION_POINT_EXCEPTION)

Message: Analysis procedures of intersection points interrupted unexpectedly in function %s.

ER_BOOST_GEOMETRY_SELF_INTERSECTION_POINT_EXCEPTION was added in 5.7.5.

- Error: 3043 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_UNKNOWN_EXCEPTION)

Message: Unknown exception thrown in function %s.

ER_BOOST_GEOMETRY_UNKNOWN_EXCEPTION was added in 5.7.5.

- Error: 3044 SQLSTATE: HY000 (ER_STD_BAD_ALLOC_ERROR)

Message: Memory allocation - Error: %s in function %s.

ER_STD_BAD_ALLOC_ERROR was added in 5.7.5.

- Error: 3045 SQLSTATE: HY000 (ER_STD_DOMAIN_ERROR)

Message: Domain - Error: %s in function %s.

ER_STD_DOMAIN_ERROR was added in 5.7.5.

- Error: 3046 SQLSTATE: HY000 (ER_STD_LENGTH_ERROR)

Message: Length - Error: %s in function %s.

ER_STD_LENGTH_ERROR was added in 5.7.5.

- Error: 3047 SQLSTATE: HY000 (ER_STD_INVALID_ARGUMENT)

Message: Invalid argument - Error: %s in function %s.

ER_STD_INVALID_ARGUMENT was added in 5.7.5.

- Error: 3048 SQLSTATE: HY000 (ER_STD_OUT_OF_RANGE_ERROR)

Message: Out of range - Error: %s in function %s.

ER_STD_OUT_OF_RANGE_ERROR was added in 5.7.5.

- Error: 3049 SQLSTATE: HY000 (ER_STD_OVERFLOW_ERROR)

Message: Overflow error - Error: %s in function %s.

ER_STD_OVERFLOW_ERROR was added in 5.7.5.

- Error: 3050 SQLSTATE: HY000 (ER_STD_RANGE_ERROR)

Message: Range - Error: %s in function %s.

ER_STD_RANGE_ERROR was added in 5.7.5.

- Error: 3051 SQLSTATE: HY000 (ER_STD_UNDERFLOW_ERROR)

Message: Underflow - Error: %s in function %s.

ER_STD_UNDERFLOW_ERROR was added in 5.7.5.

- Error: 3052 SQLSTATE: HY000 (ER_STD_LOGIC_ERROR)

Message: Logic - Error: %s in function %s.

ER_STD_LOGIC_ERROR was added in 5.7.5.

- Error: 3053 SQLSTATE: HY000 (ER_STD_RUNTIME_ERROR)

Message: Runtime - Error: %s in function %s.

ER_STD_RUNTIME_ERROR was added in 5.7.5.

- Error: 3054 SQLSTATE: HY000 (ER_STD_UNKNOWN_EXCEPTION)

Message: Unknown exception: %s in function %s.

ER_STD_UNKNOWN_EXCEPTION was added in 5.7.5.

- Error: 3055 SQLSTATE: HY000 (ER_GIS_DATA_WRONG_ENDIANESS)

Message: Geometry byte string must be little endian.

ER_GIS_DATA_WRONG_ENDIANESS was added in 5.7.5.

- Error: 3056 SQLSTATE: HY000 (ER_CHANGE_MASTER_PASSWORD_LENGTH)

Message: The password provided for the replication user exceeds the maximum length of 32 characters

ER_CHANGE_MASTER_PASSWORD_LENGTH was added in 5.7.5.

- Error: 3057 SQLSTATE: 42000 (ER_USER_LOCK_WRONG_NAME)

Message: Incorrect user-level lock name '%s'.

ER_USER_LOCK_WRONG_NAME was added in 5.7.5.

- Error: 3058 SQLSTATE: HY000 (ER_USER_LOCK_DEADLOCK)

Message: Deadlock found when trying to get user-level lock; try rolling back transaction/releasing locks and restarting lock acquisition.

This error is returned when the metdata locking subsystem detects a deadlock for an attempt to acquire a named lock with GET_LOCK.

ER_USER_LOCK_DEADLOCK was added in 5.7.5.

- Error: 3059 SQLSTATE: HY000 (ER_REPLACE_INACCESSIBLE_ROWS)

Message: REPLACE cannot be executed as it requires deleting rows that are not in the view

ER_REPLACE_INACCESSIBLE_ROWS was added in 5.7.5.

- Error: 3060 SQLSTATE: HY000 (ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_GIS)

Message: Do not support online operation on table with GIS index

ER_ALTER_OPERATION_NOT_SUPPORTED_REASON_GIS was added in 5.7.5.

- Error: 3061 SQLSTATE: 42000 (ER_ILLEGAL_USER_VAR)

Message: User variable name '%s' is illegal

ER_ILLEGAL_USER_VAR was added in 5.7.5.

- Error: 3062 SQLSTATE: HY000 (ER_GTID_MODE_OFF)

Message: Cannot %s when GTID_MODE = OFF.

ER_GTID_MODE_OFF was added in 5.7.5.

- Error: 3063 SQLSTATE: HY000 (ER_UNSUPPORTED_BY_REPLICATION_THREAD)

Message: Cannot %s from a replication slave thread.

ER_UNSUPPORTED_BY_REPLICATION_THREAD was added in 5.7.5.

- Error: 3064 SQLSTATE: HY000 (ER_INCORRECT_TYPE)

Message: Incorrect type for argument %s in function %s.

ER_INCORRECT_TYPE was added in 5.7.5.

- Error: 3065 SQLSTATE: HY000 (ER_FIELD_IN_ORDER_NOT_SELECT)

Message: Expression #%u of ORDER BY clause is not in SELECT list, references column '%s' which is not in SELECT list; this is incompatible with %s

ER_FIELD_IN_ORDER_NOT_SELECT was added in 5.7.5.

- Error: 3066 SQLSTATE: HY000 (ER_AGGREGATE_IN_ORDER_NOT_SELECT)

Message: Expression #%u of ORDER BY clause is not in SELECT list, contains aggregate function; this is incompatible with %s

ER_AGGREGATE_IN_ORDER_NOT_SELECT was added in 5.7.5.

- Error: 3067 SQLSTATE: HY000 (ER_INVALID_RPL_WILD_TABLE_FILTER_PATTERN)

Message: Supplied filter list contains a value which is not in the required format 'db_pattern.table_pattern'

ER_INVALID_RPL_WILD_TABLE_FILTER_PATTERN was added in 5.7.5.

- Error: 3068 SQLSTATE: 08S01 (ER_NET_OK_PACKET_TOO_LARGE)

Message: OK packet too large

ER_NET_OK_PACKET_TOO_LARGE was added in 5.7.5.

- Error: 3069 SQLSTATE: HY000 (ER_INVALID_JSON_DATA)

Message: Invalid JSON data provided to function %s: %s

ER_INVALID_JSON_DATA was added in 5.7.5.

- Error: 3070 SQLSTATE: HY000 (ER_INVALID_GEOJSON_MISSING_MEMBER)

Message: Invalid GeoJSON data provided to function %s: Missing required member '%s'

ER_INVALID_GEOJSON_MISSING_MEMBER was added in 5.7.5.

- Error: 3071 SQLSTATE: HY000 (ER_INVALID_GEOJSON_WRONG_TYPE)

Message: Invalid GeoJSON data provided to function %s: Member '%s' must be of type '%s'

ER_INVALID_GEOJSON_WRONG_TYPE was added in 5.7.5.

- Error: 3072 SQLSTATE: HY000 (ER_INVALID_GEOJSON_UNSPECIFIED)

Message: Invalid GeoJSON data provided to function %s

ER_INVALID_GEOJSON_UNSPECIFIED was added in 5.7.5.

- Error: 3073 SQLSTATE: HY000 (ER_DIMENSION_UNSUPPORTED)

Message: Unsupported number of coordinate dimensions in function %s: Found %u, expected %u

ER_DIMENSION_UNSUPPORTED was added in 5.7.5.

- Error: 3074 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_DOES_NOT_EXIST)

Message: Slave channel '%s' does not exist.

ER_SLAVE_CHANNEL_DOES_NOT_EXIST was added in 5.7.6.

- Error: 3075 SQLSTATE: HY000 (ER_SLAVE_MULTIPLE_CHANNELS_HOST_PORT)

Message: A slave channel '%s' already exists for the given host and port combination.

ER_SLAVE_MULTIPLE_CHANNELS_HOST_PORT was added in 5.7.6.

- Error: 3076 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_NAME_INVALID_OR_TOO_LONG)

Message: Couldn't create channel: Channel name is either invalid or too long.

ER_SLAVE_CHANNEL_NAME_INVALID_OR_TOO_LONG was added in 5.7.6.

- Error: 3077 SQLSTATE: HY000 (ER_SLAVE_NEW_CHANNEL_WRONG_REPOSITORY)

Message: To have multiple channels, repository cannot be of type FILE; Please check the repository configuration and convert them to TABLE.

ER_SLAVE_NEW_CHANNEL_WRONG_REPOSITORY was added in 5.7.6.

- Error: 3078 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_DELETE)

Message: Cannot delete slave info objects for channel '%s'.

ER_SLAVE_CHANNEL_DELETE was added in 5.7.6.

- Error: 3079 SQLSTATE: HY000 (ER_SLAVE_MULTIPLE_CHANNELS_CMD)

Message: Multiple channels exist on the slave. Please provide channel name as an argument.

ER_SLAVE_MULTIPLE_CHANNELS_CMD was added in 5.7.6.

- Error: 3080 SQLSTATE: HY000 (ER_SLAVE_MAX_CHANNELS_EXCEEDED)

Message: Maximum number of replication channels allowed exceeded.

ER_SLAVE_MAX_CHANNELS_EXCEEDED was added in 5.7.6.

- Error: 3081 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_MUST_STOP)

Message: This operation cannot be performed with running replication threads; run STOP SLAVE FOR CHANNEL '%s' first

ER_SLAVE_CHANNEL_MUST_STOP was added in 5.7.6.

- Error: 3082 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_NOT_RUNNING)

Message: This operation requires running replication threads; configure slave and run START SLAVE FOR CHANNEL '%s'

ER_SLAVE_CHANNEL_NOT_RUNNING was added in 5.7.6.

- Error: 3083 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_WAS_RUNNING)

Message: Replication thread(s) for channel '%s' are already runnning.

ER_SLAVE_CHANNEL_WAS_RUNNING was added in 5.7.6.

- Error: 3084 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_WAS_NOT_RUNNING)

Message: Replication thread(s) for channel '%s' are already stopped.

ER_SLAVE_CHANNEL_WAS_NOT_RUNNING was added in 5.7.6.

- Error: 3085 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_SQL_THREAD_MUST_STOP)

Message: This operation cannot be performed with a running slave sql thread; run STOP SLAVE SQL_THREAD FOR CHANNEL '%s' first.

ER_SLAVE_CHANNEL_SQL_THREAD_MUST_STOP was added in 5.7.6.

- Error: 3086 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_SQL_SKIP_COUNTER)

Message: When sql_slave_skip_counter > 0, it is not allowed to start more than one SQL thread by using 'START SLAVE [SQL_THREAD]'. Value of sql_slave_skip_counter can only be used by one SQL thread at a time. Please use 'START SLAVE [SQL_THREAD] FOR CHANNEL' to start the SQL thread which will use the value of sql_slave_skip_counter.

ER_SLAVE_CHANNEL_SQL_SKIP_COUNTER was added in 5.7.6.

- Error: 3087 SQLSTATE: HY000 (ER_WRONG_FIELD_WITH_GROUP_V2)

Message: Expression #%u of %s is not in GROUP BY clause and contains nonaggregated column '%s' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

ER_WRONG_FIELD_WITH_GROUP_V2 was added in 5.7.6.

- Error: 3088 SQLSTATE: HY000 (ER_MIX_OF_GROUP_FUNC_AND_FIELDS_V2)

Message: In aggregated query without GROUP BY, expression #%u of %s contains nonaggregated column '%s'; this is incompatible with sql_mode=only_full_group_by

ER_MIX_OF_GROUP_FUNC_AND_FIELDS_V2 was added in 5.7.6.

- Error: 3089 SQLSTATE: HY000 (ER_WARN_DEPRECATED_SYSVAR_UPDATE)

Message: Updating '%s' is deprecated. It will be made read-only in a future release.

ER_WARN_DEPRECATED_SYSVAR_UPDATE was added in 5.7.6.

- Error: 3090 SQLSTATE: HY000 (ER_WARN_DEPRECATED_SQLMODE)

Message: Changing sql mode '%s' is deprecated. It will be removed in a future release.

ER_WARN_DEPRECATED_SQLMODE was added in 5.7.6.

- Error: 3091 SQLSTATE: HY000 (ER_CANNOT_LOG_PARTIAL_DROP_DATABASE_WITH_GTID)

Message: DROP DATABASE failed; some tables may have been dropped but the database directory remains. The GTID has not been added to GTID_EXECUTED and the statement was not written to the binary log. Fix this as follows: (1) remove all files from the database directory %s; (2) SET GTID_NEXT='%s'; (3) DROP DATABASE `%s`.

ER_CANNOT_LOG_PARTIAL_DROP_DATABASE_WITH_GTID was added in 5.7.6.

- Error: 3092 SQLSTATE: HY000 (ER_GROUP_REPLICATION_CONFIGURATION)

Message: The server is not configured properly to be an active member of the group. Please see more details on error log.

ER_GROUP_REPLICATION_CONFIGURATION was added in 5.7.6.

- Error: 3093 SQLSTATE: HY000 (ER_GROUP_REPLICATION_RUNNING)

Message: The START GROUP_REPLICATION command failed since the group is already running.

ER_GROUP_REPLICATION_RUNNING was added in 5.7.6.

- Error: 3094 SQLSTATE: HY000 (ER_GROUP_REPLICATION_APPLIER_INIT_ERROR)

Message: The START GROUP_REPLICATION command failed as the applier module failed to start.

ER_GROUP_REPLICATION_APPLIER_INIT_ERROR was added in 5.7.6.

- Error: 3095 SQLSTATE: HY000 (ER_GROUP_REPLICATION_STOP_APPLIER_THREAD_TIMEOUT)

Message: The STOP GROUP_REPLICATION command execution is incomplete: The applier thread got the stop signal while it was busy. The applier thread will stop once the current task is complete.

ER_GROUP_REPLICATION_STOP_APPLIER_THREAD_TIMEOUT was added in 5.7.6.

- Error: 3096 SQLSTATE: HY000 (ER_GROUP_REPLICATION_COMMUNICATION_LAYER_SESSION_ERROR)

Message: The START GROUP_REPLICATION command failed as there was an error when initializing the group communication layer.

ER_GROUP_REPLICATION_COMMUNICATION_LAYER_SESSION_ERROR was added in 5.7.6.

- Error: 3097 SQLSTATE: HY000 (ER_GROUP_REPLICATION_COMMUNICATION_LAYER_JOIN_ERROR)

Message: The START GROUP_REPLICATION command failed as there was an error when joining the communication group.

ER_GROUP_REPLICATION_COMMUNICATION_LAYER_JOIN_ERROR was added in 5.7.6.

- Error: 3098 SQLSTATE: HY000 (ER_BEFORE_DML_VALIDATION_ERROR)

Message: The table does not comply with the requirements by an external plugin.

ER_BEFORE_DML_VALIDATION_ERROR was added in 5.7.6.

- Error: 3099 SQLSTATE: HY000 (ER_PREVENTS_VARIABLE_WITHOUT_RBR)

Message: Cannot change the value of variable %s without binary log format as ROW.

transaction_write_set_extraction option value is set and binlog_format is not ROW.

ER_PREVENTS_VARIABLE_WITHOUT_RBR was added in 5.7.6.

- Error: 3100 SQLSTATE: HY000 (ER_RUN_HOOK_ERROR)

Message: Error on observer while running replication hook '%s'.

ER_RUN_HOOK_ERROR was added in 5.7.6.

- Error: 3101 SQLSTATE: HY000 (ER_TRANSACTION_ROLLBACK_DURING_COMMIT)

Message: Plugin instructed the server to rollback the current transaction.

When using Group Replication, this means that a transaction failed the group certification process, due to one or more members detecting a potential conflict, and was thus rolled back. See Chapter 17, Group Replication.

ER_TRANSACTION_ROLLBACK_DURING_COMMIT was added in 5.7.6.

- Error: 3102 SQLSTATE: HY000 (ER_GENERATED_COLUMN_FUNCTION_IS_NOT_ALLOWED)

Message: Expression of generated column '%s' contains a disallowed function.

ER_GENERATED_COLUMN_FUNCTION_IS_NOT_ALLOWED was added in 5.7.6.

- Error: 3103 SQLSTATE: HY000 (ER_KEY_BASED_ON_GENERATED_COLUMN)

Message: Key/Index cannot be defined on a virtual generated column.

ER_KEY_BASED_ON_GENERATED_COLUMN was added in 5.7.6, removed after 5.7.7.

- Error: 3103 SQLSTATE: HY000 (ER_UNSUPPORTED_ALTER_INPLACE_ON_VIRTUAL_COLUMN)

Message: INPLACE ADD or DROP of virtual columns cannot be combined with other ALTER TABLE actions

ER_UNSUPPORTED_ALTER_INPLACE_ON_VIRTUAL_COLUMN was added in 5.7.8.

- Error: 3104 SQLSTATE: HY000 (ER_WRONG_FK_OPTION_FOR_GENERATED_COLUMN)

Message: Cannot define foreign key with %s clause on a generated column.

ER_WRONG_FK_OPTION_FOR_GENERATED_COLUMN was added in 5.7.6.

- Error: 3105 SQLSTATE: HY000 (ER_NON_DEFAULT_VALUE_FOR_GENERATED_COLUMN)

Message: The value specified for generated column '%s' in table '%s' is not allowed.

ER_NON_DEFAULT_VALUE_FOR_GENERATED_COLUMN was added in 5.7.6.

- Error: 3106 SQLSTATE: HY000 (ER_UNSUPPORTED_ACTION_ON_GENERATED_COLUMN)

Message: '%s' is not supported for generated columns.

ER_UNSUPPORTED_ACTION_ON_GENERATED_COLUMN was added in 5.7.6.

- Error: 3107 SQLSTATE: HY000 (ER_GENERATED_COLUMN_NON_PRIOR)

Message: Generated column can refer only to generated columns defined prior to it.

To address this issue, change the table definition to define each generated column later than any generated columns to which it refers.

ER_GENERATED_COLUMN_NON_PRIOR was added in 5.7.6.

- Error: 3108 SQLSTATE: HY000 (ER_DEPENDENT_BY_GENERATED_COLUMN)

Message: Column '%s' has a generated column dependency.

You cannot drop or rename a generated column if another column refers to it. You must either drop those columns as well, or redefine them not to refer to the generated column.

ER_DEPENDENT_BY_GENERATED_COLUMN was added in 5.7.6.

- Error: 3109 SQLSTATE: HY000 (ER_GENERATED_COLUMN_REF_AUTO_INC)

Message: Generated column '%s' cannot refer to auto-increment column.

ER_GENERATED_COLUMN_REF_AUTO_INC was added in 5.7.6.

- Error: 3110 SQLSTATE: HY000 (ER_FEATURE_NOT_AVAILABLE)

Message: The '%s' feature is not available; you need to remove '%s' or use MySQL built with '%s'

ER_FEATURE_NOT_AVAILABLE was added in 5.7.6.

- Error: 3111 SQLSTATE: HY000 (ER_CANT_SET_GTID_MODE)

Message: SET @@GLOBAL.GTID_MODE = %s is not allowed because %s.

ER_CANT_SET_GTID_MODE was added in 5.7.6.

- Error: 3112 SQLSTATE: HY000 (ER_CANT_USE_AUTO_POSITION_WITH_GTID_MODE_OFF)

Message: The replication receiver thread%s cannot start in AUTO_POSITION mode: this server uses @@GLOBAL.GTID_MODE = OFF.

ER_CANT_USE_AUTO_POSITION_WITH_GTID_MODE_OFF was added in 5.7.6.

- Error: 3113 SQLSTATE: HY000 (ER_CANT_REPLICATE_ANONYMOUS_WITH_AUTO_POSITION)

Message: Cannot replicate anonymous transaction when AUTO_POSITION = 1, at file %s, position %lld.

ER_CANT_REPLICATE_ANONYMOUS_WITH_AUTO_POSITION was added in 5.7.6.

- Error: 3114 SQLSTATE: HY000 (ER_CANT_REPLICATE_ANONYMOUS_WITH_GTID_MODE_ON)

Message: Cannot replicate anonymous transaction when @@GLOBAL.GTID_MODE = ON, at file %s, position %lld.

ER_CANT_REPLICATE_ANONYMOUS_WITH_GTID_MODE_ON was added in 5.7.6.

- Error: 3115 SQLSTATE: HY000 (ER_CANT_REPLICATE_GTID_WITH_GTID_MODE_OFF)

Message: Cannot replicate GTID-transaction when @@GLOBAL.GTID_MODE = OFF, at file %s, position %lld.

ER_CANT_REPLICATE_GTID_WITH_GTID_MODE_OFF was added in 5.7.6.

- Error: 3116 SQLSTATE: HY000 (ER_CANT_SET_ENFORCE_GTID_CONSISTENCY_ON_WITH_ONGOING_GTID_VIOLATING_TRANSACTIONS)

Message: Cannot set ENFORCE_GTID_CONSISTENCY = ON because there are ongoing transactions that violate GTID consistency.

ER_CANT_SET_ENFORCE_GTID_CONSISTENCY_ON_WITH_ONGOING_GTID_VIOLATING_TRANSACTIONS is renamed to ER_CANT_ENFORCE_GTID_CONSISTENCY_WITH_ONGOING_GTID_VIOLATING_TX in MySQL 8.0.

ER_CANT_SET_ENFORCE_GTID_CONSISTENCY_ON_WITH_ONGOING_GTID_VIOLATING_TRANSACTIONS was added in 5.7.6.

- Error: 3117 SQLSTATE: HY000 (ER_SET_ENFORCE_GTID_CONSISTENCY_WARN_WITH_ONGOING_GTID_VIOLATING_TRANSACTIONS)

Message: There are ongoing transactions that violate GTID consistency.

ER_SET_ENFORCE_GTID_CONSISTENCY_WARN_WITH_ONGOING_GTID_VIOLATING_TRANSACTIONS is renamed to ER_ENFORCE_GTID_CONSISTENCY_WARN_WITH_ONGOING_GTID_VIOLATING_TX in MySQL 8.0.

ER_SET_ENFORCE_GTID_CONSISTENCY_WARN_WITH_ONGOING_GTID_VIOLATING_TRANSACTIONS was added in 5.7.6.

- Error: 3118 SQLSTATE: HY000 (ER_ACCOUNT_HAS_BEEN_LOCKED)

Message: Access denied for user '%s'@'%s'. Account is locked.

The account was locked with CREATE USER ... ACCOUNT LOCK or ALTER USER ... ACCOUNT LOCK. An administrator can unlock it with ALTER USER ... ACCOUNT UNLOCK.

ER_ACCOUNT_HAS_BEEN_LOCKED was added in 5.7.6.

- Error: 3119 SQLSTATE: 42000 (ER_WRONG_TABLESPACE_NAME)

Message: Incorrect tablespace name `%s`

ER_WRONG_TABLESPACE_NAME was added in 5.7.6.

- Error: 3120 SQLSTATE: HY000 (ER_TABLESPACE_IS_NOT_EMPTY)

Message: Tablespace `%s` is not empty.

ER_TABLESPACE_IS_NOT_EMPTY was added in 5.7.6.

- Error: 3121 SQLSTATE: HY000 (ER_WRONG_FILE_NAME)

Message: Incorrect File Name '%s'.

ER_WRONG_FILE_NAME was added in 5.7.6.

- Error: 3122 SQLSTATE: HY000 (ER_BOOST_GEOMETRY_INCONSISTENT_TURNS_EXCEPTION)

Message: Inconsistent intersection points.

ER_BOOST_GEOMETRY_INCONSISTENT_TURNS_EXCEPTION was added in 5.7.7.

- Error: 3123 SQLSTATE: HY000 (ER_WARN_OPTIMIZER_HINT_SYNTAX_ERROR)

Message: Optimizer hint syntax error

ER_WARN_OPTIMIZER_HINT_SYNTAX_ERROR was added in 5.7.7.

- Error: 3124 SQLSTATE: HY000 (ER_WARN_BAD_MAX_EXECUTION_TIME)

Message: Unsupported MAX_EXECUTION_TIME

ER_WARN_BAD_MAX_EXECUTION_TIME was added in 5.7.7.

- Error: 3125 SQLSTATE: HY000 (ER_WARN_UNSUPPORTED_MAX_EXECUTION_TIME)

Message: MAX_EXECUTION_TIME hint is supported by top-level standalone SELECT statements only

The MAX_EXECUTION_TIME optimizer hint is supported only for SELECT statements.

ER_WARN_UNSUPPORTED_MAX_EXECUTION_TIME was added in 5.7.7.

- Error: 3126 SQLSTATE: HY000 (ER_WARN_CONFLICTING_HINT)

Message: Hint %s is ignored as conflicting/duplicated

ER_WARN_CONFLICTING_HINT was added in 5.7.7.

- Error: 3127 SQLSTATE: HY000 (ER_WARN_UNKNOWN_QB_NAME)

Message: Query block name %s is not found for %s hint

ER_WARN_UNKNOWN_QB_NAME was added in 5.7.7.

- Error: 3128 SQLSTATE: HY000 (ER_UNRESOLVED_HINT_NAME)

Message: Unresolved name %s for %s hint

ER_UNRESOLVED_HINT_NAME was added in 5.7.7.

- Error: 3129 SQLSTATE: HY000 (ER_WARN_DEPRECATED_SQLMODE_UNSET)

Message: Unsetting sql mode '%s' is deprecated. It will be made read-only in a future release.

ER_WARN_DEPRECATED_SQLMODE_UNSET was added in 5.7.7, removed after 5.7.7.

- Error: 3129 SQLSTATE: HY000 (ER_WARN_ON_MODIFYING_GTID_EXECUTED_TABLE)

Message: Please do not modify the %s table. This is a mysql internal system table to store GTIDs for committed transactions. Modifying it can lead to an inconsistent GTID state.

ER_WARN_ON_MODIFYING_GTID_EXECUTED_TABLE was added in 5.7.8.

- Error: 3130 SQLSTATE: HY000 (ER_PLUGGABLE_PROTOCOL_COMMAND_NOT_SUPPORTED)

Message: Command not supported by pluggable protocols

ER_PLUGGABLE_PROTOCOL_COMMAND_NOT_SUPPORTED was added in 5.7.8.

- Error: 3131 SQLSTATE: 42000 (ER_LOCKING_SERVICE_WRONG_NAME)

Message: Incorrect locking service lock name '%s'.

A locking service name was specified as NULL, the empty string, or a string longer than 64 characters. Namespace and lock names must be non-NULL, nonempty, and no more than 64 characters long.

ER_LOCKING_SERVICE_WRONG_NAME was added in 5.7.8.

- Error: 3132 SQLSTATE: HY000 (ER_LOCKING_SERVICE_DEADLOCK)

Message: Deadlock found when trying to get locking service lock; try releasing locks and restarting lock acquisition.

ER_LOCKING_SERVICE_DEADLOCK was added in 5.7.8.

- Error: 3133 SQLSTATE: HY000 (ER_LOCKING_SERVICE_TIMEOUT)

Message: Service lock wait timeout exceeded.

ER_LOCKING_SERVICE_TIMEOUT was added in 5.7.8.

- Error: 3134 SQLSTATE: HY000 (ER_GIS_MAX_POINTS_IN_GEOMETRY_OVERFLOWED)

Message: Parameter %s exceeds the maximum number of points in a geometry (%lu) in function %s.

ER_GIS_MAX_POINTS_IN_GEOMETRY_OVERFLOWED was added in 5.7.8.

- Error: 3135 SQLSTATE: HY000 (ER_SQL_MODE_MERGED)

Message: 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.

ER_SQL_MODE_MERGED was added in 5.7.8.

- Error: 3136 SQLSTATE: HY000 (ER_VTOKEN_PLUGIN_TOKEN_MISMATCH)

Message: Version token mismatch for %.*s. Correct value %.*s

The client has set its version_tokens_session system variable to the list of tokens it requires the server to match, but the server token list has at least one matching token name that has a value different from what the client requires. See Section 5.5.5, “Version Tokens”.

ER_VTOKEN_PLUGIN_TOKEN_MISMATCH was added in 5.7.8.

- Error: 3137 SQLSTATE: HY000 (ER_VTOKEN_PLUGIN_TOKEN_NOT_FOUND)

Message: Version token %.*s not found.

The client has set its version_tokens_session system variable to the list of tokens it requires the server to match, but the server token list is missing at least one of those tokens. See Section 5.5.5, “Version Tokens”.

ER_VTOKEN_PLUGIN_TOKEN_NOT_FOUND was added in 5.7.8.

- Error: 3138 SQLSTATE: HY000 (ER_CANT_SET_VARIABLE_WHEN_OWNING_GTID)

Message: Variable %s cannot be changed by a client that owns a GTID. The client owns %s. Ownership is released on COMMIT or ROLLBACK.

ER_CANT_SET_VARIABLE_WHEN_OWNING_GTID was added in 5.7.8.

- Error: 3139 SQLSTATE: HY000 (ER_SLAVE_CHANNEL_OPERATION_NOT_ALLOWED)

Message: %s cannot be performed on channel '%s'.

ER_SLAVE_CHANNEL_OPERATION_NOT_ALLOWED was added in 5.7.8.

- Error: 3140 SQLSTATE: 22032 (ER_INVALID_JSON_TEXT)

Message: Invalid JSON text: "%s" at position %u in value for column '%s'.

ER_INVALID_JSON_TEXT was added in 5.7.8.

- Error: 3141 SQLSTATE: 22032 (ER_INVALID_JSON_TEXT_IN_PARAM)

Message: Invalid JSON text in argument %u to function %s: "%s" at position %u.%s

ER_INVALID_JSON_TEXT_IN_PARAM was added in 5.7.8.

- Error: 3142 SQLSTATE: HY000 (ER_INVALID_JSON_BINARY_DATA)

Message: The JSON binary value contains invalid data.

ER_INVALID_JSON_BINARY_DATA was added in 5.7.8.

- Error: 3143 SQLSTATE: 42000 (ER_INVALID_JSON_PATH)

Message: Invalid JSON path expression. The error is around character position %u.%s

ER_INVALID_JSON_PATH was added in 5.7.8.

- Error: 3144 SQLSTATE: 22032 (ER_INVALID_JSON_CHARSET)

Message: Cannot create a JSON value from a string with CHARACTER SET '%s'.

ER_INVALID_JSON_CHARSET was added in 5.7.8.

- Error: 3145 SQLSTATE: 22032 (ER_INVALID_JSON_CHARSET_IN_FUNCTION)

Message: Invalid JSON character data provided to function %s: '%s'; utf8 is required.

ER_INVALID_JSON_CHARSET_IN_FUNCTION was added in 5.7.8.

- Error: 3146 SQLSTATE: 22032 (ER_INVALID_TYPE_FOR_JSON)

Message: Invalid data type for JSON data in argument %u to function %s; a JSON string or JSON type is required.

ER_INVALID_TYPE_FOR_JSON was added in 5.7.8.

- Error: 3147 SQLSTATE: 22032 (ER_INVALID_CAST_TO_JSON)

Message: Cannot CAST value to JSON.

ER_INVALID_CAST_TO_JSON was added in 5.7.8.

- Error: 3148 SQLSTATE: 42000 (ER_INVALID_JSON_PATH_CHARSET)

Message: A path expression must be encoded in the utf8 character set. The path expression '%s' is encoded in character set '%s'.

ER_INVALID_JSON_PATH_CHARSET was added in 5.7.8.

- Error: 3149 SQLSTATE: 42000 (ER_INVALID_JSON_PATH_WILDCARD)

Message: In this situation, path expressions may not contain the * and ** tokens.

ER_INVALID_JSON_PATH_WILDCARD was added in 5.7.8.

- Error: 3150 SQLSTATE: 22032 (ER_JSON_VALUE_TOO_BIG)

Message: The JSON value is too big to be stored in a JSON column.

ER_JSON_VALUE_TOO_BIG was added in 5.7.8.

- Error: 3151 SQLSTATE: 22032 (ER_JSON_KEY_TOO_BIG)

Message: The JSON object contains a key name that is too long.

ER_JSON_KEY_TOO_BIG was added in 5.7.8.

- Error: 3152 SQLSTATE: 42000 (ER_JSON_USED_AS_KEY)

Message: JSON column '%s' cannot be used in key specification.

ER_JSON_USED_AS_KEY was added in 5.7.8.

- Error: 3153 SQLSTATE: 42000 (ER_JSON_VACUOUS_PATH)

Message: The path expression '$' is not allowed in this context.

ER_JSON_VACUOUS_PATH was added in 5.7.8.

- Error: 3154 SQLSTATE: 42000 (ER_JSON_BAD_ONE_OR_ALL_ARG)

Message: The oneOrAll argument to %s may take these values: 'one' or 'all'.

ER_JSON_BAD_ONE_OR_ALL_ARG was added in 5.7.8.

- Error: 3155 SQLSTATE: 22003 (ER_NUMERIC_JSON_VALUE_OUT_OF_RANGE)

Message: Out of range JSON value for CAST to %s%s from column %s at row %ld

ER_NUMERIC_JSON_VALUE_OUT_OF_RANGE was added in 5.7.8.

- Error: 3156 SQLSTATE: 22018 (ER_INVALID_JSON_VALUE_FOR_CAST)

Message: Invalid JSON value for CAST to %s%s from column %s at row %ld

ER_INVALID_JSON_VALUE_FOR_CAST was added in 5.7.8.

- Error: 3157 SQLSTATE: 22032 (ER_JSON_DOCUMENT_TOO_DEEP)

Message: The JSON document exceeds the maximum depth.

ER_JSON_DOCUMENT_TOO_DEEP was added in 5.7.8.

- Error: 3158 SQLSTATE: 22032 (ER_JSON_DOCUMENT_NULL_KEY)

Message: JSON documents may not contain NULL member names.

ER_JSON_DOCUMENT_NULL_KEY was added in 5.7.8.

- Error: 3159 SQLSTATE: HY000 (ER_SECURE_TRANSPORT_REQUIRED)

Message: Connections using insecure transport are prohibited while --require_secure_transport=ON.

With the require_secure_transport system variable, clients can connect only using secure transports. Qualifying connections are those using SSL, a Unix socket file, or shared memory.

ER_SECURE_TRANSPORT_REQUIRED was added in 5.7.8.

- Error: 3160 SQLSTATE: HY000 (ER_NO_SECURE_TRANSPORTS_CONFIGURED)

Message: No secure transports (SSL or Shared Memory) are configured, unable to set --require_secure_transport=ON.

The require_secure_transport system variable cannot be enabled if the server does not support at least one secure transport. Configure the server with the required SSL keys/certificates to enable SSL connections, or enable the shared_memory system variable to enable shared-memory connections.

ER_NO_SECURE_TRANSPORTS_CONFIGURED was added in 5.7.8.

- Error: 3161 SQLSTATE: HY000 (ER_DISABLED_STORAGE_ENGINE)

Message: Storage engine %s is disabled (Table creation is disallowed).

An attempt was made to create a table or tablespace using a storage engine listed in the value of the disabled_storage_engines system variable, or to change an existing table or tablespace to such an engine. Choose a different storage engine.

ER_DISABLED_STORAGE_ENGINE was added in 5.7.8.

- Error: 3162 SQLSTATE: HY000 (ER_USER_DOES_NOT_EXIST)

Message: User %s does not exist.

ER_USER_DOES_NOT_EXIST was added in 5.7.8.

- Error: 3163 SQLSTATE: HY000 (ER_USER_ALREADY_EXISTS)

Message: User %s already exists.

ER_USER_ALREADY_EXISTS was added in 5.7.8.

- Error: 3164 SQLSTATE: HY000 (ER_AUDIT_API_ABORT)

Message: Aborted by Audit API ('%s';%d).

This error indicates that an audit plugin terminated execution of an event. The message typically indicates the event subclass name and a numeric status value.

ER_AUDIT_API_ABORT was added in 5.7.8.

- Error: 3165 SQLSTATE: 42000 (ER_INVALID_JSON_PATH_ARRAY_CELL)

Message: A path expression is not a path to a cell in an array.

ER_INVALID_JSON_PATH_ARRAY_CELL was added in 5.7.8.

- Error: 3166 SQLSTATE: HY000 (ER_BUFPOOL_RESIZE_INPROGRESS)

Message: Another buffer pool resize is already in progress.

ER_BUFPOOL_RESIZE_INPROGRESS was added in 5.7.9.

- Error: 3167 SQLSTATE: HY000 (ER_FEATURE_DISABLED_SEE_DOC)

Message: The '%s' feature is disabled; see the documentation for '%s'

ER_FEATURE_DISABLED_SEE_DOC was added in 5.7.9.

- Error: 3168 SQLSTATE: HY000 (ER_SERVER_ISNT_AVAILABLE)

Message: Server isn't available

ER_SERVER_ISNT_AVAILABLE was added in 5.7.9.

- Error: 3169 SQLSTATE: HY000 (ER_SESSION_WAS_KILLED)

Message: Session was killed

ER_SESSION_WAS_KILLED was added in 5.7.9.

- Error: 3170 SQLSTATE: HY000 (ER_CAPACITY_EXCEEDED)

Message: Memory capacity of %llu bytes for '%s' exceeded. %s

ER_CAPACITY_EXCEEDED was added in 5.7.9.

- Error: 3171 SQLSTATE: HY000 (ER_CAPACITY_EXCEEDED_IN_RANGE_OPTIMIZER)

Message: Range optimization was not done for this query.

ER_CAPACITY_EXCEEDED_IN_RANGE_OPTIMIZER was added in 5.7.9.

- Error: 3172 SQLSTATE: HY000 (ER_TABLE_NEEDS_UPG_PART)

Message: Partitioning upgrade required. Please dump/reload to fix it or do: ALTER TABLE `%s`.`%s` UPGRADE PARTITIONING

ER_TABLE_NEEDS_UPG_PART was added in 5.7.9.

- Error: 3173 SQLSTATE: HY000 (ER_CANT_WAIT_FOR_EXECUTED_GTID_SET_WHILE_OWNING_A_GTID)

Message: The client holds ownership of the GTID %s. Therefore, WAIT_FOR_EXECUTED_GTID_SET cannot wait for this GTID.

ER_CANT_WAIT_FOR_EXECUTED_GTID_SET_WHILE_OWNING_A_GTID was added in 5.7.9.

- Error: 3174 SQLSTATE: HY000 (ER_CANNOT_ADD_FOREIGN_BASE_COL_VIRTUAL)

Message: Cannot add foreign key on the base column of indexed virtual column.

ER_CANNOT_ADD_FOREIGN_BASE_COL_VIRTUAL was added in 5.7.10.

- Error: 3175 SQLSTATE: HY000 (ER_CANNOT_CREATE_VIRTUAL_INDEX_CONSTRAINT)

Message: Cannot create index on virtual column whose base column has foreign constraint.

ER_CANNOT_CREATE_VIRTUAL_INDEX_CONSTRAINT was added in 5.7.10.

- Error: 3176 SQLSTATE: HY000 (ER_ERROR_ON_MODIFYING_GTID_EXECUTED_TABLE)

Message: Please do not modify the %s table with an XA transaction. This is an internal system table used to store GTIDs for committed transactions. Although modifying it can lead to an inconsistent GTID state, if neccessary you can modify it with a non-XA transaction.

ER_ERROR_ON_MODIFYING_GTID_EXECUTED_TABLE was added in 5.7.11.

- Error: 3177 SQLSTATE: HY000 (ER_LOCK_REFUSED_BY_ENGINE)

Message: Lock acquisition refused by storage engine.

ER_LOCK_REFUSED_BY_ENGINE was added in 5.7.11.

- Error: 3178 SQLSTATE: HY000 (ER_UNSUPPORTED_ALTER_ONLINE_ON_VIRTUAL_COLUMN)

Message: ADD COLUMN col...VIRTUAL, ADD INDEX(col)

ER_UNSUPPORTED_ALTER_ONLINE_ON_VIRTUAL_COLUMN was added in 5.7.11.

- Error: 3179 SQLSTATE: HY000 (ER_MASTER_KEY_ROTATION_NOT_SUPPORTED_BY_SE)

Message: Master key rotation is not supported by storage engine.

ER_MASTER_KEY_ROTATION_NOT_SUPPORTED_BY_SE was added in 5.7.11.

- Error: 3180 SQLSTATE: HY000 (ER_MASTER_KEY_ROTATION_ERROR_BY_SE)

Message: Encryption key rotation error reported by SE: %s

ER_MASTER_KEY_ROTATION_ERROR_BY_SE was added in 5.7.11.

- Error: 3181 SQLSTATE: HY000 (ER_MASTER_KEY_ROTATION_BINLOG_FAILED)

Message: Write to binlog failed. However, master key rotation has been completed successfully.

ER_MASTER_KEY_ROTATION_BINLOG_FAILED was added in 5.7.11.

- Error: 3182 SQLSTATE: HY000 (ER_MASTER_KEY_ROTATION_SE_UNAVAILABLE)

Message: Storage engine is not available.

ER_MASTER_KEY_ROTATION_SE_UNAVAILABLE was added in 5.7.11.

- Error: 3183 SQLSTATE: HY000 (ER_TABLESPACE_CANNOT_ENCRYPT)

Message: This tablespace can't be encrypted.

ER_TABLESPACE_CANNOT_ENCRYPT was added in 5.7.11.

- Error: 3184 SQLSTATE: HY000 (ER_INVALID_ENCRYPTION_OPTION)

Message: Invalid encryption option.

ER_INVALID_ENCRYPTION_OPTION was added in 5.7.11.

- Error: 3185 SQLSTATE: HY000 (ER_CANNOT_FIND_KEY_IN_KEYRING)

Message: Can't find master key from keyring, please check in the server log if a keyring plugin is loaded and initialized successfully.

ER_CANNOT_FIND_KEY_IN_KEYRING was added in 5.7.11.

- Error: 3186 SQLSTATE: HY000 (ER_CAPACITY_EXCEEDED_IN_PARSER)

Message: Parser bailed out for this query.

ER_CAPACITY_EXCEEDED_IN_PARSER was added in 5.7.12.

- Error: 3187 SQLSTATE: HY000 (ER_UNSUPPORTED_ALTER_ENCRYPTION_INPLACE)

Message: Cannot alter encryption attribute by inplace algorithm.

ER_UNSUPPORTED_ALTER_ENCRYPTION_INPLACE was added in 5.7.13.

- Error: 3188 SQLSTATE: HY000 (ER_KEYRING_UDF_KEYRING_SERVICE_ERROR)

Message: Function '%s' failed because underlying keyring service returned an error. Please check if a keyring plugin is installed and that provided arguments are valid for the keyring you are using.

ER_KEYRING_UDF_KEYRING_SERVICE_ERROR was added in 5.7.13.

- Error: 3189 SQLSTATE: HY000 (ER_USER_COLUMN_OLD_LENGTH)

Message: It seems that your db schema is old. The %s column is 77 characters long and should be 93 characters long. Please run mysql_upgrade.

ER_USER_COLUMN_OLD_LENGTH was added in 5.7.13.

- Error: 3190 SQLSTATE: HY000 (ER_CANT_RESET_MASTER)

Message: RESET MASTER is not allowed because %s.

ER_CANT_RESET_MASTER was added in 5.7.14.

- Error: 3191 SQLSTATE: HY000 (ER_GROUP_REPLICATION_MAX_GROUP_SIZE)

Message: The START GROUP_REPLICATION command failed since the group already has 9 members.

ER_GROUP_REPLICATION_MAX_GROUP_SIZE was added in 5.7.14.

- Error: 3192 SQLSTATE: HY000 (ER_CANNOT_ADD_FOREIGN_BASE_COL_STORED)

Message: Cannot add foreign key on the base column of stored column.

ER_CANNOT_ADD_FOREIGN_BASE_COL_STORED was added in 5.7.14.

- Error: 3193 SQLSTATE: HY000 (ER_TABLE_REFERENCED)

Message: Cannot complete the operation because table is referenced by another connection.

ER_TABLE_REFERENCED was added in 5.7.14.

- Error: 3194 SQLSTATE: HY000 (ER_PARTITION_ENGINE_DEPRECATED_FOR_TABLE)

Message: The partition engine, used by table '%s.%s', is deprecated and will be removed in a future release. Please use native partitioning instead.

ER_PARTITION_ENGINE_DEPRECATED_FOR_TABLE was added in 5.7.17.

- Error: 3195 SQLSTATE: 01000 (ER_WARN_USING_GEOMFROMWKB_TO_SET_SRID_ZERO)

Message: %s(geometry) is deprecated and will be replaced by st_srid(geometry, 0) in a future version. Use %s(st_aswkb(geometry), 0) instead.

ER_WARN_USING_GEOMFROMWKB_TO_SET_SRID_ZERO was added in 5.7.19.

- Error: 3196 SQLSTATE: 01000 (ER_WARN_USING_GEOMFROMWKB_TO_SET_SRID)

Message: %s(geometry, srid) is deprecated and will be replaced by st_srid(geometry, srid) in a future version. Use %s(st_aswkb(geometry), srid) instead.

ER_WARN_USING_GEOMFROMWKB_TO_SET_SRID was added in 5.7.19.

- Error: 3197 SQLSTATE: HY000 (ER_XA_RETRY)

Message: The resource manager is not able to commit the transaction branch at this time. Please retry later.

ER_XA_RETRY was added in 5.7.19.

- Error: 3198 SQLSTATE: HY000 (ER_KEYRING_AWS_UDF_AWS_KMS_ERROR)

Message: Function %s failed due to: %s.

ER_KEYRING_AWS_UDF_AWS_KMS_ERROR was added in 5.7.19.

- Error: 3199 SQLSTATE: HY000 (ER_BINLOG_UNSAFE_XA)

Message: Statement is unsafe because it is being used inside a XA transaction. Concurrent XA transactions may deadlock on slaves when replicated using statements.

ER_BINLOG_UNSAFE_XA was added in 5.7.20.

- Error: 3200 SQLSTATE: HY000 (ER_UDF_ERROR)

Message: %s UDF failed; %s

ER_UDF_ERROR was added in 5.7.21.

- Error: 3201 SQLSTATE: HY000 (ER_KEYRING_MIGRATION_FAILURE)

Message: Can not perform keyring migration : %s

ER_KEYRING_MIGRATION_FAILURE was added in 5.7.21.

- Error: 3202 SQLSTATE: 42000 (ER_KEYRING_ACCESS_DENIED_ERROR)

Message: Access denied; you need %s privileges for this operation

ER_KEYRING_ACCESS_DENIED_ERROR was added in 5.7.21.

- Error: 3203 SQLSTATE: HY000 (ER_KEYRING_MIGRATION_STATUS)

Message: Keyring migration %s.

ER_KEYRING_MIGRATION_STATUS was added in 5.7.21.

- Error: 3204 SQLSTATE: HY000 (ER_PLUGIN_FAILED_TO_OPEN_TABLES)

Message: Failed to open the %s filter tables.

ER_PLUGIN_FAILED_TO_OPEN_TABLES was added in 5.7.22.

- Error: 3205 SQLSTATE: HY000 (ER_PLUGIN_FAILED_TO_OPEN_TABLE)

Message: Failed to open '%s.%s' %s table.

ER_PLUGIN_FAILED_TO_OPEN_TABLE was added in 5.7.22.

- Error: 3206 SQLSTATE: HY000 (ER_AUDIT_LOG_NO_KEYRING_PLUGIN_INSTALLED)

Message: No keyring plugin installed.

ER_AUDIT_LOG_NO_KEYRING_PLUGIN_INSTALLED was added in 5.7.22.

- Error: 3207 SQLSTATE: HY000 (ER_AUDIT_LOG_ENCRYPTION_PASSWORD_HAS_NOT_BEEN_SET)

Message: Audit log encryption password has not been set; it will be generated automatically. Use audit_log_encryption_password_get to obtain the password or audit_log_encryption_password_set to set a new one.

ER_AUDIT_LOG_ENCRYPTION_PASSWORD_HAS_NOT_BEEN_SET was added in 5.7.22.

- Error: 3208 SQLSTATE: HY000 (ER_AUDIT_LOG_COULD_NOT_CREATE_AES_KEY)

Message: Could not create AES key. OpenSSL's EVP_BytesToKey function failed.

ER_AUDIT_LOG_COULD_NOT_CREATE_AES_KEY was added in 5.7.22.

- Error: 3209 SQLSTATE: HY000 (ER_AUDIT_LOG_ENCRYPTION_PASSWORD_CANNOT_BE_FETCHED)

Message: Audit log encryption password cannot be fetched from the keyring. Password used so far is used for encryption.

ER_AUDIT_LOG_ENCRYPTION_PASSWORD_CANNOT_BE_FETCHED was added in 5.7.22.

- Error: 3210 SQLSTATE: HY000 (ER_AUDIT_LOG_JSON_FILTERING_NOT_ENABLED)

Message: Audit Log filtering has not been installed.

ER_AUDIT_LOG_JSON_FILTERING_NOT_ENABLED was added in 5.7.22.

- Error: 3211 SQLSTATE: HY000 (ER_AUDIT_LOG_UDF_INSUFFICIENT_PRIVILEGE)

Message: Request ignored for '%s'@'%s'. SUPER_ACL needed to perform operation

ER_AUDIT_LOG_UDF_INSUFFICIENT_PRIVILEGE was added in 5.7.22.

- Error: 3212 SQLSTATE: HY000 (ER_AUDIT_LOG_SUPER_PRIVILEGE_REQUIRED)

Message: SUPER privilege required for '%s'@'%s' user.

ER_AUDIT_LOG_SUPER_PRIVILEGE_REQUIRED was added in 5.7.22.

- Error: 3213 SQLSTATE: HY000 (ER_COULD_NOT_REINITIALIZE_AUDIT_LOG_FILTERS)

Message: Could not reinitialize audit log filters.

ER_COULD_NOT_REINITIALIZE_AUDIT_LOG_FILTERS was added in 5.7.22.

- Error: 3214 SQLSTATE: HY000 (ER_AUDIT_LOG_UDF_INVALID_ARGUMENT_TYPE)

Message: Invalid argument type

ER_AUDIT_LOG_UDF_INVALID_ARGUMENT_TYPE was added in 5.7.22.

- Error: 3215 SQLSTATE: HY000 (ER_AUDIT_LOG_UDF_INVALID_ARGUMENT_COUNT)

Message: Invalid argument count

ER_AUDIT_LOG_UDF_INVALID_ARGUMENT_COUNT was added in 5.7.22.

- Error: 3216 SQLSTATE: HY000 (ER_AUDIT_LOG_HAS_NOT_BEEN_INSTALLED)

Message: audit_log plugin has not been installed using INSTALL PLUGIN syntax.

ER_AUDIT_LOG_HAS_NOT_BEEN_INSTALLED was added in 5.7.22.

- Error: 3217 SQLSTATE: HY000 (ER_AUDIT_LOG_UDF_READ_INVALID_MAX_ARRAY_LENGTH_ARG_TYPE)

Message: Invalid "max_array_length" argument type.

ER_AUDIT_LOG_UDF_READ_INVALID_MAX_ARRAY_LENGTH_ARG_TYPE was added in 5.7.22.

- Error: 3218 SQLSTATE: HY000 (ER_AUDIT_LOG_UDF_READ_INVALID_MAX_ARRAY_LENGTH_ARG_VALUE)

Message: Invalid "max_array_length" argument value.

ER_AUDIT_LOG_UDF_READ_INVALID_MAX_ARRAY_LENGTH_ARG_VALUE was added in 5.7.22.

- Error: 3219 SQLSTATE: HY000 (ER_AUDIT_LOG_JSON_FILTER_PARSING_ERROR)

Message: %s

ER_AUDIT_LOG_JSON_FILTER_PARSING_ERROR was added in 5.7.22.

- Error: 3220 SQLSTATE: HY000 (ER_AUDIT_LOG_JSON_FILTER_NAME_CANNOT_BE_EMPTY)

Message: Filter name cannot be empty.

ER_AUDIT_LOG_JSON_FILTER_NAME_CANNOT_BE_EMPTY was added in 5.7.22.

- Error: 3221 SQLSTATE: HY000 (ER_AUDIT_LOG_JSON_USER_NAME_CANNOT_BE_EMPTY)

Message: User cannot be empty.

ER_AUDIT_LOG_JSON_USER_NAME_CANNOT_BE_EMPTY was added in 5.7.22.

- Error: 3222 SQLSTATE: HY000 (ER_AUDIT_LOG_JSON_FILTER_DOES_NOT_EXISTS)

Message: Specified filter has not been found.

ER_AUDIT_LOG_JSON_FILTER_DOES_NOT_EXISTS was added in 5.7.22.

- Error: 3223 SQLSTATE: HY000 (ER_AUDIT_LOG_USER_FIRST_CHARACTER_MUST_BE_ALPHANUMERIC)

Message: First character of the user name must be alphanumeric.

ER_AUDIT_LOG_USER_FIRST_CHARACTER_MUST_BE_ALPHANUMERIC was added in 5.7.22.

- Error: 3224 SQLSTATE: HY000 (ER_AUDIT_LOG_USER_NAME_INVALID_CHARACTER)

Message: Invalid character in the user name.

ER_AUDIT_LOG_USER_NAME_INVALID_CHARACTER was added in 5.7.22.

- Error: 3225 SQLSTATE: HY000 (ER_AUDIT_LOG_HOST_NAME_INVALID_CHARACTER)

Message: Invalid character in the host name.

ER_AUDIT_LOG_HOST_NAME_INVALID_CHARACTER was added in 5.7.22.

- Error: 3226 SQLSTATE: HY000 (WARN_DEPRECATED_MAXDB_SQL_MODE_FOR_TIMESTAMP)

Message: With the MAXDB SQL mode enabled, TIMESTAMP is identical with DATETIME. The MAXDB SQL mode is deprecated and will be removed in a future release. Please disable the MAXDB SQL mode and use DATETIME instead.

WARN_DEPRECATED_MAXDB_SQL_MODE_FOR_TIMESTAMP was added in 5.7.22.

- Error: 3227 SQLSTATE: HY000 (ER_XA_REPLICATION_FILTERS)

Message: The use of replication filters with XA transactions is not supported, and can lead to an undefined state in the replication slave.

ER_XA_REPLICATION_FILTERS was added in 5.7.23.

- Error: 3228 SQLSTATE: HY000 (ER_CANT_OPEN_ERROR_LOG)

Message: Could not open file '%s' for error logging%s%s

ER_CANT_OPEN_ERROR_LOG was added in 5.7.24.

B.4 Client Error Codes and Messages
Client error information comes from the following source files:

The Error values and the symbols in parentheses correspond to definitions in the include/errmsg.h MySQL source file.

The Message values correspond to the error messages that are listed in the libmysql/errmsg.c file. %d and %s represent numbers and strings, respectively, that are substituted into the messages when they are displayed.

Because updates are frequent, it is possible that those files will contain additional error information not listed here.

- Error: 2000 (CR_UNKNOWN_ERROR)

Message: Unknown MySQL error

- Error: 2001 (CR_SOCKET_CREATE_ERROR)

Message: Can't create UNIX socket (%d)

- Error: 2002 (CR_CONNECTION_ERROR)

Message: Can't connect to local MySQL server through socket '%s' (%d)

- Error: 2003 (CR_CONN_HOST_ERROR)

Message: Can't connect to MySQL server on '%s' (%d)

- Error: 2004 (CR_IPSOCK_ERROR)

Message: Can't create TCP/IP socket (%d)

- Error: 2005 (CR_UNKNOWN_HOST)

Message: Unknown MySQL server host '%s' (%d)

- Error: 2006 (CR_SERVER_GONE_ERROR)

Message: MySQL server has gone away

- Error: 2007 (CR_VERSION_ERROR)

Message: Protocol mismatch; server version = %d, client version = %d

- Error: 2008 (CR_OUT_OF_MEMORY)

Message: MySQL client ran out of memory

- Error: 2009 (CR_WRONG_HOST_INFO)

Message: Wrong host info

- Error: 2010 (CR_LOCALHOST_CONNECTION)

Message: Localhost via UNIX socket

- Error: 2011 (CR_TCP_CONNECTION)

Message: %s via TCP/IP

- Error: 2012 (CR_SERVER_HANDSHAKE_ERR)

Message: Error in server handshake

- Error: 2013 (CR_SERVER_LOST)

Message: Lost connection to MySQL server during query

- Error: 2014 (CR_COMMANDS_OUT_OF_SYNC)

Message: Commands out of sync; you can't run this command now

- Error: 2015 (CR_NAMEDPIPE_CONNECTION)

Message: Named pipe: %s

- Error: 2016 (CR_NAMEDPIPEWAIT_ERROR)

Message: Can't wait for named pipe to host: %s pipe: %s (%lu)

- Error: 2017 (CR_NAMEDPIPEOPEN_ERROR)

Message: Can't open named pipe to host: %s pipe: %s (%lu)

- Error: 2018 (CR_NAMEDPIPESETSTATE_ERROR)

Message: Can't set state of named pipe to host: %s pipe: %s (%lu)

- Error: 2019 (CR_CANT_READ_CHARSET)

Message: Can't initialize character set %s (path: %s)

- Error: 2020 (CR_NET_PACKET_TOO_LARGE)

Message: Got packet bigger than 'max_allowed_packet' bytes

- Error: 2021 (CR_EMBEDDED_CONNECTION)

Message: Embedded server

- Error: 2022 (CR_PROBE_SLAVE_STATUS)

Message: Error on SHOW SLAVE STATUS:

- Error: 2023 (CR_PROBE_SLAVE_HOSTS)

Message: Error on SHOW SLAVE HOSTS:

- Error: 2024 (CR_PROBE_SLAVE_CONNECT)

Message: Error connecting to slave:

- Error: 2025 (CR_PROBE_MASTER_CONNECT)

Message: Error connecting to master:

- Error: 2026 (CR_SSL_CONNECTION_ERROR)

Message: SSL connection - Error: %s

- Error: 2027 (CR_MALFORMED_PACKET)

Message: Malformed packet

- Error: 2028 (CR_WRONG_LICENSE)

Message: This client library is licensed only for use with MySQL servers having '%s' license

- Error: 2029 (CR_NULL_POINTER)

Message: Invalid use of null pointer

- Error: 2030 (CR_NO_PREPARE_STMT)

Message: Statement not prepared

- Error: 2031 (CR_PARAMS_NOT_BOUND)

Message: No data supplied for parameters in prepared statement

- Error: 2032 (CR_DATA_TRUNCATED)

Message: Data truncated

- Error: 2033 (CR_NO_PARAMETERS_EXISTS)

Message: No parameters exist in the statement

- Error: 2034 (CR_INVALID_PARAMETER_NO)

Message: Invalid parameter number

The column number for mysql_stmt_fetch_column() was invalid.

The parameter number for mysql_stmt_send_long_data() was invalid.

A key name was empty or the amount of connection attribute data for mysql_options4() exceeds the 64KB limit.

- Error: 2035 (CR_INVALID_BUFFER_USE)

Message: Can't send long data for non-string/non-binary data types (parameter: %d)

- Error: 2036 (CR_UNSUPPORTED_PARAM_TYPE)

Message: Using unsupported buffer type: %d (parameter: %d)

- Error: 2037 (CR_SHARED_MEMORY_CONNECTION)

Message: Shared memory: %s

- Error: 2038 (CR_SHARED_MEMORY_CONNECT_REQUEST_ERROR)

Message: Can't open shared memory; client could not create request event (%lu)

- Error: 2039 (CR_SHARED_MEMORY_CONNECT_ANSWER_ERROR)

Message: Can't open shared memory; no answer event received from server (%lu)

- Error: 2040 (CR_SHARED_MEMORY_CONNECT_FILE_MAP_ERROR)

Message: Can't open shared memory; server could not allocate file mapping (%lu)

- Error: 2041 (CR_SHARED_MEMORY_CONNECT_MAP_ERROR)

Message: Can't open shared memory; server could not get pointer to file mapping (%lu)

- Error: 2042 (CR_SHARED_MEMORY_FILE_MAP_ERROR)

Message: Can't open shared memory; client could not allocate file mapping (%lu)

- Error: 2043 (CR_SHARED_MEMORY_MAP_ERROR)

Message: Can't open shared memory; client could not get pointer to file mapping (%lu)

- Error: 2044 (CR_SHARED_MEMORY_EVENT_ERROR)

Message: Can't open shared memory; client could not create %s event (%lu)

- Error: 2045 (CR_SHARED_MEMORY_CONNECT_ABANDONED_ERROR)

Message: Can't open shared memory; no answer from server (%lu)

- Error: 2046 (CR_SHARED_MEMORY_CONNECT_SET_ERROR)

Message: Can't open shared memory; cannot send request event to server (%lu)

- Error: 2047 (CR_CONN_UNKNOW_PROTOCOL)

Message: Wrong or unknown protocol

- Error: 2048 (CR_INVALID_CONN_HANDLE)

Message: Invalid connection handle

- Error: 2049 (CR_SECURE_AUTH)

Message: Connection using old (pre-4.1.1) authentication protocol refused (client option 'secure_auth' enabled)

CR_SECURE_AUTH was removed after 5.7.4.

- Error: 2049 (CR_UNUSED_1)

Message: Connection using old (pre-4.1.1) authentication protocol refused (client option 'secure_auth' enabled)

CR_UNUSED_1 was added in 5.7.5.

- Error: 2050 (CR_FETCH_CANCELED)

Message: Row retrieval was canceled by mysql_stmt_close() call

- Error: 2051 (CR_NO_DATA)

Message: Attempt to read column without prior row fetch

- Error: 2052 (CR_NO_STMT_METADATA)

Message: Prepared statement contains no metadata

- Error: 2053 (CR_NO_RESULT_SET)

Message: Attempt to read a row while there is no result set associated with the statement

- Error: 2054 (CR_NOT_IMPLEMENTED)

Message: This feature is not implemented yet

- Error: 2055 (CR_SERVER_LOST_EXTENDED)

Message: Lost connection to MySQL server at '%s', system - Error: %d

- Error: 2056 (CR_STMT_CLOSED)

Message: Statement closed indirectly because of a preceding %s() call

- Error: 2057 (CR_NEW_STMT_METADATA)

Message: The number of columns in the result set differs from the number of bound buffers. You must reset the statement, rebind the result set columns, and execute the statement again

- Error: 2058 (CR_ALREADY_CONNECTED)

Message: This handle is already connected. Use a separate handle for each connection.

- Error: 2059 (CR_AUTH_PLUGIN_CANNOT_LOAD)

Message: Authentication plugin '%s' cannot be loaded: %s

- Error: 2060 (CR_DUPLICATE_CONNECTION_ATTR)

Message: There is an attribute with the same name already

A duplicate connection attribute name was specified for mysql_options4().

- Error: 2061 (CR_AUTH_PLUGIN_ERR)

Message: Authentication plugin '%s' reported - Error: %s

CR_AUTH_PLUGIN_ERR was added in 5.7.1.

- Error: 2062 (CR_INSECURE_API_ERR)

Message: Insecure API function call: '%s' Use instead: '%s'

An insecure function call was detected. Modify the application to use the suggested alternative function instead.

CR_INSECURE_API_ERR was added in 5.7.6.

B.5 Problems and Common Errors
B.5.1 How to Determine What Is Causing a Problem
B.5.2 Common Errors When Using MySQL Programs
B.5.3 Administration-Related Issues
B.5.4 Query-Related Issues
B.5.5 Optimizer-Related Issues
B.5.6 Table Definition-Related Issues
B.5.7 Known Issues in MySQL
This section lists some common problems and error messages that you may encounter. It describes how to determine the causes of the problems and what to do to solve them.

B.5.1 How to Determine What Is Causing a Problem
When you run into a problem, the first thing you should do is to find out which program or piece of equipment is causing it:

If you have one of the following symptoms, then it is probably a hardware problems (such as memory, motherboard, CPU, or hard disk) or kernel problem:

The keyboard does not work. This can normally be checked by pressing the Caps Lock key. If the Caps Lock light does not change, you have to replace your keyboard. (Before doing this, you should try to restart your computer and check all cables to the keyboard.)

The mouse pointer does not move.

The machine does not answer to a remote machine's pings.

Other programs that are not related to MySQL do not behave correctly.

Your system restarted unexpectedly. (A faulty user-level program should never be able to take down your system.)

In this case, you should start by checking all your cables and run some diagnostic tool to check your hardware! You should also check whether there are any patches, updates, or service packs for your operating system that could likely solve your problem. Check also that all your libraries (such as glibc) are up to date.

It is always good to use a machine with ECC memory to discover memory problems early.

If your keyboard is locked up, you may be able to recover by logging in to your machine from another machine and executing kbd_mode -a.

Please examine your system log file (/var/log/messages or similar) for reasons for your problem. If you think the problem is in MySQL, you should also examine MySQL's log files. See Section 5.4, “MySQL Server Logs”.

If you do not think you have hardware problems, you should try to find out which program is causing problems. Try using top, ps, Task Manager, or some similar program, to check which program is taking all CPU or is locking the machine.

Use top, df, or a similar program to check whether you are out of memory, disk space, file descriptors, or some other critical resource.

If the problem is some runaway process, you can always try to kill it. If it does not want to die, there is probably a bug in the operating system.

If after you have examined all other possibilities and you have concluded that the MySQL server or a MySQL client is causing the problem, it is time to create a bug report for our mailing list or our support team. In the bug report, try to give a very detailed description of how the system is behaving and what you think is happening. You should also state why you think that MySQL is causing the problem. Take into consideration all the situations in this chapter. State any problems exactly how they appear when you examine your system. Use the “copy and paste” method for any output and error messages from programs and log files.

Try to describe in detail which program is not working and all symptoms you see. We have in the past received many bug reports that state only “the system does not work.” This provides us with no information about what could be the problem.

If a program fails, it is always useful to know the following information:

Has the program in question made a segmentation fault (did it dump core)?

Is the program taking up all available CPU time? Check with top. Let the program run for a while, it may simply be evaluating something computationally intensive.

If the mysqld server is causing problems, can you get any response from it with mysqladmin -u root ping or mysqladmin -u root processlist?

What does a client program say when you try to connect to the MySQL server? (Try with mysql, for example.) Does the client jam? Do you get any output from the program?

When sending a bug report, you should follow the outline described in Section 1.7, “How to Report Bugs or Problems”.

B.5.2 Common Errors When Using MySQL Programs
B.5.2.1 Access denied
B.5.2.2 Can't connect to [local] MySQL server
B.5.2.3 Lost connection to MySQL server
B.5.2.4 Password Fails When Entered Interactively
B.5.2.5 Host 'host_name' is blocked
B.5.2.6 Too many connections
B.5.2.7 Out of memory
B.5.2.8 MySQL server has gone away
B.5.2.9 Packet Too Large
B.5.2.10 Communication Errors and Aborted Connections
B.5.2.11 The table is full
B.5.2.12 Can't create/write to file
B.5.2.13 Commands out of sync
B.5.2.14 Ignoring user
B.5.2.15 Table 'tbl_name' doesn't exist
B.5.2.16 Can't initialize character set
B.5.2.17 File Not Found and Similar Errors
B.5.2.18 Table-Corruption Issues
This section lists some errors that users frequently encounter when running MySQL programs. Although the problems show up when you try to run client programs, the solutions to many of the problems involves changing the configuration of the MySQL server.

B.5.2.1 Access denied
An Access denied error can have many causes. Often the problem is related to the MySQL accounts that the server permits client programs to use when connecting. See Section 6.2, “The MySQL Access Privilege System”, and Section 6.2.7, “Troubleshooting Problems Connecting to MySQL”.

B.5.2.2 Can't connect to [local] MySQL server
A MySQL client on Unix can connect to the mysqld server in two different ways: By using a Unix socket file to connect through a file in the file system (default /tmp/mysql.sock), or by using TCP/IP, which connects through a port number. A Unix socket file connection is faster than TCP/IP, but can be used only when connecting to a server on the same computer. A Unix socket file is used if you do not specify a host name or if you specify the special host name localhost.

If the MySQL server is running on Windows, you can connect using TCP/IP. If the server is started with the --enable-named-pipe option, you can also connect with named pipes if you run the client on the host where the server is running. The name of the named pipe is MySQL by default. If you do not give a host name when connecting to mysqld, a MySQL client first tries to connect to the named pipe. If that does not work, it connects to the TCP/IP port. You can force the use of named pipes on Windows by using . as the host name.

The error (2002) Can't connect to ... normally means that there is no MySQL server running on the system or that you are using an incorrect Unix socket file name or TCP/IP port number when trying to connect to the server. You should also check that the TCP/IP port you are using has not been blocked by a firewall or port blocking service.

The error (2003) Can't connect to MySQL server on 'server' (10061) indicates that the network connection has been refused. You should check that there is a MySQL server running, that it has network connections enabled, and that the network port you specified is the one configured on the server.

Start by checking whether there is a process named mysqld running on your server host. (Use ps xa | grep mysqld on Unix or the Task Manager on Windows.) If there is no such process, you should start the server. See Section 2.10.2, “Starting the Server”.

If a mysqld process is running, you can check it by trying the following commands. The port number or Unix socket file name might be different in your setup. host_ip represents the IP address of the machine where the server is running.

shell> mysqladmin version
shell> mysqladmin variables
shell> mysqladmin -h `hostname` version variables
shell> mysqladmin -h `hostname` --port=3306 version
shell> mysqladmin -h host_ip version
shell> mysqladmin --protocol=SOCKET --socket=/tmp/mysql.sock version
Note the use of backticks rather than forward quotation marks with the hostname command; these cause the output of hostname (that is, the current host name) to be substituted into the mysqladmin command. If you have no hostname command or are running on Windows, you can manually type the host name of your machine (without backticks) following the -h option. You can also try -h 127.0.0.1 to connect with TCP/IP to the local host.

Make sure that the server has not been configured to ignore network connections or (if you are attempting to connect remotely) that it has not been configured to listen only locally on its network interfaces. If the server was started with --skip-networking, it will not accept TCP/IP connections at all. If the server was started with --bind-address=127.0.0.1, it will listen for TCP/IP connections only locally on the loopback interface and will not accept remote connections.

Check to make sure that there is no firewall blocking access to MySQL. Your firewall may be configured on the basis of the application being executed, or the port number used by MySQL for communication (3306 by default). Under Linux or Unix, check your IP tables (or similar) configuration to ensure that the port has not been blocked. Under Windows, applications such as ZoneAlarm or Windows Firewall may need to be configured not to block the MySQL port.

Here are some reasons the Can't connect to local MySQL server error might occur:

mysqld is not running on the local host. Check your operating system's process list to ensure the mysqld process is present.

You're running a MySQL server on Windows with many TCP/IP connections to it. If you're experiencing that quite often your clients get that error, you can find a workaround here: Section B.5.2.2.1, “Connection to MySQL Server Failing on Windows”.

Someone has removed the Unix socket file that mysqld uses (/tmp/mysql.sock by default). For example, you might have a cron job that removes old files from the /tmp directory. You can always run mysqladmin version to check whether the Unix socket file that mysqladmin is trying to use really exists. The fix in this case is to change the cron job to not remove mysql.sock or to place the socket file somewhere else. See Section B.5.3.6, “How to Protect or Change the MySQL Unix Socket File”.

You have started the mysqld server with the --socket=/path/to/socket option, but forgotten to tell client programs the new name of the socket file. If you change the socket path name for the server, you must also notify the MySQL clients. You can do this by providing the same --socket option when you run client programs. You also need to ensure that clients have permission to access the mysql.sock file. To find out where the socket file is, you can do:

shell> netstat -ln | grep mysql
See Section B.5.3.6, “How to Protect or Change the MySQL Unix Socket File”.

You are using Linux and one server thread has died (dumped core). In this case, you must kill the other mysqld threads (for example, with kill) before you can restart the MySQL server. See Section B.5.3.3, “What to Do If MySQL Keeps Crashing”.

The server or client program might not have the proper access privileges for the directory that holds the Unix socket file or the socket file itself. In this case, you must either change the access privileges for the directory or socket file so that the server and clients can access them, or restart mysqld with a --socket option that specifies a socket file name in a directory where the server can create it and where client programs can access it.

If you get the error message Can't connect to MySQL server on some_host, you can try the following things to find out what the problem is:

Check whether the server is running on that host by executing telnet some_host 3306 and pressing the Enter key a couple of times. (3306 is the default MySQL port number. Change the value if your server is listening to a different port.) If there is a MySQL server running and listening to the port, you should get a response that includes the server's version number. If you get an error such as telnet: Unable to connect to remote host: Connection refused, then there is no server running on the given port.

If the server is running on the local host, try using mysqladmin -h localhost variables to connect using the Unix socket file. Verify the TCP/IP port number that the server is configured to listen to (it is the value of the port variable.)

If you are running under Linux and Security-Enhanced Linux (SELinux) is enabled, make sure you have disabled SELinux protection for the mysqld process.
