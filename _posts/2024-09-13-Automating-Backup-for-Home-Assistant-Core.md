---
layout: post
title: "Automating Backup for Home Assistant Core"
date: 2024-09-13 10:20:00 -0500
categories: homeassistant docker-compose backup docker
tags: homeassistant docker docker-compose backup homelab
image:
  path: /assets/img/headers/homeassistant.webp
---

## Backup your configs!
I’ve developed a custom script that is scheduled to run automatically every morning at 3 AM using cron. This script is designed to back up my Home Assistant configuration and manage those backups efficiently. It ensures that I always have access to the last 7 days of daily backups, providing peace of mind for minor issues or recent changes that need to be reversed. In addition to the daily snapshots, the script also creates and retains one monthly backup for each of the past 12 months. This way, I have a reliable archive of my configurations over the long term, which is helpful for tracking changes or restoring specific configurations after significant updates.

Beyond the daily and monthly backups, the script goes a step further by maintaining an annual backup. Once a year, on January 1st, the script saves a snapshot that will be kept for 7 years, ensuring that I have a long-term record of my Home Assistant setup. This is especially important for more extensive or complex automations that evolve over time. By keeping yearly snapshots, I can review my entire home automation journey, restore older versions of configurations, or recover from any unforeseen disasters, even years later.

This backup strategy is designed with both flexibility and longevity in mind. It covers short-term issues through daily backups, offers medium-term recovery options with monthly snapshots, and provides long-term restoration capabilities through the annual archives. By automating this process, I can rest easy knowing my configurations are secure, recoverable, and easily accessible—no matter how far back I need to go.


## The Script

I've placed this right at the root folder that holds my other Docker services. Later, we'll implement rsync to take these files off of my main produciton machine and store them safely in backup.

``` bash
#!/bin/bash

###
# 13 Sep 2024
# Ryan McCawley
# Crontab:
# 0 3 * * * /path/to/backup_homeassistant.sh
###

# Directories
CONFIG_DIR="./homeassistant/config"  # Change to your Home Assistant config directory
BACKUP_DIR="./backup"  # Change to your desired backup directory

# Get current date details
TODAY=$(date +'%Y%m%d')
DAY_OF_MONTH=$(date +'%d')
MONTH=$(date +'%m')
YEAR=$(date +'%Y')

# Backup file names
BACKUP_FILE="$BACKUP_DIR/homeassistant_$TODAY.tar.gz"
MONTHLY_BACKUP_FILE="$BACKUP_DIR/homeassistant_monthly_$YEAR$MONTH.tar.gz"
ANNUAL_BACKUP_FILE="$BACKUP_DIR/homeassistant_annual_$YEAR.tar.gz"

# Create a new backup
tar -czvf "$BACKUP_FILE" -C "$CONFIG_DIR" .

# Logic for keeping specific backups

# 1. Keep last 7 daily backups
find "$BACKUP_DIR" -type f -name "homeassistant_*.tar.gz" -mtime +7 -exec rm {} \;

# 2. If today is the 1st, save a monthly backup
if [ "$DAY_OF_MONTH" -eq "01" ]; then
    cp "$BACKUP_FILE" "$MONTHLY_BACKUP_FILE"
    # Delete monthly backups older than 12 months
    find "$BACKUP_DIR" -type f -name "homeassistant_monthly_*.tar.gz" -mtime +365 -exec rm {} \;
fi

# 3. If today is Jan 1st, save an annual backup
if [ "$DAY_OF_MONTH" -eq "01" ] && [ "$MONTH" -eq "01" ]; then
    cp "$BACKUP_FILE" "$ANNUAL_BACKUP_FILE"
    # Delete annual backups older than 7 years
    find "$BACKUP_DIR" -type f -name "homeassistant_annual_*.tar.gz" -mtime +2555 -exec rm {} \;
fi
```

## Backup Strategy
Once you’ve spent time fine-tuning your Home Assistant setup—customizing automations, integrating third-party services, and configuring devices to suit your household’s needs—it’s critical to ensure that these configurations are automatically backed up. Manually backing up configurations can be time-consuming and is often forgotten, but automating the process ensures that you’re never left scrambling to restore your system in the event of a failure. Home Assistant Core configurations, including your YAML files, secrets, and automations, represent hours of work that would be difficult to reconstruct from memory.

Additionally, Home Assistant handles complex integrations with IoT devices, and as your setup grows, the potential for data corruption, device failures, or user errors increases. By setting up a cron job to back up your configurations daily, you create a safety net. Should anything go wrong—whether due to a system crash, failed update, or misconfiguration—you’ll always have recent copies to restore your working environment without significant downtime.

A reliable backup system not only preserves your configurations but also helps protect your home automation system against gradual changes over time. With the right schedule, you can store daily, monthly, and even annual snapshots, ensuring you can roll back to a working state at any point, even after large system upgrades. This allows you to maintain a stable environment while experimenting with new integrations, ensuring your setup is resilient against failures or unforeseen issues.​