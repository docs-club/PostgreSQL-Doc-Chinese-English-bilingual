# initdb

initdb — create a new PostgreSQL database cluster

创建新的 PostgreSQL 数据库集群

## Synopsis

initdb [option...] [ --pgdata | -D ] directory

## Description

initdb creates a new PostgreSQL database cluster. A database cluster is a collection of databases that are managed by a single server instance.

initdb 创建一个新的 PostgreSQL 数据库集群。数据库集群是由单个服务器实例管理的数据库的集合。

Creating a database cluster consists of creating the directories in which the database data will live, generating the shared catalog tables (tables that belong to the whole cluster rather than to any particular database), and creating the `postgres`, `template1`, and `template0` databases. The `postgres` database is a default database meant for use by users, utilities and third party applications. `template1` and `template0` are meant as source databases to be copied by later `CREATE DATABASE` commands. `template0` should never be modified, but you can add objects to `template1`, which by default will be copied into databases created later. See Section 23.3 for more details.

创建数据库集群包括创建数据库数据所在的目录、生成共享目录表（属于整个集群而不是任何特定数据库的表）以及创建 `postgres` 、`template1` 和 `template0` 数据库。`postgres` 数据库是供用户、实用程序和第三方应用程序使用的默认数据库。`template1` 和 `template0` 是指由后续的 `CREATE DATABASE` 命令复制的源数据库。永远不要修改 `template0` ，但可以将对象添加到 `template1`，默认情况下该对象将被复制到稍后创建的数据库中。更多细节请参见第 23.3 节。

Although `initdb` will attempt to create the specified data directory, it might not have permission if the parent directory of the desired data directory is root-owned. To initialize in such a setup, create an empty data directory as root, then use chown to assign ownership of that directory to the database user account, then `su` to become the database user to run `initdb`.

尽管 `initdb` 将尝试创建指定的数据目录，但如果所需数据目录的父目录为 root 所有，则它可能没有权限。要在此类设置中进行初始化，请以 root 身份创建一个空数据目录，然后使用 chown 将该目录的所有权分配给数据库用户帐户，然后使用 `su` 成为运行 `initdb` 的数据库用户。

`initdb` must be run as the user that will own the server process, because the server needs to have access to the files and directories that `initdb` creates. Since the server cannot be run as root, you must not run `initdb` as root either. (It will in fact refuse to do so.)

`initdb` 必须以拥有服务器进程的用户身份运行，因为服务器需要访问 `initdb` 创建的文件和目录。由于服务器不能以 root 身份运行，因此也不能以 root 身份运行 `initdb`。 （事实上 ​​ 它会拒绝这样做。）

For security reasons the new cluster created by `initdb` will only be accessible by the cluster owner by default. The `--allow-group-access` option allows any user in the same group as the cluster owner to read files in the cluster. This is useful for performing backups as a non-privileged user.

出于安全原因，默认情况下，由 `initdb` 创建的新集群只能由集群所有者访问。`--allow-group-access` 选项允许与集群所有者位于同一组中的任何用户读取集群中的文件。这对于以非特权用户身份执行备份非常有用。

`initdb` initializes the database cluster's default locale and character set encoding. These can also be set separately for each database when it is created. `initdb` determines those settings for the template databases, which will serve as the default for all other databases. By default, `initdb` uses the locale provider `libc`, takes the locale settings from the environment, and determines the encoding from the locale settings. This is almost always sufficient, unless there are special requirements.

`initdb` 初始化数据库集群的默认区域设置和字符集编码。这些也可以在创建每个数据库时单独设置。`initdb` 确定模板数据库的这些设置，这将作为所有其他数据库的默认设置。默认情况下，`initdb` 使用区域设置提供程序 `libc`，从环境中获取区域设置，并根据区域设置确定编码。除非有特殊要求，否则这几乎总是足够的。

To choose a different locale for the cluster, use the option `--locale`. There are also individual options `--lc-\*` (see below) to set values for the individual locale categories. Note that inconsistent settings for different locale categories can give nonsensical results, so this should be used with care.

要为集群选择不同的区域设置，请使用选项 `--locale`。还有单独的选项 `--lc-\*` （见下文）来设置各个区域设置类别的值。请注意，不同区域设置类别的不一致设置可能会产生无意义的结果，因此应谨慎使用。

