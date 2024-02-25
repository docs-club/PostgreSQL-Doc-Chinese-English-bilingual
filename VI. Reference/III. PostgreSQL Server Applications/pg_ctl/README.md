# pg_ctl

pg_ctl — initialize, start, stop, or control a PostgreSQL server

初始化、启动、停止或控制 PostgreSQL 服务器

## Synopsis

`pg_ctl init[db] [-D datadir] [-s] [-o initdb-options]`

`pg_ctl start [-D datadir] [-l filename] [-W] [-t seconds] [-s] [-o options] [-p path] [-c]`

`pg_ctl stop [-D datadir] [-m s[mart] | f[ast] | i[mmediate] ] [-W] [-t seconds] [-s]`

`pg_ctl restart [-D datadir] [-m s[mart] | f[ast] | i[mmediate] ] [-W] [-t seconds] [-s] [-o options] [-c]`

`pg_ctl reload [-D datadir] [-s]`

`pg_ctl status [-D datadir]`

`pg_ctl promote [-D datadir] [-W] [-t seconds] [-s]`

`pg_ctl logrotate [-D datadir] [-s]`

`pg_ctl kill signal_name process_id`

On Microsoft Windows, also:

`pg_ctl register [-D datadir] [-N servicename] [-U username] [-P password] [-S a[uto] | d[emand] ] [-e source] [-W] [-t seconds] [-s] [-o options]`

`pg_ctl unregister [-N servicename]`

## Description

pg_ctl is a utility for initializing a PostgreSQL database cluster, starting, stopping, or restarting the PostgreSQL database server (postgres), or displaying the status of a running server. Although the server can be started manually, pg_ctl encapsulates tasks such as redirecting log output and properly detaching from the terminal and process group. It also provides convenient options for controlled shutdown.

pg_ctl 是一个实用程序，用于初始化 PostgreSQL 数据库集群，启动、停止或重新启动 PostgreSQL 数据库服务器 (postgres)，或显示正在运行的服务器的状态。虽然服务器可以手动启动，但 pg_ctl 封装了诸如重定向日志输出以及正确地从终端和进程组分离等任务。它还提供了方便的受控关闭选项。

The init or initdb mode creates a new PostgreSQL database cluster, that is, a collection of databases that will be managed by a single server instance. This mode invokes the initdb command. See initdb for details.

init 或 initdb 模式创建一个新的 PostgreSQL 数据库集群，即将由单个服务器实例管理的数据库集合。此模式调用 initdb 命令。有关详细信息，请参阅 initdb。

start mode launches a new server. The server is started in the background, and its standard input is attached to /dev/null (or nul on Windows). On Unix-like systems, by default, the server's standard output and standard error are sent to pg_ctl's standard output (not standard error). The standard output of pg_ctl should then be redirected to a file or piped to another process such as a log rotating program like rotatelogs; otherwise postgres will write its output to the controlling terminal (from the background) and will not leave the shell's process group. On Windows, by default the server's standard output and standard error are sent to the terminal. These default behaviors can be changed by using -l to append the server's output to a log file. Use of either -l or output redirection is recommended.

启动模式启动一个新服务器。服务器在后台启动，其标准输入附加到 /dev/null（或 Windows 上的 nul）。在类 Unix 系统上，默认情况下，服务器的标准输出和标准错误被发送到 pg_ctl 的标准输出（不是标准错误）。然后 pg_ctl 的标准输出应该被重定向到一个文件或通过管道传输到另一个进程，例如像 rotatelogs 这样的日志轮换程序；否则 postgres 会将其输出写入控制终端（从后台）并且不会离开 shell 的进程组。在 Windows 上，默认情况下服务器的标准输出和标准错误将发送到终端。可以通过使用 `-l` 将服务器的输出附加到日志文件来更改这些默认行为。建议使用 `-l` 或输出重定向。

stop mode shuts down the server that is running in the specified data directory. Three different shutdown methods can be selected with the -m option. “Smart” mode disallows new connections, then waits for all existing clients to disconnect. If the server is in hot standby, recovery and streaming replication will be terminated once all clients have disconnected. “Fast” mode (the default) does not wait for clients to disconnect. All active transactions are rolled back and clients are forcibly disconnected, then the server is shut down. “Immediate” mode will abort all server processes immediately, without a clean shutdown. This choice will lead to a crash-recovery cycle during the next server start.

