Certainly! Here's the revised script explanation with links to the HNG Internship websites:

---

# DevOps-Stage-1-Linux-User-Creation-Bash-Script
Internship Program

This script is designed to create users and assign them to specified groups based on an input file. It logs all actions and securely stores user passwords.

## Script Explanation

### Shebang and Description
```bash
#!/bin/bash
```
- The script starts with `#!/bin/bash`, indicating it should be run in the Bash shell.

### Check for Arguments
```bash
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <user_file>"
    exit 1
fi
```
- The script checks if exactly one argument (the input file) is provided. If not, it displays usage information and exits.

### File Existence Check
```bash
USER_FILE="$1"

if [ ! -f "$USER_FILE" ]; then
    echo "Error: File $USER_FILE does not exist."
    exit 1
fi
```
- The script checks if the provided input file exists. If not, it displays an error message and exits.

### Variable Assignments
```bash
LOG_FILE="/var/log/user_management.log"
PASSWORD_DIR="/var/secure"
PASSWORD_FILE="$PASSWORD_DIR/user_passwords.csv"
```
- The paths for the log file and password file are assigned to variables for easy reference.

### Log File Creation
```bash
touch $LOG_FILE
```
- The script ensures the log file exists by using `touch`.

### Password Directory and File Setup
```bash
if [ ! -d "$PASSWORD_DIR" ]; then
    mkdir -p "$PASSWORD_DIR"
    chmod 700 "$PASSWORD_DIR"
fi

if [ ! -f "$PASSWORD_FILE" ]; then
    echo "username,password" >> $PASSWORD_FILE
fi
chmod 600 $PASSWORD_FILE
```
- The script checks if the password directory exists and creates it with restricted permissions if not.
- If the password file doesn't exist, it is created with a header, and its permissions are set to be readable only by the owner.

### Logging Function
```bash
log_action() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}
```
- A function `log_action` is defined to log actions with timestamps to the log file.

### Reading and Processing the Input File
```bash
while IFS=';' read -r username groups; do

    username=$(echo "$username" | xargs)
    groups=$(echo "$groups" | xargs)
```
- The script reads the input file line by line, using `IFS=';'` to split each line into a username and groups.
- Leading and trailing whitespace is removed from both the username and groups.

### User and Group Creation
```bash
    if id -u "$username" >/dev/null 2>&1; then
        log_action "User $username already exists."
        continue
    fi
    
    if ! getent group "$username" >/dev/null 2>&1; then
        groupadd "$username"
    fi
    
    useradd -m -g "$username" -s /bin/bash "$username"
    
    chmod 700 /home/"$username"
    chown "$username:$username" /home/"$username"
    
    password=$(openssl rand -base64 12)
    
    echo "$username:$password" | chpasswd
    
    echo "$username,$password" >> $PASSWORD_FILE
```
- The script checks if a user already exists. If so, it logs the information and skips to the next user.
- If not, it creates a personal group for the user if it doesn't already exist.
- The user is created with a home directory and added to their personal group.
- Home directory permissions are set to be accessible only by the user.
- A random password is generated, set for the user, and stored in the password file.

### Additional Groups
```bash
    if [ -n "$groups" ]; then
        IFS=',' read -r -a group_array <<< "$groups"
        for group in "${group_array[@]}"; do
            group=$(echo "$group" | xargs)
            if ! getent group "$group" >/dev/null 2>&1; then
                groupadd "$group"
            fi
            usermod -aG "$group" "$username"
        done
    fi
    
    log_action "User $username created with groups: $username, $groups"
done < $USER_FILE
```
- If additional groups are specified, the script splits the groups by commas, creates any missing groups, and adds the user to these groups.

### Logging Completion
```bash
log_action "User creation process completed."
```
- Actions such as user creation and group assignments are logged.
- A final log entry indicates the completion of the user creation process.

---

This Markdown document provides a detailed explanation of each part of the script, helping users understand its functionality and structure.

For more information about HNG Internship programs, visit [HNG Internship](https://hng.tech/internship) and [HNG Premium](https://hng.tech/premium).