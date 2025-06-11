# The Self Hosted Server Survival Guide - Manuals

- **[Categories](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md)**
  1. **[Getting Started with Linux](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/000_Getting_Started_with_Linux)**
     - [Choosing and Installing Your First Linux Distribution](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/000_Getting_Started_with_Linux/001_choosing_and_installing_your_first_linux_distro.md)
     - [Getting Started with the Linux CLI](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/000_Getting_Started_with_Linux/002_getting_started_with_the_linux_cli.md)
     - [Understanding the Linux Filesystem Hierarchy](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/000_Getting_Started_with_Linux/003_understanding_linux_filesystem_hierarchy.md)
     - [Editing Files with nano](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/000_Getting_Started_with_Linux/004_editing_files_with_nano.md)
     - [Basic vim Commands](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/000_Getting_Started_with_Linux/005_basic_vim_commands_for_sysadmins.md)
  2. **[Core System Management](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/001_Core_System_Management)**
     - [Managing Processes with PS, Top, and Kill](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/001_Core_System_Management/001_managing_processes_with_ps_top_kill.md)
     - [Understanding systemd Basics](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/001_Core_System_Management/002_understanding_systemd_basics.md)
     - [Scheduling Tasks With Cron](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/001_Core_System_Management/003_scheduling-tasks-with-cron.md)
     - [Setting Up a Chroot Environment](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/001_Core_System_Management/004_setting-up-a-chroot-environment.md)
  3. **[Managing Users and Groups](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/002_Managing_Users_and_Groups)**
     - [Best Practices for User Account Management](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/002_Managing_Users_and_Groups/001_best_practices_for_user_and_account_management.md)
     - [Understanding and Using Sudo](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/002_Managing_Users_and_Groups/002_understanding_and_using_sudo.md)
     - [Implementing Role-Based Access Control (RBAC)](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/002_Managing_Users_and_Groups/003_implementing_role_based_access_control.md)
  4. **[Data Management and Storage](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/003_Data_Management_and_Storage)**
     - **[Storage Management](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/003_Data_Management_and_Storage/003_000_Storage_Management)**
        - [Understanding Disk Partitioning](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/003_Data_Management_and_Storage/003_000_Storage_Management/001_understanding_disk_partitioning.md)
        - [Mounting and Unmounting Filesystems](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/003_Data_Management_and_Storage/003_000_Storage_Management/002_mounting_and_unmounting_filesystems.md)
        - [Managing Logical Volumes with LVM](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/003_Data_Management_and_Storage/003_000_Storage_Management/003_managing_logical_volumes_with_lvm.md)
     - **[Backup Strategies](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/003_Data_Management_and_Storage/003_001_Backup_Strategies)**
        - [Setting Up Local Backups with rsync](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/003_Data_Management_and_Storage/003_001_Backup_Strategies/001_setting_up_local_backups_with_rsync.md)
        - [Automating Backups with Systemd Timers](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/003_Data_Management_and_Storage/003_001_Backup_Strategies/002_automating_backups_with_systemd_timers.md)
        - [Configuring Remote Backups](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/003_Data_Management_and_Storage/003_001_Backup_Strategies/003_configuring_remote_backups.md)
  5. **[Networking Fundamentals](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/004_Networking_Fundamentals)**
      - **[IP Addressing](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/004_Networking_Fundamentals/000_IP_Addressing)**
        - [Configuring Static IP Addresses On Your Linux System](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/004_Networking_Fundamentals/000_IP_Addressing/001_configuring_static_ip_addresses.md)
      - **[Interfaces](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/004_Networking_Fundamentals/001_Interfaces)**
        - [Setting Up a Network Bridge](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/004_Networking_Fundamentals/001_Interfaces/001_network_bridging.md)
      - **[Protocols](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/004_Networking_Fundamentals/003_Protocols)**
        - **[SSH](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/004_Networking_Fundamentals/003_Protocols/000_SSH)**
          - [Using SSH On Ubuntu](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/004_Networking_Fundamentals/003_Protocols/000_SSH/001_using_ssh_on_ubuntu.md)
          - [Restrict SSH Access with TCP Wrappers](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/004_Networking_Fundamentals/003_Protocols/000_SSH/002_restrict_ssh_access_with_tcp_wrappers.md)
  6. **[System Monitoring and Logging](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/005_System_Monitoring_and_Logging)**
      - **[System Monitoring](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/005_System_Monitoring_and_Logging/000_System_Monitoring)**
        - [Using top, htop, vmstat to Monitor System Resources](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/005_System_Monitoring_and_Logging/000_System_Monitoring/001_using_top_htop_and_vmstat_to_monitor_system_resources.md)
        - [Identifying Active Connections On Specific Ports](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/005_System_Monitoring_and_Logging/000_System_Monitoring/002_identifying_active_connections_on_specific_ports.md)
        - [Installing and Configuring Netdata](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/005_System_Monitoring_and_Logging/000_System_Monitoring/003_installing_and_configuring_netdata.md)
      - **[Log Management](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/005_System_Monitoring_and_Logging/001_Log_Management)**
        - [Understanding Linux System Logs](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/005_System_Monitoring_and_Logging/001_Log_Management/001_understanding_linux_system_logs.md)
        - [Configuring logrotate](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/005_System_Monitoring_and_Logging/001_Log_Management/002_configuring_logrotate.md)
  7. **[Security Hardening](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/006_Security_Hardening-Initial_Steps)**
      - **[Firewalls](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/006_Security_Hardening-Initial_Steps/000_Firewalls)**
        - [Configuring ufw for Common Services](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/006_Security_Hardening-Initial_Steps/000_Firewalls/001_configuring_ufw_for_common_services.md)
        - [Setting Up firewalld](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/006_Security_Hardening-Initial_Steps/000_Firewalls/002_setting_up_firewalld.md)
        - [Advanced iptables Rules](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/006_Security_Hardening-Initial_Steps/000_Firewalls/003_advanced_iptables_rules.md)
      - **[Data Encryption](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/006_Security_Hardening-Initial_Steps/001_Data_Encryption)**
        - [Using GPG for Secure Encryption and Decryption](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/006_Security_Hardening-Initial_Steps/001_Data_Encryption/001_using_gpg_for_secure_file_encryption_and_decryption.md)
      - **[Secure Certificates](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/006_Security_Hardening-Initial_Steps/002_Secure_Certificates)**
        - [Obtaining and Installing Let's Encrypt Certificates](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/006_Security_Hardening-Initial_Steps/002_Secure_Certificates/001_obtaining_and_installing_lets_encrypt_certificates.md)
        - [Forcing HTTPS on Your Web Server](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/006_Security_Hardening-Initial_Steps/002_Secure_Certificates/002_forcing_https_on_your_web_server.md)
  8. **[Web Services and Database Management](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/007_Web_Services_and_Database_Management)**
      - **[DNS Management](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/007_Web_Services_and_Database_Management/000_DNS_Management)**
        - [Understanding DNS Records](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/000_DNS_Management/001_understanding_dns_records.md)
        - [Setting Up a Local DNS Resolver with unbound](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/000_DNS_Management/002_setting_up_a_local_dns_resolver_with_unbound.md)
      - **[Web Servers](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/007_Web_Services_and_Database_Management/001_Web_Servers)**
        - [Installing and Configuring Apache](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/001_Web_Servers/001_installing_and_configuring_apache.md)
        - [Configuring and Installing Nginx](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/001_Web_Servers/002_setting_up_nginx_as_a_reverse_proxy.md)
        - [Optimizing Your Webserver Performance](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/001_Web_Servers/003_optimizing_your_web_server_performance.md)
      - **[Email Servers](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/007_Web_Services_and_Database_Management/002_Email_Servers)**
        - [Introduction to Mail Server Components: SMPT, IMAP, POP3](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/002_Email_Servers/001_introduction_to_mail_server_components_smtp_imap_pop3.md)
      - **[Database Management](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/007_Web_Services_and_Database_Management/003_Database_Management)**
        - [Installing and Securing MySQL and MariaDB](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/003_Database_Management/001_installing_and_securing_mysql_mariadb.md)
        - [Installing, Setting Up and Securing PostgreSQL](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/003_Database_Management/002_installing_setting_up_and_securing_postgresql.md)
        - [Basic Database Administration Tasks](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/007_Web_Services_and_Database_Management/003_Database_Management/003_basic_database_administration_tasks.md)
  9. **[Virtualization and Containerization](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/008_Virtualization_and_Containerization)**
      - **[Virtual Machines](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/008_Virtualization_and_Containerization/000_Virtual_Machines)**
        - [Using KVM](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/008_Virtualization_and_Containerization/000_Virtual_Machines/001_using_kvm.md)
      - **[Containerization](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization)**
        - **[Docker](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization/000_Docker)**
          - [Introduction to Docker Concepts](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization/000_Docker/001_introduction_to_docker_concepts.md)
          - [Building and Managing Docker Containers](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization/000_Docker/002_building_and_managing_docker_containers.md)
          - [Using Docker Compose](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization/000_Docker/003_using_docker_compose.md)
        - **[Podman](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization/001_Podman)**
          - [Getting Started with Podman](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/008_Virtualization_and_Containerization/001_Containerization/001_Podman/001_getting_started_with_podman.md)
  10. **[Advanced Security and Troubleshooting](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/009_Advanced_Security_and_Troubleshooting)**
      - **[Intrusion Detection and Prevention](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/000_Intrusion_Detection_and_Prevention)**
        - [Installing and Configuring Fail2ban](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/000_Intrusion_Detection_and_Prevention/001_installing_and_configuring_fail2ban.md)
        - [Introduction to Intrusion Detection Systems (IDS) Using Snort](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/000_Intrusion_Detection_and_Prevention/002_introduction_to_intrusion_detection_systems_with_snort.md)
      - **[Troubleshooting](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/001_Troubleshooting)**
        - [Common Linux Boot Issues and How to Fix Them](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/001_Troubleshooting/001_common_linux_boot_issues_and_how_to_fix_them.md)
        - [Diagnosing Network Connectivity Problems](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/001_Troubleshooting/002_diagnosing_network_connectivity_problems.md)
        - [Interpreting Error Messages and Logs](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/009_Advanced_Security_and_Troubleshooting/001_Troubleshooting/003_interpreting_error_messages_and_logs.md)
  11. **[Automation and Scripting](https://github.com/paradoxresearch/SHSSG/tree/main/Manuals/md/010_Automation_and_Scripting/000_Bash_Scripting)**
      - **[Bash Scripting](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/010_Automation_and_Scripting/000_Bash_Scripting)**
        - [Introduction to Bash Scripting](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/010_Automation_and_Scripting/000_Bash_Scripting/001_introduction_to_bash_scripting.md)
        - [Automating System Tasks With Bash Scripts](https://github.com/paradoxresearch/SHSSG/blob/main/Manuals/md/010_Automation_and_Scripting/000_Bash_Scripting/002_automating_system_tasks_with_bash_scripts.md)
