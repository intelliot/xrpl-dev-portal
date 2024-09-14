---
html: fix-sqlite-tx-db-page-size-issue.html
parent: troubleshoot-the-rippled-server.html
seo:
    description: Fix a problem with the SQLite page size on full-history servers.
---
# Fix SQLite Transaction Database Page Size Issue

`rippled` servers with full [ledger history](../../concepts/networks-and-servers/ledger-history.md) (or a very large amount of transaction history) may encounter a problem with their SQLite database page size and/or maximum page count that stops the server from operating properly. Servers that store only recent transaction history (the default configuration) are not likely to have this problem. <!-- STYLE_OVERRIDE: encounter -->

This document describes steps to detect and correct this problem if it occurs.

## Background

`rippled` servers store a copy of their transaction history in a SQLite database. Before version 0.40.0, `rippled` configured this database to have a capacity of roughly 2 TB. For most uses, this was plenty. However, full transaction history back to ledger 32570 (the oldest ledger version available in the production XRP Ledger history) exceeds that SQLite database capacity. `rippled` servers version 0.40.0 and later create their SQLite database files with a larger capacity, roughly 8 TB. Starting with version 2.2.3, `rippled` attempts to set `max_page_count=4294967294`, which is the SQLite default as of SQLite 3.45.0 (2024-01.15). With a page size of 4096 bytes, this allows for roughly 17 TB. With a page size of 65536 bytes, the database can grow to roughly 281 TB.

