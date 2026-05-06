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

<img width="1215" height="614" alt="nano script" src="https://github.com/user-attachments/assets/0e8d28c6-1467-4fb5-a9f7-297e9baa15a1" />


<img width="813" height="486" alt="Screenshot 2026-05-05 134923" src="https://github.com/user-attachments/assets/abadfd63-aec3-41f4-9d07-a4492cfccac5" />


<img width="476" height="335" alt="osticket fail" src="https://github.com/user-attachments/assets/700e5315-d1fc-47d6-bee8-88cdf9c972f0" />


Installation Script Problems

The installation script was supposed to:

Install Apache
Install PHP
Install MariaDB
Create the osTicket database
Create the database user
Configure Apache
Download osTicket

However, multiple parts of the script failed during execution.

TROUBLESHOOTING — osTicket Download Failure Issue

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

<img width="477" height="199" alt="wget failure" src="https://github.com/user-attachments/assets/4f95dcd9-0d46-4b2b-82b7-69c380e6ec62" />


<img width="478" height="153" alt="install wget" src="https://github.com/user-attachments/assets/a300ca11-599e-49d6-b831-8608b4ac35c0" />

<img width="477" height="79" alt="curl" src="https://github.com/user-attachments/assets/a7c25f77-5ca6-4986-9a6f-efcf9bb8ec25" />

<img width="479" height="310" alt="wget unpack" src="https://github.com/user-attachments/assets/a30a19e5-b524-4ed2-9f38-4b0ae2498b4b" />


Step 2 — MariaDB Database Configuration

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

<img width="478" height="334" alt="587873019-8bb00973-0346-4b3a-9caf-e3192c8ec454" src="https://github.com/user-attachments/assets/9697cc89-9b92-4f69-aaa1-f4ddb76ed580" />

<img width="482" height="337" alt="587874046-7690134a-73e1-48c0-b441-7843551c64cf" src="https://github.com/user-attachments/assets/2d0b6f7d-bcfc-4a6f-944f-54eb86fc670f" />


Step 4 — osTicket Installer Authentication Failure

Accessed:

http://localhost/osticket/setup/install.php

During installation, osTicket returned:

Access denied for user 'osticket'@'localhost'




TROUBLESHOOTING — Authentication Failure
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

<img width="456" height="341" alt="sqlfatalerror" src="https://github.com/user-attachments/assets/54a4cf90-f568-46fd-b5b0-041c17c4e38d" />


<img width="475" height="141" alt="successful sql login" src="https://github.com/user-attachments/assets/9335d601-9e26-41af-8b6e-c601ed266935" />


<img width="456" height="341" alt="osticket install" src="https://github.com/user-attachments/assets/b0511c01-31d2-4cbe-84d4-b6d7f701a381" />



Step 5 — Post-Install Security

Removed setup directory:

sudo rm -rf /var/www/html/osticket/setup

Locked configuration file:

chmod 644 /var/www/html/osticket/include/ost-config.php
Why
prevents accidental reinstallation
reduces attack surface
protects configuration settings

<img width="455" height="340" alt="Configuration file missing " src="https://github.com/user-attachments/assets/7aec6616-6e15-458e-8533-c73610caf7c5" />


<img width="479" height="106" alt="var" src="https://github.com/user-attachments/assets/5b73747f-46ad-4caa-94d9-ea8718249aae" />


Step 6 — Departments & Staff

Created departments:

Support
Billing

Configured staff users inside the admin panel.

<img width="736" height="371" alt="supportmanjohn" src="https://github.com/user-attachments/assets/3e4a73b2-4a1f-4f73-944d-80863b42246a" />


<img width="797" height="422" alt="sarahbilling" src="https://github.com/user-attachments/assets/54cfb3fb-6263-43ad-814a-46504c2434f7" />



Step 7 — SLA Policies

Created SLA policies:

SLA	Grace Period
Critical Issues	1 Hour
Standard Issues	4 Hours


TROUBLESHOOTING — SLA UI Differences Issue

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



<img width="563" height="374" alt="sla" src="https://github.com/user-attachments/assets/ec305e8d-84b9-46db-9824-140cddec9cbb" />



Step 8 — Custom Ticket Fields

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


TROUBLESHOOTING — Custom Field Validation
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

<img width="577" height="321" alt="field error" src="https://github.com/user-attachments/assets/41a07f33-79ff-4e53-8485-a3ff69d4ee90" />


<img width="379" height="285" alt="Field config" src="https://github.com/user-attachments/assets/fc3359b8-b89c-4cf4-909d-7aa05bbdb6bc" />


<img width="577" height="388" alt="sucessful form update" src="https://github.com/user-attachments/assets/a22974c7-635c-4e22-91ea-f394efba1cd3" />


Step 9 — Ticket Workflow Testing

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

<img width="489" height="374" alt="ghost copies" src="https://github.com/user-attachments/assets/3d9449d6-0218-47aa-81fa-4c7f7c5f9bf0" />

<img width="577" height="388" alt="sucessful form update" src="https://github.com/user-attachments/assets/f5b8e06f-f68a-4fcf-8f89-1090b3b1beeb" />


Step 10 — Ticket Resolution Workflow

Logged into admin panel and:

opened ticket
added internal note
replied to user
closed ticket

Internal note:

Checked printer and cleared paper jam.

User response:

Printer issue resolved. Device operational.

<img width="739" height="369" alt="ticket que" src="https://github.com/user-attachments/assets/ae75256e-cac1-48df-91b4-5cfbfa33ec23" />


<img width="563" height="352" alt="6ternal note" src="https://github.com/user-attachments/assets/10b14052-9ae1-49a2-b9ca-0aaae44f8661" />


<img width="564" height="179" alt="ticket closed" src="https://github.com/user-attachments/assets/e3963117-a214-48f0-9756-be29efb73753" />



<img width="737" height="378" alt="emailerroe" src="https://github.com/user-attachments/assets/d06e6ccc-94da-4fa4-90a1-0e11da9c9463" />


Additional Observation — Email Errors

Observed mailer errors inside system logs.

Cause

Email functionality was not configured during Phase 1.

Decision

Deferred email integration to a later phase.

Lesson

Not all system errors are blockers. Prioritize core functionality first.

Skills Demonstrated


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


Key Takeaways
Most failures are configuration-related
Scripts should always be verified manually
SQL errors require contextual interpretation
Authentication is a common deployment failure point
Troubleshooting is the real technical skill



Final Result

Successfully deployed and configured a fully functional osTicket help desk environment with:

working database connectivity
functional ticket workflows
SLA policies
custom ticket fields
admin/user separation
completed incident lifecycle testing