Alternatively, the ICU library can be used to provide locale services. (Again, this only sets the default for subsequently created databases.) To select this option, specify `--locale-provider=icu`. To choose the specific ICU locale ID to apply, use the option `--icu-locale`. Note that for implementation reasons and to support legacy code, `initdb` will still select and initialize libc locale settings when the ICU locale provider is used.

或者，ICU 库可用于提供本地服务。 （同样，这仅为后续创建的数据库设置默认值。）要选择此选项，请指定 `--locale-provider=icu`。要选择要应用的特定 ICU 区域设置 ID，请使用选项 `--icu-locale`。请注意，出于实现原因并支持遗留代码，当使用 ICU 语言环境提供程序时， `initdb` 仍将选择并初始化 libc 语言环境设置。

When `initdb` runs, it will print out the locale settings it has chosen. If you have complex requirements or specified multiple options, it is advisable to check that the result matches what was intended.

当 `initdb` 运行时，它将打印出它选择的区域设置。如果有复杂的要求或指定了多个选项，建议检查结果是否与预期相符。

More details about locale settings can be found in Section 24.1.

有关区域设置的更多详细信息可以在第 24.1 节中找到。

To alter the default encoding, use the `--encoding`. More details can be found in Section 24.3.

要更改默认编码，请使用 `--encoding`。更多详细信息请参见第 24.3 节。

## Options

`-A authmethod`

`--auth=authmethod`

This option specifies the default authentication method for local users used in `pg_hba.conf` (host and local lines). See Section 21.1 for an overview of valid values.

此选项指定 `pg_hba.conf`（主机和本地行）中使用的本地用户的默认身份验证方法。有关有效值的概述，请参见第 21.1 节。

`initdb` will prepopulate `pg_hba.conf` entries using the specified authentication method for non-replication as well as replication connections.

`initdb` 将使用指定的身份验证方法为非复制和复制连接预填充 `pg_hba.conf` 条目。

Do not use trust unless you trust all local users on your system. trust is the default for ease of installation.

除非信任系统上的所有本地用户，否则请勿使用信任。 trust 是默认值，以便于安装。

`--auth-host=authmethod`

This option specifies the authentication method for local users via TCP/IP connections used in `pg_hba.conf` (host lines).

此选项指定通过 `pg_hba.conf`（主机）中使用的 TCP/IP 连接对本地用户进行身份验证的方法。

`--auth-local=authmethod`

This option specifies the authentication method for local users via Unix-domain socket connections used in `pg_hba.conf` (local lines).

此选项指定通过 `pg_hba.conf`（本地）中使用的 Unix 域套接字连接对本地用户进行身份验证的方法。

`-D directory`

`--pgdata=directory`

This option specifies the directory where the database cluster should be stored. This is the only information required by `initdb`, but you can avoid writing it by setting the PGDATA environment variable, which can be convenient since the database server (postgres) can find the database directory later by the same variable.

该选项指定数据库集群的存储目录。这是 `initdb` 所需的唯一信息，但可以通过设置 PGDATA 环境变量来避免写入它，这很方便，因为数据库服务器（postgres）稍后可以通过同一变量找到数据库目录。

`-E encoding`

`--encoding=encoding`

Selects the encoding of the template databases. This will also be the default encoding of any database you create later, unless you override it then. The default is derived from the locale, if the libc locale provider is used, or UTF8 if the ICU locale provider is used. The character sets supported by the PostgreSQL server are described in Section 24.3.1.

选择模板数据库的编码。这也将是以后创建的任何数据库的默认编码，除非随后覆盖它。如果使用 libc 语言环境提供程序，则默认值源自语言环境；如果使用 ICU 语言环境提供程序，则默认值源自 UTF8。 PostgreSQL 服务器支持的字符集在第 24.3.1 节中描述。

`-g`

`--allow-group-access`

Allows users in the same group as the cluster owner to read all cluster files created by `initdb`.

允许与集群所有者位于同一组的用户读取 `initdb` 创建的所有集群文件。

This option is ignored on Windows as it does not support POSIX-style group permissions.

