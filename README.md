#!/bin/bash

SERVERS_FILE="servers.txt"
REPORT="/tmp/server_health_report.txt"
DATE=$(date)

# Email settings (Outlook SMTP)
MAIL_TO="yourmail@outlook.com"
SMTP_SERVER="smtp.office365.com"
SMTP_PORT="587"
SMTP_USER="yourmail@outlook.com"
SMTP_PASS="YOUR_APP_PASSWORD"

echo "Server Health Report" > $REPORT
echo "Generated on: $DATE" >> $REPORT
echo "===================================" >> $REPORT

while read -r SERVER; do
    [[ -z "$SERVER" || "$SERVER" =~ ^# ]] && continue

    echo "" >> $REPORT
    echo "Server: $SERVER" >> $REPORT
    echo "-----------------------------------" >> $REPORT

    ssh -o ConnectTimeout=10 $SERVER << 'EOF' >> $REPORT 2>&1
echo "CPU Usage:"
top -bn1 | grep "Cpu(s)" | awk '{print "Used: " 100-$8 "%"}'

echo ""
echo "Memory Usage:"
free -m | awk 'NR==2{printf "Used: %sMB / Total: %sMB (%.2f%%)\n", $3,$2,$3*100/$2}'

echo ""
echo "Disk Usage:"
df -h | grep -E '^/dev/' | awk '{print $1 ": " $5 " used (" $6 ")"}'
EOF

    if [ $? -ne 0 ]; then
        echo "ERROR: Unable to connect to $SERVER" >> $REPORT
    fi

done < "$SERVERS_FILE"
