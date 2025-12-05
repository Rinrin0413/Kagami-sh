# 🪞 Kagami.sh

My backup tool that works on my machine.

**Kagami.sh** is a backup script for Linux.
It creates a mirror of your specified directories using [rsync](https://github.com/RsyncProject/rsync).

## Features

- **Incremental Mirroring**: Uses rsync to efficiently sync changes.
- **Configurable**: Simple text files for configuration, targets, and exclusions.
- **Safety Checks**: Prevents accidental overwrites of system directories (e.g., `/`, `/usr`).
- **Dry Run**: Preview changes before they happen.
- **Logging**: Keeps a log of every run in the destination folder.
- **Stale Target Retention**: If you remove targets from your backup list,
  its existing backup is moved to a trash folder `.kagami-trash/` in the destination folder instead of being deleted immediately.

## How to Use

### Prerequisites

- Linux environment
- Bash
- [rsync](https://github.com/RsyncProject/rsync)
- Standard utility commands: `tput`, `realpath`, `find`, `grep`
- Git (if cloning from GitHub)

### Installation

1. Clone the repository and switch to the `release` branch:
   ```bash
   git clone https://github.com/Rinrin0413/Kagami-sh.git
   cd Kagami-sh
   git switch release
   ```

2. Make the script executable:
   ```bash
   chmod +x kagami.sh
   ```

### Configuration

The script relies on three configuration files in the same directory:

1. [**`kagami.conf`**](/kagami.conf) (Required)  
   Set the destination path and trash retention period.
   ```bash
   DEST="/run/media/USER/FOO-DRIVE/kagami-backups/"
   TRASH_RETENTION_DAYS=30
   ```

2. [**`targets.conf`**](/targets.conf) (Required)  
   List the files and directories to backup (one per line).
   ```conf
   /etc
   ~/Pictures
   ~/Videos/Screencasts
   ~/.bashrc
   ```

3. [**`excludes.conf`**](/excludes.conf) (Optional)  
   List patterns to exclude (passed to `rsync --exclude-from`).  
   For more details, see the "Include/Exclude Pattern Rules" section in:
   [rsync(1) - Linux man page](https://linux.die.net/man/1/rsync)
   ```conf
   node_modules/
   .git/
   *.log
   /home/USER/Downloads/*.zip
   ```

## Usage

> [!NOTE]
> The first run will ask for confirmation to initialize the destination directory.
> 
> So if you plan to run this from non-interactive scripts (e.g., cron),
> run it once interactively first to initialize the destination directory.

### Dry Run

Check what will be changed without modifying any files:

```bash
./kagami.sh dry
```

> [!NOTE]
> Dry runs do not save logs to `.kagami-logs/` directory.

### Run Backup

```bash
./kagami.sh
```

## Destination Directory Structure

The backup destination directory will contain:

- **`path/to/source`**: The mirrored files (e.g., `home/USER/Pictures`).
- **`.kagami-manifest`**: Internal file to track backup targets.
- **`.kagami-logs/`**: Logs of output for each run.
- **`.kagami-trash/`**: Contains data from targets that were removed from `targets.conf`.
  These are automatically deleted during execution after `TRASH_RETENTION_DAYS`.

```
/path/to/backup-destination/
├── .kagami-logs/
│   └── 20250413_080000_1234.log
├── .kagami-manifest
├── .kagami-trash/
│   └── 20250413_080000_1234/
├── etc/                      ┓
└── home/                     ┃
    └── USER/                 ┃
        ├── .bashrc           ┠ Mirrored files
        ├── Pictures/         ┃
        └── Videos/           ┃
            └── Screencasts/  ┛
```

You can safely delete `.kagami-logs/` and `.kagami-trash/` directories manually if you do not need them.

## Important Notes

> [!IMPORTANT]
> - This script performs a **mirror** backup.
>   If you delete a file in the source directory (and it is still a valid target), it **will be deleted** from the backup destination.
>   It means this script does not keep previous backups.
> - The "Trash" feature only protects data when you remove a line from `targets.conf`.
>   It does not protect against accidental file deletions within an active target.
> - The developer of Kagami.sh assumes no responsibility for any data loss or damage caused by the use of this tool.

[![MIT](https://img.shields.io/github/license/Rinrin0413/Kagami-sh?color=%23A11D32&style=for-the-badge)](/LICENSE)
