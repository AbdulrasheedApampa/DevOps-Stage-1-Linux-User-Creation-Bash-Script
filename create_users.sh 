#!/bin/bash

# This script creates users and assigns them to specified groups based on an input file.
# The script logs all actions and stores user passwords securely.

# Check if a file name argument is provided
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <user_file>"
    exit 1
fi

# Assign the provided file name to a variable
USER_FILE="$1"

# Check if the user file exists
if [ ! -f "$USER_FILE" ]; then
    echo "Error: File $USER_FILE does not exist."
    exit 1
fi

# Assign log file and password file paths to variables
LOG_FILE="/var/log/user_management.log"
PASSWORD_DIR="/var/secure"
PASSWORD_FILE="$PASSWORD_DIR/user_passwords.csv"

# Create log file if it does not exist
touch $LOG_FILE

# Check if the password directory exists, and if not, create it with owner read/write/execute permissions
if [ ! -d "$PASSWORD_DIR" ]; then
    mkdir -p "$PASSWORD_DIR"
    chmod 700 "$PASSWORD_DIR"
fi

# If the password file doesn't exist, create it and add the header
if [ ! -f "$PASSWORD_FILE" ]; then
    echo "username,password" >> $PASSWORD_FILE
fi
chmod 600 $PASSWORD_FILE

# Create a logging function to log actions with timestamps
log_action() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Read the user file line by line
while IFS=';' read -r username groups; do

    # Remove leading/trailing whitespace from username and groups
    username=$(echo "$username" | xargs)
    groups=$(echo "$groups" | xargs)
    
    # Check if the user already exists
    if id -u "$username" >/dev/null 2>&1; then
        log_action "User $username already exists."
        continue
    fi
    
    # Create a personal group for the user if it doesn't exist
    if ! getent group "$username" >/dev/null 2>&1; then
        groupadd "$username"
    fi
    
    # Create the user with the personal group and home directory
    useradd -m -g "$username" -s /bin/bash "$username"
    
    # Set up the home directory permissions to be accessible only by the user
    chmod 700 /home/"$username"
    chown "$username:$username" /home/"$username"
    
    # Generate a random password for the user
    password=$(openssl rand -base64 12)
    
    # Set the password for the user
    echo "$username:$password" | chpasswd
    
    # Save the username and password to the secure password file
    echo "$username,$password" >> $PASSWORD_FILE
    
    # Add the user to additional groups if specified
    if [ -n "$groups" ]; then
        IFS=',' read -r -a group_array <<< "$groups"
        for group in "${group_array[@]}"; do
            group=$(echo "$group" | xargs)
            # Create the group if it doesn't exist
            if ! getent group "$group" >/dev/null 2>&1; then
                groupadd "$group"
            fi
            # Add the user to the group
            usermod -aG "$group" "$username"
        done
    fi
    
    # Log the successful creation of the user and their groups
    log_action "User $username created with groups: $username, $groups"
done < $USER_FILE

# Log the completion of the user creation process
log_action "User creation process completed."