stop 模式关闭在指定数据目录中运行的服务器。使用 -m 选项可以选择三种不同的关闭方法。 「智能」模式不允许新连接，然后等待所有现有客户端断开连接。如果服务器处于热备状态，一旦所有客户端断开连接，恢复和流式复制就会终止。「快速」模式（默认）不等待客户端断开连接。所有活动事务都将回滚，客户端将被强制断开连接，然后服务器将关闭。「立即」模式将立即中止所有服务器进程，而不会完全关闭。此选择将导致下次服务器启动期间出现崩溃恢复周期。

restart mode effectively executes a stop followed by a start. This allows changing the postgres command-line options, or changing configuration-file options that cannot be changed without restarting the server. If relative paths were used on the command line during server start, restart might fail unless pg_ctl is executed in the same current directory as it was during server start.

重新启动模式有效地执行停止然后启动。这允许更改 postgres 命令行选项，或更改在不重新启动服务器的情况下无法更改的配置文件选项。如果在服务器启动期间在命令行上使用相对路径，则重新启动可能会失败，除非在与服务器启动期间相同的当前目录中执行 pg_ctl。

reload mode simply sends the postgres server process a SIGHUP signal, causing it to reread its configuration files (postgresql.conf, pg_hba.conf, etc.). This allows changing configuration-file options that do not require a full server restart to take effect.

重新加载模式只是向 postgres 服务器进程发送 SIGHUP 信号，使其重新读取其配置文件（`postgresql.conf`、`pg_hba.conf` 等）。这允许更改配置文件选项，而不需要完全重新启动服务器即可生效。

status mode checks whether a server is running in the specified data directory. If it is, the server's PID and the command line options that were used to invoke it are displayed. If the server is not running, pg_ctl returns an exit status of 3. If an accessible data directory is not specified, pg_ctl returns an exit status of 4.

状态模式检查服务器是否正在指定的数据目录中运行。如果是，则会显示服务器的 PID 和用于调用它的命令行选项。如果服务器未运行，pg_ctl 返回退出状态 3。如果未指定可访问的数据目录，pg_ctl 返回退出状态 4。

promote mode commands the standby server that is running in the specified data directory to end standby mode and begin read-write operations.

升级模式命令在指定数据目录中运行的备用服务器结束备用模式并开始读写操作。

logrotate mode rotates the server log file. For details on how to use this mode with external log rotation tools, see Section 25.3.

logrotate 模式轮换服务器日志文件。有关如何与外部日志轮换工具一起使用此模式的详细信息，请参阅第 25.3 节。

kill mode sends a signal to a specified process. This is primarily valuable on Microsoft Windows which does not have a built-in kill command. Use --help to see a list of supported signal names.

Kill 模式向指定进程发送信号。这在没有内置终止命令的 Microsoft Windows 上尤其有价值。使用 --help 查看支持的信号名称列表。

register mode registers the PostgreSQL server as a system service on Microsoft Windows. The -S option allows selection of service start type, either “auto” (start service automatically on system startup) or “demand” (start service on demand).

注册模式将 PostgreSQL 服务器注册为 Microsoft Windows 上的系统服务。 -S 选项允许选择服务启动类型，“auto”（系统启动时自动启动服务）或“demand”（按需启动服务）。

unregister mode unregisters a system service on Microsoft Windows. This undoes the effects of the register command.

取消注册模式在 Microsoft Windows 上取消注册系统服务。这会消除寄存器命令的影响。

## Options

`-c`

`--core-files`

Attempt to allow server crashes to produce core files, on platforms where this is possible, by lifting any soft resource limit placed on core files. This is useful in debugging or diagnosing problems by allowing a stack trace to be obtained from a failed server process.

尝试通过解除对核心文件施加的任何软资源限制，在可能的平台上允许服务器崩溃生成核心文件。通过允许从失败的服务器进程获取堆栈跟踪，这在调试或诊断问题时非常有用。

`-D datadir`

`--pgdata=datadir`

Specifies the file system location of the database configuration files. If this option is omitted, the environment variable PGDATA is used.

指定数据库配置文件的文件系统位置。如果省略此选项，则使用环境变量 PGDATA。

`-l filename`

`--log=filename`

Append the server log output to filename. If the file does not exist, it is created. The umask is set to 077, so access to the log file is disallowed to other users by default.

将服务器日志输出附加到文件名。如果该文件不存在，则创建该文件。 umask 设置为 077，因此默认情况下不允许其他用户访问日志文件。

`-m mode`

`--mode=mode`

