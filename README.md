# Command Cursor Editor on Linux

This guide will help you set up a custom command to run the Cursor Editor from the terminal on Linux.

## Steps

### 1. Create the Command Script

Open a terminal and create a new file using your preferred text editor. If you want the command to be `cursor`, use the following command:

```sh
sudo nano /usr/bin/cursor
```
If you prefer to use the command `code`, use this command instead (not recommended as it may override VSCode functionality if installed):
```sh
sudo nano /usr/bin/code
```
### 2. Add the Following Code
Copy and paste the following code into the file you just created:
```sh
#!/usr/bin/env sh

# Path to the cursor editor appimage
CURSOR_APPIMAGE="/opt/cursor/cursor.appimage"

# Check if the appimage exists
if [ ! -f "$CURSOR_APPIMAGE" ]; then
    echo "Cursor appimage not found at $CURSOR_APPIMAGE" 1>&2
    exit 1
fi

# Ensure the appimage is executable
chmod +x "$CURSOR_APPIMAGE"

# If running in a remote terminal, use the remote CLI (adjust as needed for cursor)
if [ -n "$VSCODE_IPC_HOOK_CLI" ]; then
    REMOTE_CLI="$(which -a 'cursor' | grep /remote-cli/)"
    if [ -n "$REMOTE_CLI" ]; then
        "$REMOTE_CLI" "$@"
        exit $?
    fi
fi

# Test that cursor wasn't installed inside WSL
if grep -qi Microsoft /proc/version && [ -z "$DONT_PROMPT_WSL_INSTALL" ]; then
    echo "To use Cursor with the Windows Subsystem for Linux, please install Cursor in Windows and uninstall the Linux version in WSL. You can then use the \`cursor\` command in a WSL terminal just as you would in a normal command prompt." 1>&2
    printf "Do you want to continue anyway? [y/N] " 1>&2
    read -r YN
    YN=$(printf '%s' "$YN" | tr '[:upper:]' '[:lower:]')
    case "$YN" in
        y | yes )
        ;;
        * )
            exit 1
        ;;
    esac
    echo "To no longer see this prompt, start Cursor with the environment variable DONT_PROMPT_WSL_INSTALL defined." 1>&2
fi

# If root, ensure that --user-data-dir or --file-write is specified
if [ "$(id -u)" = "0" ]; then
    for i in "$@"
    do
        case "$i" in
            --user-data-dir | --user-data-dir=* | --file-write | tunnel | serve-web )
                CAN_LAUNCH_AS_ROOT=1
            ;;
        esac
    done
    if [ -z $CAN_LAUNCH_AS_ROOT ]; then
        echo "You are trying to start Cursor as a super user which isn't recommended. If this was intended, please add the argument \`--no-sandbox\` and specify an alternate user data directory using the \`--user-data-dir\` argument." 1>&2
        exit 1
    fi
fi

# Run the cursor appimage
"$CURSOR_APPIMAGE" "$@"
exit $?

```
Replace the CURSOR_APPIMAGE variable with the path to your cursor.appimage if it is located elsewhere.

### 3. Give Execution Permission
After saving the file, give it execution permissions with the following command:
```sh
sudo chmod +x /usr/bin/cursor
```
Or if you used the `code` command:
```sh
sudo chmod +x /usr/bin/code
```

### 4. Use the Command
Now you can open the Cursor Editor by simply typing:
```sh
cursor .
```
Or, if you chose the code command:
```sh
code .
```