The capacity of the SQLite database is a result of the database's _page size_ parameter, which cannot be easily changed after the database is created, and SQLite's _max_page_count_ parameter. (For more information on SQLite's internals, see [the official SQLite documentation](https://www.sqlite.org/fileformat.html).) The database can reach its capacity even if there is still free space on the disk and filesystem where it is stored. As described in the [Fix](#fix) below, reconfiguring the page size to avoid this problem requires a somewhat time-consuming migration process. <!-- STYLE_OVERRIDE: easily -->

**Tip:** Full history is not necessary for most use cases. Servers with full transaction history may be useful for long-term analysis and archive purposes or as a precaution against disasters. For a less resource-intense way to store transaction history, see [the Clio server](../../concepts/networks-and-servers/the-clio-server).


## Detection

If your server is vulnerable to this problem, you can detect it two ways:

- You can detect the problem [proactively](#proactive-detection) (before it causes problems)
- You can identify the problem [reactively](#reactive-detection) (when your server is crashing)

**Tip:** The location of the debug log depends on your `rippled` server's config file. The [default configuration](https://github.com/XRPLF/rippled/blob/master/cfg/rippled-example.cfg#L1139-L1142) writes the server's debug log to the file `/var/log/rippled/debug.log`.

The `rippled` server writes a message such as the following in its debug log periodically, at least once every 2 minutes. (The exact numeric values from the log entry and the path to your transaction database depend on your environment.)

```text
Transaction DB pathname: /opt/rippled/transaction.db; SQLite page size: 1024
  bytes; Free pages: 247483646; Free space: 253423253504 bytes; Note that this
  does not take into account available disk space.
```

The value `SQLite page size: 1024 bytes` indicates that your transaction database is configured with a small page size and does not have capacity for full transaction history. If the value is already 4096 bytes or higher, then your SQLite database may have adequate capacity to store full transaction history.

The maximum size of the transaction database can be calculated by multiplying the page size and the maximum page count. For example, if the page size is 4096 bytes, and `max_page_count=2147483646`:

4096 bytes * 2147483646 = 8.79609301 terabytes.

The `rippled` server halts if the `Free space` described in this log message becomes less than 524288000 bytes (500 MB). If your free space is approaching that threshold, [fix the problem](#fix) to avoid an unexpected outage.

### Reactive Detection

If your server's SQLite database capacity has already been exceeded, the `rippled` service writes a log message indicating the problem and halts. The server shuts down gracefully with a message such as the following in the server's debug log:

```text
Free SQLite space for transaction db is less than 512MB. To fix this, rippled
  must be executed with the vacuum <sqlitetmpdir> parameter before restarting.
  Note that this activity can take multiple days, depending on database size.
```

## Fix

You can fix this issue using `rippled` on supported Linux systems according to the steps described in this document. In the case of a full-history server with system specs approximately matching the [recommended hardware configuration](../installation/capacity-planning.md#recommendation-1), the process may take more than two full days.

### Prerequisites

- [Upgrade rippled](../installation/index.md) to the latest stable version before starting this process.

    - You can check what version of `rippled` you have installed locally by running the following command:

        ```
        rippled --version
        ```

- You must have enough free space to temporarily store a second copy of the transaction database, in a directory that is writable by the `rippled` user. This free space does not need to be in the same filesystem as the existing transaction database.

    The transaction database is stored in the `transaction.db` file in the folder specified by your configuration's `[database_path]` setting. You can check the size of this file to see how much free space you need. For example:

    ```
    ls -l /var/lib/rippled/db/transaction.db
    ```

### Migration Process

To migrate your transaction database to a larger page size, perform the following steps:

1. Check that you meet all the [prerequisites](#prerequisites).

2. Create a folder to store temporary files during the migration process.

    ```
    mkdir /tmp/rippled_txdb_migration
    ```

3. Grant the `rippled` user ownership of the temporary folder so it can write files there. (This is not necessary if your temporary folder is somewhere the `rippled` user already has write access to.)

    ```
    chown rippled /tmp/rippled_txdb_migration
    ```

4. Confirm that your temporary folder has enough free space to store a copy of the transaction database.

    For example, compare the `Avail` output from the `df` command to the [size of your `transaction.db` file](#prerequisites).

    ```
    df -h /tmp/rippled_txdb_migration

    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda2       5.4T  2.6T  2.6T  50% /tmp
    ```

5. If `rippled` is still running, stop it:

    ```
    sudo systemctl stop rippled
    ```

6. Open a `screen` session (or other similar tool) so that the process does not stop when you log out:

    ```
    screen
    ```

7. Become the `rippled` user:

    ```
    sudo su - rippled
    ```

8. Run `rippled` executable directly, providing the `--vacuum` command with the path to the temporary directory:

    ```
    /opt/ripple/bin/rippled -q --vacuum /tmp/rippled_txdb_migration
    ```

    The `rippled` executable immediately displays the following message:

    ```
    VACUUM beginning. page_size: 1024
    ```

9. Wait for the process to complete. This can take more than two full days.

    When the process is complete, the `rippled` executable displays the following message, then exits:

    ```
    VACUUM finished. page_size: 4096
    ```

    While you wait, you can detach your `screen` session by pressing **CTRL-A**, then **D**. Later, you can reattach your screen session with a command such as the following:

    ```
    screen -x -r
    ```

    When the process is over, exit the screen session:

    ```
    exit
    ```

    For more information on the `screen` command, see [the official Screen User's Manual](https://www.gnu.org/software/screen/manual/screen.html) or any of the other many resources available online.

10. Restart the `rippled` service.

    ```
    sudo systemctl start rippled
    ```

11. Confirm that the `rippled` service started successfully.

    You can use the [commandline interface](../../tutorials/http-websocket-apis/build-apps/get-started.md#commandline) to check the server status (unless you have configured your server not to accept JSON-RPC requests). For example:

    ```
    /opt/ripple/bin/rippled server_info
    ```

    For a description of the expected response from this command, see the [server_info method][] documentation.

12. Watch the server's debug log to confirm that the `SQLite page size` is now 4096:

    ```
    tail -F /var/log/rippled/debug.log
    ```

    The [periodic log message](#proactive-detection) should also show significantly more free pages and free pages than it did before the migration.

13. Optionally, you may now remove the temporary folder you created for the migration process.

    ```
    rm -r /tmp/rippled_txdb_migration
    ```

    If you mounted additional storage to hold the temporary copy of the transaction database, you can unmount and remove it now.


## See Also

- **Concepts:**
    - [The `rippled` Server](../../concepts/networks-and-servers/index.md)
    - [Ledger History](../../concepts/networks-and-servers/ledger-history.md)
- **Tutorials:**
    - [Understanding Log Messages](understanding-log-messages.md)
    - [Configure Full History](../configuration/data-retention/configure-full-history.md)
- **References:**
    - [rippled API Reference](../../references/http-websocket-apis/index.md)
        - [`rippled` Commandline Usage](../commandline-usage.md)
        - [server_info method][]

{% raw-partial file="/docs/_snippets/common-links.md" /%}
