# osTicketing
This project documents the deployment, configuration, and troubleshooting of an osTicket help desk system inside an Ubuntu Linux virtual machine using Apache, MariaDB, and PHP.


Technologies Used
Ubuntu 24.04 LTS
Oracle VirtualBox
Apache2
MariaDB
PHP
osTicket
Bash / Linux Terminal


Environment Setup
Component	Value
Host OS	Windows
Hypervisor	Oracle VirtualBox
Guest OS	Ubuntu 24.04 LTS
RAM	4 GB
Storage	20 GB Dynamic Disk
Network	NAT


Step 0 — System Hardening

Updated Ubuntu and enabled the firewall.

Commands Used
sudo apt update && sudo apt upgrade -y
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status


Purpose
Updated all system packages
Enabled UFW firewall
Allowed HTTP traffic for osTicket web access
Blocked all other incoming traffic by default

<img width="477" height="337" alt="step 0" src="https://github.com/user-attachments/assets/7cd60186-784a-49e0-8072-2672093537a1" />

<img width="477" height="144" alt="step 0 pt 2" src="https://github.com/user-attachments/assets/4f2e427b-a101-4cb8-a800-7a33cb041636" />


<img width="476" height="335" alt="osticket fail" src="https://github.com/user-attachments/assets/31a42a2a-9d05-4967-a978-9ca030c5824f" />
<img width="476" height="335" alt="osticket fail" src="https://github.com/user-attachments/assets/c26576c6-6100-4e5e-b8b8-55cbc1ed95a6" />
Step 1 — osTicket Installation Script
Method Used

Initially attempted to create the installer using:

cat << EOF

This failed due to unreliable clipboard behavior inside VirtualBox.

Switched to:

nano osticket-install.sh

Pasted the script manually, saved it, then ran:

chmod +x osticket-install.sh
sudo bash osticket-install.sh


TROUBLESHOOTING — Script Creation Failure Issue

The original cat << EOF paste method repeatedly failed.

Cause

VirtualBox clipboard behavior was inconsistent during terminal paste operations.

Fix

Used the nano text editor instead.

📸 Screenshot Placeholder

<img width="1215" height="614" alt="nano script" src="https://github.com/user-attachments/assets/0e8d28c6-1467-4fb5-a9f7-297e9baa15a1" />


<img width="813" height="486" alt="Screenshot 2026-05-05 134923" src="https://github.com/user-attachments/assets/abadfd63-aec3-41f4-9d07-a4492cfccac5" />




nano editor with installation script
⚠️ Installation Script Problems

The installation script was supposed to:

Install Apache
Install PHP
Install MariaDB
Create the osTicket database
Create the database user
Configure Apache
Download osTicket

However, multiple parts of the script failed during execution.

🔧 TROUBLESHOOTING — osTicket Download Failure
Issue

The script failed while attempting to download osTicket using:

wget https://github.com/osTicket/osTicket/releases/latest/download/osTicket.zip

The download returned:

ERROR 404: Not Found.
Root Cause

The GitHub release asset URL used in the script/tutorial no longer existed.

This was NOT:

a Linux issue
a networking issue
an Apache issue

The issue was the external GitHub release URL itself.

Troubleshooting Steps

Verified required tools existed:

sudo apt install -y wget unzip curl

Retried the download:

cd /tmp
wget https://github.com/osTicket/osTicket/releases/latest/download/osTicket.zip

Still failed.

Tested with curl:

curl -L -o osTicket.zip https://github.com/osTicket/osTicket/releases/latest/download/osTicket.zip

The resulting file size was only a few bytes, confirming the download was invalid.

Fix

Used the GitHub repository archive instead:

cd /tmp
wget https://github.com/osTicket/osTicket/archive/refs/heads/develop.zip -O osTicket.zip

Extracted the archive:

unzip osTicket.zip

Verified extracted files successfully.

📸 Screenshot Placeholder

wget 404 failure
successful repository download
unzip output


<img width="476" height="335" alt="osticket fail" src="https://github.com/user-attachments/assets/c36b34e6-e575-4eef-87c0-13f040a2ed2f" />
<img width="476" height="335" alt="osticket fail" src="https://github.com/user-attachments/assets/68e7c9ff-d203-41ae-927f-4fd6ceae47f6" />
<img width="476" height="335" alt="osticket fail" src="https://github.com/user-attachments/assets/a7c8bb60-a954-480c-b69b-8a62fb44cc2b" />
⚙️ Step 3 — MariaDB Database Configuration

Opened MariaDB:

sudo mysql

Attempted database configuration:

CREATE DATABASE osticket;
CREATE USER 'osticket'@'localhost' IDENTIFIED BY 'osticket_password';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticket'@'localhost';
FLUSH PRIVILEGES;
🔧 TROUBLESHOOTING — MariaDB Duplicate Object Errors
Errors Encountered
ERROR 1007 (HY000): Can't create database 'osticket'; database exists
ERROR 1396 (HY000): Operation CREATE USER failed for 'osticket'@'localhost'
Root Cause

These errors occurred because:

the database already existed from a previous partial install
the user account already existed
setup commands were re-run during troubleshooting

These were NOT critical failures.

The errors actually confirmed:

