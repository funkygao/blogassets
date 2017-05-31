---
title: DB Schema Management
date: 2017-05-31 11:06:12
tags: database
---

https://github.com/skeema/skeema

统一控制dev/test/staging/prod等环境的scheme

```
$skeema
Skeema is a MySQL schema management tool. It allows you to export a database
schema to the filesystem, and apply online schema changes by modifying files.

Usage:
      skeema [<options>] <command>

Commands:
      add-environment  Add a new named environment to an existing host directory
      diff             Compare a DB instance's schemas and tables to the filesystem
      help             Display usage information
      init             Save a DB instance's schemas and tables to the filesystem
      lint             Verify table files and reformat them in a standardized way
      pull             Update the filesystem representation of schemas and tables
      push             Alter tables on DBs to reflect the filesystem representation
      version          Display program version
```

与[mysql在线alter table设计](http://funkygao.github.io/2017/05/11/osc/)不同，它是higher level的，底层仍旧需要OSC支持: 
```
alter-wrapper="/usr/local/bin/pt-online-schema-change --execute --alter {CLAUSES} D={SCHEMA},t={TABLE},h={HOST},P={PORT},u={USER},p={PASSWORDX}"
```

## References

https://www.percona.com/live/17/sessions/automatic-mysql-schema-management-skeema