此选项在 Windows 上被忽略，因为它不支持 POSIX 样式组权限。

`--icu-locale=locale`

Specifies the ICU locale ID, if the ICU locale provider is used.

如果使用 ICU 区域设置提供程序，则指定 ICU 区域设置 ID。

`-k`

`--data-checksums`

Use checksums on data pages to help detect corruption by the I/O system that would otherwise be silent. Enabling checksums may incur a noticeable performance penalty. If set, checksums are calculated for all objects, in all databases. All checksum failures will be reported in the `pg_stat_database` view. See Section 30.2 for details.

在数据页上使用校验和可以帮助检测 I/O 系统的损坏，否则该系统将保持沉默。启用校验和可能会导致明显的性能损失。如果设置，则会计算所有数据库中所有对象的校验和。所有校验和失败都将在 `pg_stat_database` 视图中报告。详细信息请参见第 30.2 节。

`--locale=locale`

Sets the default locale for the database cluster. If this option is not specified, the locale is inherited from the environment that `initdb` runs in. Locale support is described in Section 24.1.

设置数据库集群的默认区域设置。如果未指定此选项，则区域设置将从 `initdb` 运行的环境继承。区域设置支持在第 24.1 节中描述。

```sh
--lc-collate=locale
--lc-ctype=locale
--lc-messages=locale
--lc-monetary=locale
--lc-numeric=locale
--lc-time=locale
```

Like `--locale`, but only sets the locale in the specified category.

与 `--locale` 类似，但仅设置指定类别中的区域设置。

`--no-locale`

Equivalent to `--locale=C`.

相当于 `--locale=C`。

`--locale-provider={libc|icu}`

This option sets the locale provider for databases created in the new cluster. It can be overridden in the CREATE DATABASE command when new databases are subsequently created. The default is `libc`.

此选项为新集群中创建的数据库设置区域设置提供程序。随后创建新数据库时，可以在 CREATE DATABASE 命令中覆盖它。默认是 `libc`

`-N`

`--no-sync`

By default, `initdb` will wait for all files to be written safely to disk. This option causes `initdb` to return without waiting, which is faster, but means that a subsequent operating system crash can leave the data directory corrupt. Generally, this option is useful for testing, but should not be used when creating a production installation.

默认情况下， `initdb` 将等待所有文件安全写入磁盘。此选项会导致 `initdb` 立即返回而无需等待，这会更快，但意味着随后的操作系统崩溃可能会导致数据目录损坏。通常，此选项对于测试很有用，但在创建生产安装时不应使用。

`--no-instructions`

By default, `initdb` will write instructions for how to start the cluster at the end of its output. This option causes those instructions to be left out. This is primarily intended for use by tools that wrap `initdb` in platform-specific behavior, where those instructions are likely to be incorrect.

默认情况下，`initdb` 将在其输出末尾写入有关如何启动集群的指令。此选项会导致这些指令被忽略。这主要供以特定于平台的行为包装 `initdb` 的工具使用，其中这些指令可能不正确。

`--pwfile=filename`

Makes `initdb` read the database superuser's password from a file. The first line of the file is taken as the password.

使 `initdb` 从文件中读取数据库超级用户的密码。文件的第一行作为密码。

`-S`

`--sync-only`

Safely write all database files to disk and exit. This does not perform any of the normal `initdb` operations. Generally, this option is useful for ensuring reliable recovery after changing fsync from off to on.

将所有数据库文件安全地写入磁盘并退出。这不会执行任何正常的 `initdb` 操作。一般来说，此选项对于确保 fsync 从关闭更改为打开后的可靠恢复很有用。

`-T config`

`--text-search-config=config`

Sets the default text search configuration. See default_text_search_config for further information.

设置默认文本搜索配置。有关详细信息，请参阅 default_text_search_config。

`-U username`

`--username=username`

Selects the user name of the database superuser. This defaults to the name of the effective user running `initdb`. It is really not important what the superuser's name is, but one might choose to keep the customary name postgres, even if the operating system user's name is different.

选择数据库超级用户的用户名。默认为运行 `initdb` 的有效用户的名称。超级用户的名称实际上并不重要，但人们可能会选择保留惯用名称 postgres，即使操作系统用户的名称不同。