the database existed successfully
the user already existed
Resolution

Permissions were re-applied successfully:

GRANT ALL PRIVILEGES ON osticket.* TO 'osticket'@'localhost';
FLUSH PRIVILEGES;

MariaDB returned:

Query OK

📸 Screenshot Placeholder

duplicate database/user errors
successful GRANT + FLUSH PRIVILEGES
⚠️ Step 4 — osTicket Installer Authentication Failure

Accessed:

http://localhost/osticket/setup/install.php

During installation, osTicket returned:

Access denied for user 'osticket'@'localhost'

📸 Screenshot Placeholder

osTicket installer access denied error
🔧 TROUBLESHOOTING — Authentication Failure
Root Cause

Misunderstood the installer prompt.

Initially treated the password field as:

“Create a new password”

Instead of:

“Enter the existing MariaDB user password”

Debug Process

Tested login directly:

mysql -u osticket -p

Login failed initially.

Returned to MariaDB:

sudo mysql

Verified the user:

SELECT User, Host FROM mysql.user WHERE User='osticket';

Reset the password:

ALTER USER 'osticket'@'localhost' IDENTIFIED BY 'osticket_password';
FLUSH PRIVILEGES;

Retested login successfully.

Outcome

Re-entered the correct credentials into the installer:

Field	Value
Database	osticket
User	osticket
Password	osticket_password

Installation completed successfully.

📸 Screenshot Placeholder

failed mysql login
successful mysql login
osTicket install success page
🔐 Step 5 — Post-Install Security

Removed setup directory:

sudo rm -rf /var/www/html/osticket/setup

Locked configuration file:

chmod 644 /var/www/html/osticket/include/ost-config.php
Why
prevents accidental reinstallation
reduces attack surface
protects configuration settings

📸 Screenshot Placeholder

setup directory warning
terminal deletion command
⚙️ Step 6 — Departments & Staff

Created departments:

Support
Billing

Configured staff users inside the admin panel.

📸 Screenshot Placeholder

departments page
staff configuration
⚙️ Step 7 — SLA Policies

Created SLA policies:

SLA	Grace Period
Critical Issues	1 Hour
Standard Issues	4 Hours
🔧 TROUBLESHOOTING — SLA UI Differences
Issue

The UI terminology differed from the tutorial instructions.

The tutorial referenced:

response time
resolution time

The actual osTicket version used:

Grace Period
Fix

Adapted configuration to the actual UI.

Lesson

Real systems rarely match tutorials exactly.

📸 Screenshot Placeholder

SLA configuration page
⚙️ Step 8 — Custom Ticket Fields

Added custom ticket fields:

Priority Level

Options:

Low
Medium
High
Critical
Equipment Type

Options:

Desktop
Laptop
Printer
Network
🔧 TROUBLESHOOTING — Custom Field Validation
Problems Encountered
form would not save
missing variable errors
config confusion
duplicate rows
invalid dropdown formatting
Root Cause

osTicket required:

backend variable names
structured choice formatting

Incorrect format:

Desktop
Laptop
Printer

Correct format:

desktop:Desktop
laptop:Laptop
printer:Printer
network:Network

Required variable name:

equipment_type
Resolution

Configured the field correctly and saved successfully.

📸 Screenshot Placeholder

validation error
correct dropdown config
successful field creation
⚙️ Step 9 — Ticket Workflow Testing

Opened user portal:

http://localhost/osticket/

Created test ticket:

Field	Value
Name	John Doe
Email	john@example.com

Subject	Broken Printer
Equipment Type	Printer

Issue submitted successfully.

📸 Screenshot Placeholder

ticket submission form
ticket confirmation
⚙️ Step 10 — Ticket Resolution Workflow

Logged into admin panel and:

opened ticket
added internal note
replied to user
closed ticket

Internal note:

Checked printer and cleared paper jam.

User response:

Printer issue resolved. Device operational.

📸 Screenshot Placeholder

ticket queue
internal note
closed ticket
⚠️ Additional Observation — Email Errors

Observed mailer errors inside system logs.

Cause

Email functionality was not configured during Phase 1.

Decision

Deferred email integration to a later phase.

Lesson

Not all system errors are blockers. Prioritize core functionality first.

📊 Skills Demonstrated
Linux Administration
package management
firewall configuration
permissions management
Web Application Deployment
Apache configuration
PHP dependency installation
MariaDB integration
Troubleshooting
authentication debugging
SQL error interpretation
deployment recovery
validation troubleshooting
Help Desk Operations
SLA creation
ticket lifecycle management
user communication
incident resolution
🧠 Key Takeaways
Most failures are configuration-related
Scripts should always be verified manually
SQL errors require contextual interpretation
Authentication is a common deployment failure point
Troubleshooting is the real technical skill
🧾 Resume Summary
Deployed and configured osTicket in an Ubuntu VirtualBox environment. Troubleshot MySQL authentication issues, repaired failed installer dependencies, configured SLA policies, created custom ticket fields, and completed full ticket lifecycle testing from submission to resolution.
✅ Final Result

Successfully deployed and configured a fully functional osTicket help desk environment with:

working database connectivity
functional ticket workflows
SLA policies
custom ticket fields
admin/user separation
completed incident lifecycle testing
