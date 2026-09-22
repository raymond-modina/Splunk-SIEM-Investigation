# Splunk SIEM Investigation — Ubuntu Linux Log Monitoring with Splunk Enterprise

## 1. Project Overview

This project demonstrates the setup of a basic Security Operations Center (SOC) monitoring environment using:

- **Ubuntu 24.04.4 LTS** as the monitored Linux endpoint
- **Splunk Universal Forwarder 10.4.3** installed on Ubuntu
- **Splunk Enterprise 10.2.2** running on a Windows host
- **Splunk Free license** on the Windows Splunk installation
- TCP port **9997** for Splunk log forwarding
- TCP port **8000** for Splunk Web

The objective was to collect Ubuntu authentication and system logs and forward them to Splunk Enterprise for centralized security monitoring and investigation.

---

## 2. Lab Architecture

```text
                    SOC HOME LAB

┌───────────────────────────────────────┐
│           Ubuntu VM                   │
│                                       │
│ Ubuntu 24.04.4 LTS                    │
│ IP: 192.168.xxx.xxx                   │
│                                       │
│ Splunk Universal Forwarder 10.4.3     │
│                                       │
│ Logs:                                 │
│  /var/log/auth.log                    │
│  /var/log/syslog                      │
└──────────────────┬────────────────────┘
                   │
                   │ TCP 9997
                   │ Splunk Forwarder
                   ▼
┌───────────────────────────────────────┐
│          Windows Host                 │
│                                       │
│ IP: 192.168.1.120                     │
│                                       │
│ Splunk Enterprise 10.2.2              │
│ Splunk Web: TCP 8000                  │
│ Receiving: TCP 9997                   │
│ License: Splunk Free                  │
└───────────────────────────────────────┘
```

---

## 3. Windows Splunk Environment

Splunk Enterprise was installed natively on the Windows host.

Installation directory:

```text
C:\Program Files\Splunk
```

Splunk Web:

```text
http://192.168.x.x:8000
```

The Windows host's LAN IP was:

```text
192.168.x.x
```

Splunk was confirmed to be listening on port 8000:

```text
TCP 0.0.0.0:8000
```

This confirmed that Splunk Web was listening on the available Windows network interfaces.

---

## 4. Splunk Account Recovery

The Splunk Web administrator credentials had previously been forgotten.

The Splunk installation's password configuration was inspected and the administrator username was identified as:

```text
admin
```

The Splunk administrator password was subsequently reset and access to Splunk Web was restored.

This was separate from the Linux `splunk` account used inside the Ubuntu VM.

---

## 5. Splunk Trial to Free License

The Windows Splunk installation had previously been operating under the Splunk Enterprise trial.

The license group was changed through:

```text
Settings
→ Licensing
→ Change license group
→ Free
```

The Splunk installation was successfully converted to the Free license group while preserving the existing Splunk installation and lab environment.

---

## 6. Initial Ubuntu Environment

The Ubuntu virtual machine was identified as:

```text
Ubuntu 24.04.4 LTS
Codename: noble
Architecture: amd64
```

Ubuntu hostname:

```text
splunk-VMware-Virtual-Platform
```

Linux username:

```text
splunk
```

Ubuntu IP address:

```text
192.168.x.x
```

---

## 7. Ubuntu Sudo Password Recovery

The password for the Ubuntu Linux user `splunk` had been forgotten.

The VM was booted into Ubuntu recovery mode and a root maintenance shell was accessed:

```text
root@splunk-VMware-Virtual-Platform:#
```

The root filesystem was remounted as writable:

```bash
mount -o remount,rw /
```

The password for the Linux `splunk` account was reset:

```bash
passwd splunk
```

The VM was then rebooted:

```bash
reboot
```

The new password was successfully verified using:

```bash
sudo whoami
```

which returned:

```text
root
```

This restored administrative access to the Ubuntu VM.

---

## 8. Verification of Ubuntu Authentication Logs

Ubuntu authentication logs were verified with:

```bash
sudo tail -20 /var/log/auth.log
```

The output showed successful sudo activity, including entries such as:

```text
pam_unix(sudo:session): session opened
```

This confirmed that Ubuntu's authentication logging was functioning correctly.

---

## 9. VMware Network Configuration

The VMware environment was investigated when Ubuntu temporarily lost Internet connectivity.

The VMware Virtual Network Editor showed:

```text
VMnet0
Type: Bridged
External Connection: Auto-bridging
```

```text
VMnet1
Type: Host-only
Subnet: 192.168.x.x
```

```text
VMnet8
Type: NAT
Subnet: 192.168.x.x
DHCP: Enabled
```

Ubuntu's IP:

```text
192.168.x.x
```

was consistent with the VMnet8 NAT subnet.

The Windows VMware virtual adapters were also checked:

```text
VMware Network Adapter VMnet8 — Enabled
VMware Network Adapter VMnet1 — Enabled
```

The VMware NAT Service was also enabled.

---

## 10. Testing Ubuntu → Windows Connectivity

Before configuring log forwarding, connectivity between Ubuntu and Windows was tested.

Splunk Web was accessible from Ubuntu using:

```text
http://192.168.x.x:8000
```

A TCP connectivity test was performed from Ubuntu:

```bash
nc -vz 192.168.x.x 8000
```

Result:

```text
Connection to 192.168.x.x 8000 port [tcp/*] succeeded!
```

This confirmed that Ubuntu could reach Splunk Web on the Windows host.

---

## 11. Configuring Splunk Receiving Port

Splunk Enterprise on Windows was configured to receive forwarded data.

In Splunk Web:

