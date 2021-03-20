# [Catena](https://github.com/alysoid/catena) - Log2ram Ansible Role

[Log2Ram](https://github.com/azlux/log2ram) helps not to stress a drive (like a SD Card on RaspberryPi) with continuous log writing. The script creates a `/var/log` mount point in RAM, so any writing to the `/var/log` folder will not be actually written on disk but directly on RAM. It works great in combination with [systemd-swap](https://github.com/Nefelim4ag/systemd-swap) and [zram](https://www.kernel.org/doc/Documentation/blockdev/zram.txt).

## Role variables

| Variable                        | Default      | Info
| ------------------------------- | ------------ | --------------------------
| `log2ram_version`               | `'1.6.0'`    | Version to install (see: [GitHub releases](https://github.com/azlux/log2ram/releases)).
| `log2ram_size`                  | `'40M'`      | Size the log folder will reserve into the RAM.
| `log2ram_send_mail`             | `yes`        | Use system mail to get notified on errors.
| `log2ram_path_disk`             | `'/var/log'` | Semicolon separated list of folders to put in RAM.
| `log2ram_zram_enabled`          | `false`      | Type of device to use. `true` for zram, `false` for default tmpfs.
| `log2ram_compression_algorithm` | `'lz4'`      | When zram is enabled, specify the compression algorithm to use.

---

### `log2ram_size`

Defines the size of the log folder that will be reserved into the RAM. If it's not enough, log2ram will not be able to work correctly. Please check the size of `/var/log` folder before making decisions. The default is `40M` and it should be enough for a lot of applications. For a server with many log storage, it can be increased to `100M` or more.

The available space can be increased setting the `log2ram_zram_enabled` variable to `true` and specifing a [`log2ram_compression_algorithm`](#log2ram_compression_algorithm) to use.

### `log2ram_send_mail`

When `yes`, uses the system `mail` binary to notify the user about errors with available RAM space. Change it to `no` to only use a log when there's no more place on RAM.

### `log2ram_path_disk`

Folders to put in RAM. Always specify an existing folder `/path/folder` without using the final `/`.

Separate paths using the semicolon: `';'` character. E.g.: `log2ram_path_disk: '/var/log;/home/user/RamFolder'`

The `/path/hdd.folder` will be automatically created.

### `log2ram_compression_algorithm`

It's a chose compression algorithm listed in `/proc/crypto`. The fastest and lightest `lz4` or `zstd` (Zstandard) for better compression ratios, are the recommended choices.

| Compressor name     | Ratio | Compression | Decompress. |
|---------------------|-------|-------------|-------------|
| `zstd` 1.4.5 -1     | 2.884 | 500 MB/s    | 1660 MB/s   |
| `zlib` 1.2.11 -1    | 2.743 | 90 MB/s     | 400 MB/s    |
| `brotli` 1.0.7 -0   | 2.703 | 400 MB/s    | 450 MB/s    |
| `lzo`1x 2.10 -1     | 2.106 | 690 MB/s    | 830 MB/s    |
| `lz4` 1.9.2         | 2.101 | 740 MB/s    | 4530 MB/s   |
| `snappy` 1.1.8      | 2.073 | 560 MB/s    | 1790 MB/s   |
