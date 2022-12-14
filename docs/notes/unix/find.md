---
id: find
title: find
---

## `find`

### Find by name

```shell
find -name "query"
find -iname "query" # Ignore case
find -not -name "query_to_avoid"
find \! -name "query_to_avoid"
```

### Find by type

```shell
find -type <type_descriptor> <query> # Basic usage
# Common type descriptors:
# f: File
# d: Directory
# l: Symlink
# c: Character device
# b: Block device

find /usr -type f -name "*.conf" # Search for files ending in `.conf` in `/usr` directory
find /usr -type f -and -name "*.conf" # Equivalent to above; `-and` combines two queries
find -name query_1 -or -name query_2 # `-or` returns results that match either expression
```

### Find by size

```shell
find -size <number><suffix> # Basic usage
# Common suffixes
# c: Bytes
# k: Kilobytes
# M: Megabytes
# G: Gigabytes
# b: 512-byte blocks

find /usr -size 50c # Find all files in `/usr` exactly 50 bytes in size
find /usr -size -50c # Less than 50 bytes in size
find /usr -size +600M # More than 600 megabytes in size
```

### Find by time

:::info

- **Access time (`atime`)**: The last time a file was read or written to.
- **Modification time (`mtime`)**: The last time the contents of the file were modified.
- **Change time (`ctime`)**: The last time the file’s inode metadata was changed.

:::

```shell
find /usr -mtime 1 # Files modified within the last day
find /usr -atime -1 # Accessed less than a day ago
find /usr -ctime +3 # Meta information changed more than 3 days ago
find /usr -mmin -1 # Modified within the last minute
find / -newer reference_file # Created/changed more recently than reference_file
```

### Find by owner

```shell
find / -user <user>
find / -group <group>
```

### Find by permissions

```shell
find / -perm 644
find / -perm -644 # Files with at least these permisions
```

### Find by depth

```shell
find -mindepth <num>
find -maxdepth <num>
```

### Execute commands on `find` results

Other commands can be executed on results returned by `find` using the `-exec`/`-delete` options or by piping the output to the [`xargs`](/notes/unix/xargs) command or [GNU Parallel](<https://en.wikipedia.org/wiki/Parallel_(software)>).

```shell
find <find_parameters> -exec <command_and_options> {} \;
find . -type d -perm 755 -exec chmod 700 {} \; # Change directory permissions from 755 to 700
find . -name "*.json" -delete # Delete files ending in `.json`

# Useful options
# -exec command {} +: Build command by appending each selected file name at the end, and then execute it
# -execdir: Run command from the subdirectory containing the matched file

find ./docs -type f -print | xargs rm
# Find all files in `./docs` and remove them

find . -name "*.foo" -print0 | xargs -0 grep bar
# Use the null character to delimit file names (necessary for dealing with filenames with `,` or space)

find . -name "*.foo" -print0 | parallel -0 grep bar
# Equivalent to above
```

## `locate`

:::info

`locate` is faster because it searches through a prebuilt database of files generated by the `updatedb` command or a cronjob automatically created during installation that runs every 24 hours. To manually update the database, use `sudo updatedb`.

:::

```shell
locate <query> # Return files whose full path names contain the query
locate -b <query> # Return only files whose names contain the query
locate -n 10 *.py # Limit to 10 results; Wildcard search
locate -i readme.md # Case-insensitive search
locate -c *.md # Display number of found entries
locate -S # Print statistics about each used database
```

## `whereis`

`whereis` command can be used to efficiently locate the binary, source, and manual page files for a command.

```shell
> whereis rg
rg: /usr/bin/rg /usr/share/man/man1/rg.1.gz
```

## `which`

`which` searches for the binary for a command in your `PATH`.

```shell
> which rg
/usr/bin/rg
```

## `fd`

:::info

[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh/)'s [common-aliases](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/common-aliases/common-aliases.plugin.zsh) plugin aliases `fd` to `find . -type d -name`. To avoid the clash between this alias and `fd`, put `unalias fd` or `alias fd='\fd'` at the end of your `.zshrc`.

:::

### Basic usage

```shell
fd [options] <pattern> <path> # Basic usage
fd # List all files in current directory recursively, similar to `ls -R`

fd netfl # Basic search; Recursively search current directory for the pattern `netfl`
fd '^x.*rc$' # Regex search; Search for entries that start with `x` and end with `rc`
fd '^x.*rc$' /etc # Search in `/etc` directory
fd -g libc.so /usr # Glob-based search; Find all `libc.so` files in `usr`

# Useful options
# -H, --hidden: Search in hidden directories
# -I, --no-ignore: Search directories and show files that match `.gitignore` patterns
# -p, --full-path: Search in full paths instead of just filenames
# -l, --list-details: Use a long listing format with file metadata
# -L, --follow: Follow symbolic links
```

### Find by type

```shell
fd -t <filetype> <pattern>
# Common filetypes:
# f: File
# d: Directory
# l: Symlink
# x: Executable
# e: Empty
# s: Socket
# p: Pipe
```

### Find by size

```shell
fd -s <size> <pattern>
```

### Find by time

```shell
fd --changed-within <date|dur> <pattern> # Filter by file modification time (newer than)
fd --changed-before <date|dur> <pattern> # Filter by file modification time (older than)
```

### Find by depth

```shell
fd -d <depth> <pattern> # Set maximum search depth
```

### Find by extension

```shell
fd -e json # Find files with `.json` extension
fd -e json <pattern> # Find json files that contain the pattern
```

### Exclude files and directories

```shell
fd -E '*.zip' <pattern> # Exclude zip files
fd -H -E .git <pattern> # Search in hidden dirs but exclude matches from `.git`
# You can create a `.fdignore` file (similar to `.gitignore`) to make exclude-patterns permanent.
```

### Execute commands on `fd` results

```shell
# -x, --exec <cmd>: Execute a command for each search result
# -X, --exec-batch <cmd>: Execute a command with all search results at once

fd -e zip -x unzip # Recursively find and unzip all zip archives
fd -e h -e cpp -x clang-format -i # Find all `*.h` and `*.cpp` files and auto-format them inplace with `clang-format -i`

fd -g 'test_*.py' -X vim # Find all `test_*.py` files and open them with `vim`
# https://vimhelp.org/usr_07.txt.html#07.2

fd -H '^\.DS_Store$' -tf -X rm # Recursively remove all `.DS_Store` files
```

### Placeholder tokens

```shell
fd -e jpg -x convert {} {.}.png # Convert all `*.jpg` files to `*.png` files
# `{}` will be replaced by the path of the search result (`docs/images/party.jpg`)

# Other placeholder tokens
# {.}: Like {}, but without the file extension (`docs/images/party`)
# {/}: The basename of the search result (`party.jpg`)
# {//}: The parent of the discovered path (`docs/images`)
# {/.}: The basename without the extension (`party`)
```