Specifies the shutdown mode. mode can be smart, fast, or immediate, or the first letter of one of these three. If this option is omitted, fast is the default.

指定关闭模式。 mode 可以是 smart、fast、immediate，或者是这三者之一的第一个字母。如果省略此选项，则默认为 fast

`-o options`

`--options=options`

Specifies options to be passed directly to the postgres command. -o can be specified multiple times, with all the given options being passed through. The options should usually be surrounded by single or double quotes to ensure that they are passed through as a group.

指定直接传递给 postgres 命令的选项。`-o` 可以指定多次，所有给定的选项都会被传递。这些选项通常应该用单引号或双引号引起来，以确保它们作为一个组传递。

`-o initdb-options`

`--options=initdb-options`

Specifies options to be passed directly to the initdb command. -o can be specified multiple times, with all the given options being passed through. The initdb-options should usually be surrounded by single or double quotes to ensure that they are passed through as a group.

指定直接传递给 initdb 命令的选项。`-o` 可以指定多次，所有给定的选项都会被传递。initdb-options 通常应该用单引号或双引号引起来，以确保它们作为一个组传递。

`-p path`

Specifies the location of the postgres executable. By default the postgres executable is taken from the same directory as pg_ctl, or failing that, the hard-wired installation directory. It is not necessary to use this option unless you are doing something unusual and get errors that the postgres executable was not found.

指定 postgres 可执行文件的位置。默认情况下，postgres 可执行文件取自与 pg_ctl 相同的目录，如果失败，则取自硬连线安装目录。没有必要使用此选项，除非您正在执行一些不寻常的操作并收到未找到 postgres 可执行文件的错误。

In init mode, this option analogously specifies the location of the initdb executable.

在 init 模式下，此选项类似地指定 initdb 可执行文件的位置。

`-s`

`--silent`

Print only errors, no informational messages.

仅打印错误，不打印信息性消息。

`-t seconds`

`--timeout=seconds`

Specifies the maximum number of seconds to wait when waiting for an operation to complete (see option -w). Defaults to the value of the PGCTLTIMEOUT environment variable or, if not set, to 60 seconds.

指定等待操作完成时要等待的最大秒数（请参阅选项 -w）。默认为 PGCTLTIMEOUT 环境变量的值，如果未设置，则默认为 60 秒。

`-V`
`--version`

Print the pg_ctl version and exit.

打印 pg_ctl 版本并退出。

`-w`

`--wait`

Wait for the operation to complete. This is supported for the modes start, stop, restart, promote, and register, and is the default for those modes. When waiting, pg_ctl repeatedly checks the server's PID file, sleeping for a short amount of time between checks. Startup is considered complete when the PID file indicates that the server is ready to accept connections. Shutdown is considered complete when the server removes the PID file. pg_ctl returns an exit code based on the success of the startup or shutdown.

等待操作完成。启动、停止、重新启动、升级和注册模式支持此功能，并且是这些模式的默认设置。等待时，pg_ctl 会重复检查服务器的 PID 文件，在检查之间休眠一小段时间。当 PID 文件指示服务器已准备好接受连接时，启动被视为完成。当服务器删除 PID 文件时，关闭被视为完成。 pg_ctl 根据启动或关闭是否成功返回退出代码。

If the operation does not complete within the timeout (see option -t), then pg_ctl exits with a nonzero exit status. But note that the operation might continue in the background and eventually succeed.

如果操作未在超时内完成（请参阅选项 -t），则 pg_ctl 将以非零退出状态退出。但请注意，该操作可能会在后台继续进行并最终成功。

`-W`

`--no-wait`

Do not wait for the operation to complete. This is the opposite of the option -w.

不要等待操作完成。这与选项 -w 相反。

If waiting is disabled, the requested action is triggered, but there is no feedback about its success. In that case, the server log file or an external monitoring system would have to be used to check the progress and success of the operation.

如果禁用等待，则会触发请求的操作，但没有有关其成功的反馈。在这种情况下，必须使用服务器日志文件或外部监控系统来检查操作的进度和成功。

In prior releases of PostgreSQL, this was the default except for the stop mode.

在 PostgreSQL 的早期版本中，这是默认模式（停止模式除外）。

`-?`

`--help`

Show help about pg_ctl command line arguments, and exit.

显示有关 pg_ctl 命令行参数的帮助，然后退出。

If an option is specified that is valid, but not relevant to the selected operating mode, pg_ctl ignores it.

如果指定的选项有效，但与所选操作模式无关，则 pg_ctl 会忽略它。

