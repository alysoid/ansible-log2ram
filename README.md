# [Catena](https://github.com/alysoid/catena) - Log2ram Ansible Role

[Log2Ram](https://github.com/azlux/log2ram) helps not to stress a drive (like a SD Card on RaspberryPi) with continuous log writing. The script creates a `/var/log` mount point in RAM, so any writing to the `/var/log` folder will not be actually written on disk but directly on RAM. It works great in combination with [systemd-swap](https://github.com/Nefelim4ag/systemd-swap) and [zram](https://www.kernel.org/doc/Documentation/blockdev/zram.txt).

## Role variables

### `log2ram`

```yaml
# Defaults
log2ram:
  # Version to install (GitHub tags)
  # https://github.com/azlux/log2ram/releases
  version: "1.6.1"
  # Size the log folder will reserve into the RAM
  size: 40M
  # When `true` use rsync for log syncing, when `false` use cp
  rsync: true
  # Use system mail to get notified on errors
  mail: false
  # List of folders to put in RAM
  path: ["/var/log"]
  # When `true` use zram, when `false` use tmpfs
  zram: true
  # Compression algorithm when zram is enabled
  compression: lz4
```

#### `log2ram.size`

Defines the size of the log folder that will be reserved into the RAM. If it's not enough, log2ram will not be able to work correctly. Please check the size of `/var/log` folder before making decisions. The default is `40M` and it should be enough for a lot of applications. For a server with many log storage, it can be increased to `100M` or more.

The available space can be increased setting the `log2ram.zram` variable to `true` and specifing a [`log2ram.compression`](#log2ram_compression_algorithm) algorithm to use.

#### `log2ram.rsync`

Select the log syncing method between the log directory on disk and in the RAM. When `true`, use `rsync`, instead use `cp`, that is also available for fallback when `rsync` is missing.

#### `log2ram.mail`

When `true`, uses the system `mail` binary to notify the user about errors with available RAM space. Change it to `false` to only use a log when there's no more place on RAM.

#### `log2ram.path`

List of folders to store in RAM. Always specify an existing folder `/path/folder` without using the final `/`.

The `/path/hdd.folder` will be automatically created.

Example with multiple folders:

```yaml
log2ram:
  path:
    - "/var/log"
    - "/home/user/ramfolder"
```

#### `log2ram.compression`

It's a chose compression algorithm listed in `/proc/crypto`. The fastest and lightest `lz4` or `zstd` (Zstandard) for better compression ratios, are the recommended choices.

| Compressor name   | Ratio | Compression | Decompress. |
| ----------------- | ----- | ----------- | ----------- |
| `zstd` 1.4.5 -1   | 2.884 | 500 MB/s    | 1660 MB/s   |
| `zlib` 1.2.11 -1  | 2.743 | 90 MB/s     | 400 MB/s    |
| `brotli` 1.0.7 -0 | 2.703 | 400 MB/s    | 450 MB/s    |
| `lzo`1x 2.10 -1   | 2.106 | 690 MB/s    | 830 MB/s    |
| `lz4` 1.9.2       | 2.101 | 740 MB/s    | 4530 MB/s   |
| `snappy` 1.1.8    | 2.073 | 560 MB/s    | 1790 MB/s   |

### `log2ram_version`

```yaml
# Defaults
log2ram_version: "{{ log2ram.version | default('1.6.1') }}"
```

Version of Log2Ram to install, see [GitHub releases](https://github.com/azlux/log2ram/releases). The version can also be set in `log2ram.version` but this variable has the precedence.

### `log2ram_journal_cleanup`

When is `true` executes the command: `journalctl --vacuum-size=` targeting the chosen `log2ram_size` - 5%. This reduces the `/var/log` folder size by removing the oldest systemd journals before re/start log2ram service. This should avoid most of the errors due to a too big existing folder.
