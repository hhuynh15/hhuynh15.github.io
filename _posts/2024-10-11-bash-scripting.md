---
title: How I Only Spend 30 Minutes a Month to Run My Rust Server
author:
  name: Hieu Huynh
  link: https://github.com/hhuynh15
date: 2024-10-11 09:00:00 -0700
categories: [Linux, Game Servers]
tags: [Bash, Cron, At, Scripting, Automation, Linux, Rust]
image: 
  src: 
  width: 1200
  height: 630
---

# Introduction
---

Running a Rust game server can be a complex and time consuming, but with the power of Bash scripting and Linux scheduling tools like Cron and At, we can automate many aspects of server management. In this post, we'll explore how to use these tools to make running your Rust server easier and more efficient.

# Use Bash to its Fullest Potential
---

Bash is the default command-line interface for most Linux distributions. For Rust server management, it allows us to create scripts for common tasks. Let's start with a simple example:

```bash
#!/bin/bash

# Start the Rust server
start_server() {
    echo "Starting Rust server..."
    /path/to/rustserver start
}

# Stop the Rust server
stop_server() {
    echo "Stopping Rust server..."
    /path/to/rustserver stop
}

# Restart the Rust server
restart_server() {
    stop_server
    sleep 5  # Wait for 5 seconds
    start_server
}

# Check server status
server_status() {
    echo "Checking server status..."
    /path/to/rustserver status
}

# Main script logic
case "$1" in
    start)   start_server ;;
    stop)    stop_server ;;
    restart) restart_server ;;
    status)  server_status ;;
    *)       echo "Usage: $0 {start|stop|restart|status}" ;;
esac
```

Save this as `rust_server.sh`, make it executable with `chmod +x rust_server.sh`, and you can now easily manage your server with commands like `./rust_server.sh start` or `./rust_server.sh restart`.

# Automating Server Tasks with Cron
---

Cron allows us to schedule recurring tasks. Here are some useful automations for a Rust server:

## Regular Server Restarts

To restart your server daily at 4:00 AM:

```bash
0 4 * * * /path/to/rust_server.sh restart
```

Add this line to your crontab by running `crontab -e`.

## Automatic Backups

Create a backup script:

```bash
#!/bin/bash

BACKUP_DIR="/path/to/backups"
SERVER_DIR="/path/to/rust_server"

# Create timestamp
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")

# Stop the server
/path/to/rust_server.sh stop

# Create backup
tar -czf "$BACKUP_DIR/rust_backup_$TIMESTAMP.tar.gz" -C "$SERVER_DIR" .

# Start the server
/path/to/rust_server.sh start

echo "Backup completed: rust_backup_$TIMESTAMP.tar.gz"
```

Save this as `rust_backup.sh` and make it executable. Then, to run it every day at 3:00 AM:

```bash
0 3 * * * /path/to/rust_backup.sh
```

## Server Updates

For automatic updates, create an update script:

```bash
#!/bin/bash

# Stop the server
/path/to/rust_server.sh stop

# Update the server
/path/to/steamcmd +login anonymous +force_install_dir /path/to/rust_server +app_update 258550 validate +quit

# Start the server
/path/to/rust_server.sh start

echo "Server update completed"
```

Save this as `rust_update.sh`, make it executable, and schedule it weekly:

```bash
0 2 * * 1 /path/to/rust_update.sh  # Runs every Monday at 2:00 AM
```

# Using At for One-time Tasks
---

At is useful for scheduling one-time tasks. For example, to schedule a server restart in 2 hours:

```bash
echo "/path/to/rust_server.sh restart" | at now + 2 hours
```

This is handy for scheduling maintenance after notifying players.

# Advanced Automation: Server Monitoring and Alerts
---

Let's create a script to monitor server health and send alerts:

```bash
#!/bin/bash

# Check if server is running
if ! pgrep -f rustserver > /dev/null
then
    echo "Rust server is down! Attempting to restart..."
    /path/to/rust_server.sh restart
    
    # Send alert (replace with your preferred method)
    echo "Rust server was down and has been restarted" | mail -s "Server Alert" your@email.com
fi

# Check server resources
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
MEM_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')

if (( $(echo "$CPU_USAGE > 90" | bc -l) )) || (( $(echo "$MEM_USAGE > 90" | bc -l) ))
then
    echo "High resource usage detected! CPU: $CPU_USAGE%, Memory: $MEM_USAGE%" | mail -s "Server Resource Alert" your@email.com
fi
```

Save this as `monitor_rust.sh`, make it executable, and run it every 5 minutes with cron:

```bash
*/5 * * * * /path/to/monitor_rust.sh
```

# Conclusion
---

By leveraging Bash scripting along with Cron and At, we've automated several key aspects of managing a Rust game server:

1. Basic server management (start, stop, restart)
2. Scheduled restarts
3. Automatic backups
4. Server updates
5. Health monitoring and alerts

These automations can significantly reduce the manual work involved in server management, ensure your server stays up-to-date and running smoothly, and quickly alert you to any issues.

Remember to test your scripts thoroughly in a safe environment before deploying them on a live server. As you become more comfortable with these tools, you can create more complex automations tailored to your specific needs.

Happy server managing!
