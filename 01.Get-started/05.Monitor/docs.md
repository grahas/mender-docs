---
title: Monitor your device
taxonomy:
    category: docs
    label: tutorial
---

!!!!! Requires the Mender Monitor add-on package.
!!!!! See [the Mender features page](https://mender.io/plans/features?target=_blank)
!!!!! for an overview of all Mender plans and features.

This tutorial will walk you through how to monitor your device and its application with
Mender. We will be using the [Monitor Add-on](../../09.Add-ons/20.Monitor/docs.md) which
allows you to monitor both services and log files.

## Prerequisites

To follow this tutorial, you will need to install the
[Monitor Add-on](../../09.Add-ons/20.Monitor/10.Installation/docs.md) on your
device.

## Check if the mender-connect systemd service is running

Assume you want to monitor the state of the `mender-connect` systemd service,
and you want to receive _CRITICAL_ alerts if the service is not running,
and _OK_ alerts when it is back up. To get this working, we need to create
a systemd service checker using `mender-monitorctl`:

```bash
mender-monitorctl create service mender-connect systemd
```

This command creates a file in `/etc/mender-monitor/monitor.d/available`
with the details of the server name and type to check:

```bash
cat /etc/mender-monitor/monitor.d/available/service_mender-connect.sh
```
> ```bash
> # This file was autogenerated by Monitoring Utilities based on the configuration
> SERVICE_NAME="mender-connect"
> SERVICE_TYPE="systemd"
> ```

You can now enable the check running:

```bash
mender-monitorctl enable service mender-connect
```

This command links the file in `/etc/mender-monitor/monitor.d/available` to
`/etc/mender-monitor/monitor.d/enabled`, as you can verify running:

```bash
readlink /etc/mender-monitor/monitor.d/enabled/service_mender-connect.sh
```
> ```bash
> /etc/mender-monitor/monitor.d/available/service_mender-connect.sh
> ```

The `mender-monitor` daemon will automatically reload the configuration files and start the checks.

## Receive alerts on new sessions for the root user

To receive alerts on new sessions on the device for the root user, we can check for
the pattern `Started User Manager for UID 0` in the `/var/log/auth.log` file, defining
a new check named `auth_root_session`:

```bash
mender-monitorctl create log auth_root_session "Started User Manager for UID 0" /var/log/auth.log
```

This command creates a file in `/etc/mender-monitor/monitor.d/available`
with the details of the log file and pattern to check:

```bash
cat /etc/mender-monitor/monitor.d/available/log_auth_root_session.sh
```
> ```bash
> # This file was autogenerated by Monitoring Utilities based on the configuration
> SERVICE_NAME="auth_root_session"
> LOG_PATTERN="Started User Manager for UID 0"
> LOG_FILE="/var/log/auth.log"
> ```

You can now enable the check running:

```bash
mender-monitorctl enable log auth_root_session
```

This command links the file in `/etc/mender-monitor/monitor.d/available` to
`/etc/mender-monitor/monitor.d/enabled`, as you can verify running:

```bash
readlink /etc/mender-monitor/monitor.d/enabled/log_auth_root_session.sh
```
> ```bash
> /etc/mender-monitor/monitor.d/available/log_auth_root_session.sh
> ```

The `mender-monitor` daemon will automatically reload the configuration files and start the checks.

!!! You can also do Perl-compatible regular expressions (PCRE) pattern matching for the UID to catch users other than root, using for instance:
!!! `LOG_PATTERN='Started User Manager for UID \d+'`
!!! If your device does not support PCRE, it falls back to -E if available or plain grep if not.