`-W`

`--pwprompt`

Makes `initdb` prompt for a password to give the database superuser. If you don't plan on using password authentication, this is not important. Otherwise you won't be able to use password authentication until you have a password set up.

使 `initdb` 提示输入密码以提供给数据库超级用户。如果不打算使用密码身份验证，那么这并不重要。否则，在设置密码之前，将无法使用密码身份验证。

`-X directory`

`--waldir=directory`

This option specifies the directory where the write-ahead log should be stored.

该选项指定预写日志的存储目录。

`--wal-segsize=size`

Set the WAL segment size, in megabytes. This is the size of each individual file in the WAL log. The default size is 16 megabytes. The value must be a power of 2 between 1 and 1024 (megabytes). This option can only be set during initialization, and cannot be changed later.

设置 WAL 段大小（以兆字节为单位）。这是 WAL 日志中每个单独文件的大小。默认大小为 16 兆字节。该值必须是 1 到 1024（兆字节）之间的 2 的幂。该选项只能在初始化时设置，以后无法更改。

It may be useful to adjust this size to control the granularity of WAL log shipping or archiving. Also, in databases with a high volume of WAL, the sheer number of WAL files per directory can become a performance and management problem. Increasing the WAL file size will reduce the number of WAL files.

调整此大小以控制 WAL 日志传送或归档的粒度可能很有用。此外，在具有大量 WAL 的数据库中，每个目录的 WAL 文件数量庞大可能会成为性能和管理问题。增加 WAL 文件大小将会减少 WAL 文件的数量。

Other, less commonly used, options are also available:

其他不太常用的选项也可用：

`-d`

`--debug`

Print debugging output from the bootstrap backend and a few other messages of lesser interest for the general public. The bootstrap backend is the program `initdb` uses to create the catalog tables. This option generates a tremendous amount of extremely boring output.

打印引导程序后端的调试输出以及公众不太感兴趣的一些其他消息。引导后端是 `initdb` 程序用来创建目录表的程序。此选项会生成大量极其无聊的输出。

`--discard-caches`

Run the bootstrap backend with the debug_discard_caches=1 option. This takes a very long time and is only of use for deep debugging.

使用 debug_discard_caches=1 选项运行引导后端。这需要很长时间并且仅用于深度调试。

`-L directory`

Specifies where `initdb` should find its input files to initialize the database cluster. This is normally not necessary. You will be told if you need to specify their location explicitly.

指定 `initdb` 应在何处找到其输入文件以初始化数据库集群。这通常是没有必要的。系统会告知是否需要明确指定其位置。

`-n`

`--no-clean`

By default, when `initdb` determines that an error prevented it from completely creating the database cluster, it removes any files it might have created before discovering that it cannot finish the job. This option inhibits tidying-up and is thus useful for debugging.

默认情况下，当 `initdb` 确定错误阻止其完全创建数据库集群时，它会删除在发现无法完成作业之前可能已创建的所有文件。此选项禁止整理，因此对于调试很有用。

Other options:

`-V`

`--version`

Print the `initdb` version and exit.

`-?`

`--help`

Show help about `initdb` command line arguments, and exit.

## Environment

`PGDATA`

Specifies the directory where the database cluster is to be stored; can be overridden using the `-D` option.

指定数据库集群存放的目录；可以使用 `-D` 选项覆盖。

`PG_COLOR`

Specifies whether to use color in diagnostic messages. Possible values are always, auto and never.

指定是否在诊断消息中使用颜色。可能的值有 always、auto 和 never。

`TZ`

Specifies the default time zone of the created database cluster. The value should be a full time zone name (see Section 8.5.3).

指定创建的数据库集群的默认时区。该值应该是完整的时区名称（请参阅第 8.5.3 节）。

This utility, like most other PostgreSQL utilities, also uses the environment variables supported by libpq (see Section 34.15).

与大多数其他 PostgreSQL 实用程序一样，该实用程序也使用 libpq 支持的环境变量（请参阅第 34.15 节）。

## Notes

`initdb` can also be invoked via `pg_ctl initdb`.

`initdb` 也可以通过 `pg_ctl initdb` 调用。

## See Also

pg_ctl, postgres, Section 21.1