```text
Settings
→ Forwarding and receiving
→ Configure receiving
→ New Receiving Port
```

Port configured:

```text
9997
```

Windows was then checked using:

```powershell
netstat -ano | findstr :9997
```

The result:

```text
TCP    0.0.0.0:9997    0.0.0.0:0    LISTENING
```

confirmed that Splunk was listening for Universal Forwarder connections.

---

## 12. Testing Ubuntu → Splunk Forwarding Port

From Ubuntu:

```bash
nc -vz 192.168.x.x 9997
```

Result:

```text
Connection to 192.168.1.120 9997 port [tcp/*] succeeded!
```

This confirmed that the Ubuntu VM could establish a TCP connection to the Splunk receiving port.

---

## 13. Splunk Universal Forwarder Installation

The Splunk Universal Forwarder was downloaded from Splunk's official website.

Downloaded package:

```text
splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb
```

The package was installed on Ubuntu using:

```bash
sudo dpkg -i splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb
```

The installation completed successfully.

The following messages appeared during installation:

```text
Setting up splunkforwarder (10.4.3) ...
complete
```

Some `find:` messages concerning Python site-packages were displayed, but the package installation itself completed successfully.

---

## 14. Universal Forwarder Verification

The Universal Forwarder installation was verified:

```bash
sudo /opt/splunkforwarder/bin/splunk status
```

Result:

```text
splunkd is running
splunk helpers are running
```

The installed version was verified:

```bash
sudo /opt/splunkforwarder/bin/splunk version
```

Result:

```text
Splunk Universal Forwarder 10.4.3
Build 4174a2deda5d
```

---

## 15. Configuring the Forward Server

The Universal Forwarder was configured to send data to the Windows Splunk Enterprise instance:

```text
192.168.x.x:9997
```

Command used:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.x.xxx:9997
```

The Forwarder reported:

```text
192.168.x.x:9997 forwarded-server already present
```

This confirmed that the forwarding destination was already configured.

---

## 16. Ubuntu Authentication Log Monitoring

The Universal Forwarder was configured to monitor:

```text
/var/log/auth.log
```

This log contains Linux authentication and authorization activity, including events generated by commands such as `sudo`.

The monitor was added using:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

---

## 17. Ubuntu System Log Monitoring

The Universal Forwarder was also configured to monitor:

```text
/var/log/syslog
```

Command:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
```

This provides additional system-level events for centralized monitoring.

---

## 18. Universal Forwarder Restart

After configuring the monitored log sources, the Universal Forwarder was restarted:

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

This ensured the updated forwarding configuration was active.

---

## 19. Generating a Test Security Event

A new sudo event was generated on Ubuntu:

```bash
sudo whoami
```

The resulting authentication activity appeared in:

```text
/var/log/auth.log
```

For example:

```text
pam_unix(sudo:session): session opened
```

This created a fresh event that could be used to verify the end-to-end Splunk pipeline.

---

## 20. Final Verification

The Ubuntu logs were successfully forwarded to Splunk Enterprise.

The completed pipeline is:

```text
Ubuntu Authentication/System Logs
              │
              ▼
/var/log/auth.log
/var/log/syslog
              │
              ▼
Splunk Universal Forwarder 10.4.3
              │
              │ TCP 9997
              ▼
Splunk Enterprise 10.2.2
              │
              ▼
Splunk Search & Reporting
```

The system was successfully tested and confirmed to be working.

---

## 21. Final Lab Inventory

| Component | Version | IP / Port |
|---|---|---|
| Ubuntu | 24.04.4 LTS | 192.168.x.x |
| Splunk Universal Forwarder | 10.4.3 | Ubuntu |
| Windows Host | Windows | 192.168.x.x |
| Splunk Enterprise | 10.2.2 | Windows |
| Splunk Web | — | TCP 8000 |
| Splunk Receiving | — | TCP 9997 |
| Authentication Logs | — | `/var/log/auth.log` |
| System Logs | — | `/var/log/syslog` |
| VMware NAT | — | VMnet8 / 192.168.x.x |
| VMware Host-only | — | VMnet1 / 192.168.x.x |

---

## 22. Skills Demonstrated

This project demonstrates practical experience with:

- Linux administration
- Ubuntu 24.04
- VMware networking
- NAT and virtual networking
- IP addressing and subnetting
- TCP connectivity testing
- Windows networking
- Splunk Enterprise
- Splunk Universal Forwarder
- SIEM architecture
- Centralized log collection
- Authentication log monitoring
- System log monitoring
- Linux `sudo` and permissions
- Password recovery
- Network troubleshooting
- TCP port troubleshooting
- Splunk receiving configuration
- Log ingestion validation

---

## 23. SOC Analyst Relevance

The completed lab demonstrates a simplified version of a real SOC log collection architecture.

An endpoint generates security-relevant events:

```text
Linux Endpoint
      │
      ▼
Authentication/System Logs
      │
      ▼
Log Forwarder
      │
      ▼
SIEM
      │
      ▼
Security Analyst
```

In a production SOC, this type of architecture can be expanded to collect events from many endpoints, servers, network devices, applications, firewalls and security tools.

---

## 24. Project Status

**Status: Completed**

The lab successfully demonstrates:

- Ubuntu endpoint log generation
- Splunk Universal Forwarder installation
- Splunk Forward Server configuration
- TCP 9997 connectivity
- Authentication log collection
- System log collection
- Successful forwarding to Splunk Enterprise
- End-to-end SIEM log ingestion

This project forms the foundation for additional SOC detection, investigation, dashboarding, and incident-response exercises.

