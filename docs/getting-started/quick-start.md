# Quick Start Guide

Get FloofOS configured and operational in minutes.

## Prerequisites

- FloofOS installed on hardware or VM
- Console or SSH access
- Network connectivity information

---

## Step 1: First Login

Login with default credentials:

```
floofos login: floofos
Password: floofos
```

---

## Step 2: Interface Configuration

Accept the first-boot interface wizard:

```
Configure interfaces? [Y/n]: y
```

Enter configuration mode and commit:

```
floofos> conf
floofos(config)# commit
```

---

## Step 3: Change Hostname

```
floofos(config)# set hostname core-router
floofos(config)# commit
```

---

## Step 4: Change Passwords

Change default user password:

```
core-router(config)# create user floofos password YourSecurePassword
core-router(config)# commit
```

Change root password:

```
core-router(config)# create user root password AnotherSecurePassword
core-router(config)# commit
```

---

## Step 5: Configure Interface IP

```
core-router(config)# set interface ip address GE12/0/0 192.0.2.1/24
core-router(config)# set interface state GE12/0/0 up
core-router(config)# commit
```

---

## Step 6: Set Timezone

```
core-router(config)# set system time-zone Europe/London
core-router(config)# commit
```

---

## Step 7: Enable Security

Enable firewall:

```
core-router(config)# set security firewall enable
core-router(config)# commit
```

Verify Fail2ban is active:

```
core-router(config)# show security fail2ban status
```

---

## Step 8: Configure BGP (Optional)

Open BGP configuration:

```
core-router(config)# set protocol bgp
```

Edit configuration, then save (`Ctrl+Q`) and commit.

---

## Step 9: Install to Disk (If Needed)

If running from installation media:

```
core-router(config)# system install
```

Follow the installation wizard prompts.

---

## Verification Commands

```
# System status
show system

# Resource usage
show resource

# Running configuration
show configuration

# Interface status
show interface

# Security status
show security
```

---

## Next Steps

- [Configure BGP](../configuration/bgp.md)
- [Set Up SNMP Monitoring](../monitoring/snmp.md)
- [Create Additional Users](../security/users.md)
- [Review Firewall Rules](../security/firewall.md)