## Options for Windows

`-e source`

Name of the event source for pg_ctl to use for logging to the event log when running as a Windows service. The default is PostgreSQL. Note that this only controls messages sent from pg_ctl itself; once started, the server will use the event source specified by its event_source parameter. Should the server fail very early in startup, before that parameter has been set, it might also log using the default event source name PostgreSQL.

作为 Windows 服务运行时，pg_ctl 用于记录到事件日志的事件源的名称。默认是 PostgreSQL。请注意，这仅控制从 pg_ctl 本身发送的消息；一旦启动，服务器将使用其 event_source 参数指定的事件源。如果服务器在启动初期（在设置该参数之前）失败，它也可能使用默认事件源名称 PostgreSQL 进行记录。

`-N servicename`

Name of the system service to register. This name will be used as both the service name and the display name. The default is PostgreSQL.

要注册的系统服务的名称。该名称将用作服务名称和显示名称。默认是 PostgreSQL。

`-P password`

Password for the user to run the service as.

运行服务的用户的密码。

`-S start-type`

Start type of the system service. start-type can be auto, or demand, or the first letter of one of these two. If this option is omitted, auto is the default.

系统服务的启动类型。 start-type 可以是 auto、demand 或这两者之一的第一个字母。如果省略此选项，则默认为 auto。

`-U username`

User name for the user to run the service as. For domain users, use the format DOMAIN\username.

运行服务的用户的用户名。对于域用户，请使用 DOMAIN\username 格式。

## Environment

`PGCTLTIMEOUT`

Default limit on the number of seconds to wait when waiting for startup or shutdown to complete. If not set, the default is 60 seconds.

等待启动或关闭完成时等待的秒数的默认限制。如果未设置，则默认为 60 秒。

`PGDATA`

Default data directory location.

默认数据目录位置。

Most pg_ctl modes require knowing the data directory location; therefore, the -D option is required unless PGDATA is set.

大多数 pg_ctl 模式需要知道数据目录位置；因此，除非设置了 PGDATA，否则需要 -D 选项。

pg_ctl, like most other PostgreSQL utilities, also uses the environment variables supported by libpq(see Section 34.15).

pg_ctl 与大多数其他 PostgreSQL 实用程序一样，也使用 libpq 支持的环境变量（参见第 34.15 节）。

For additional variables that affect the server, see postgres.

有关影响服务器的其他变量，请参阅 postgres。

## Files

`postmaster.pid`

pg_ctl examines this file in the data directory to determine whether the server is currently running.

pg_ctl 检查数据目录中的此文件以确定服务器当前是否正在运行。

`postmaster.opts`

If this file exists in the data directory, pg_ctl (in restart mode) will pass the contents of the file as options to postgres, unless overridden by the -o option. The contents of this file are also displayed in status mode.

如果该文件存在于数据目录中，pg_ctl（在重新启动模式下）将把该文件的内容作为选项传递给 postgres，除非被 -o 选项覆盖。该文件的内容也在状态模式下显示。

## Examples

## Starting the Server

To start the server, waiting until the server is accepting connections:

`$ pg_ctl start`

To start the server using port 5433, and running without fsync, use:

`$ pg_ctl -o "-F -p 5433" start`

## Stopping the Server

To stop the server, use:

`$ pg_ctl stop`

The -m option allows control over how the server shuts down:

`$ pg_ctl stop -m smart`

## Restarting the Server

Restarting the server is almost equivalent to stopping the server and starting it again, except that by default, pg_ctl saves and reuses the command line options that were passed to the previously-running instance. To restart the server using the same options as before, use:

重新启动服务器几乎等同于停止服务器并再次启动它，除了默认情况下，pg_ctl 保存并重用传递给先前运行的实例的命令行选项。要使用与之前相同的选项重新启动服务器，请使用：

`$ pg_ctl restart`

But if -o is specified, that replaces any previous options. To restart using port 5433, disabling fsync upon restart:

但如果指定了 `-o`，则会替换之前的任何选项。要使用端口 5433 重新启动，请在重新启动时禁用 fsync：

`$ pg_ctl -o "-F -p 5433" restart`

## Showing the Server Status

Here is sample status output from pg_ctl:

`$ pg_ctl status`

```sh
pg_ctl: server is running (PID: 13718)
/usr/local/pgsql/bin/postgres "-D" "/usr/local/pgsql/data" "-p" "5433" "-B" "128"
The second line is the command that would be invoked in restart mode.
```

## See Also

initdb, postgres
