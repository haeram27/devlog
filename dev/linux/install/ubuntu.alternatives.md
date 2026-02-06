# update-alternatives

### List all master alternative names

```bash
update-alternatives --get-selections
```

### To see all symbolic links managed in /etc/alternatives

```bash
ls -l /etc/alternatives
```

### Install a new alternative path

```bash
sudo update-alternatives --install <link> <name> <path> <priority>
# Example: sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 90
```

### List all choices for a specific command

```bash
update-alternatives --list <command_name>
# Example: update-alternatives --list java
```

### Configure/Switch default version interactively

```bash
sudo update-alternatives --config <command_name>
# Example: sudo update-alternatives --config gcc
```

### Display status and available paths for a command

```bash
update-alternatives --display <command_name>
```
