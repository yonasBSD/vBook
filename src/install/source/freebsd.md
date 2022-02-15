# FreeBSD installation

The installation described above was performed on an FreeBSD 13.0 server.

## Installing RUST language

Rust port, packages and information can be found on [freshports] website. You can find more information about packages and port in the [FreeBSD handbook].

[freshports]: https://www.freshports.org/lang/rust/
[FreeBSD handbook]: https://docs.freebsd.org/en/books/handbook/

```shell
pkg install lang/rust
```

> You may have to switch to the latest ports branch. Please refer to [freeBSD wiki]. 
[freeBSD wiki]: https://wiki.freebsd.org/Ports/QuarterlyBranch

## Dependencies

FreeBSD 13.x comes with all required dependencies.

## vSMTP compilation

```shell
git clone https://github.com/viridIT/vSMTP.git
cargo build --release
cargo run -V
```

## Configuring the Operating System for vSMTP

Create the directories and change the owner and group.

```shell
mkdir /etc/vsmtp /etc/vsmtp/rules /etc/vsmtp/certs /var/log/vsmtp /var/spool/vsmtp
```

Copy the vsmtp binaries and the config files.

```shell
cp /home/freebsd/vSMTP//target/release/vsmtp /usr/sbin/
chmod 555 /usr/sbin/vsmtp
cp -p ./config/vsmtp.default.toml /etc/vsmtp/vsmtp.toml
```

### Disabling sendmail

Sendmail may have been disabled during FreeBSD install. If not, add the following in the /etc/rc.conf file and reboot the system.

```shell
sendmail_enable="NO"
sendmail_submit_enable="NO"
sendmail_outbound_enable="NO"
sendmail_msp_queue_enable="NO"
```

Use sockstat command to check that sendmail is disabled.

```shell
# sockstat -4l
USER     COMMAND    PID   FD PROTO  LOCAL ADDRESS         FOREIGN ADDRESS
root     sshd       1053  4  tcp4   *:22                  *:*
root     syslogd    957   7  udp4   *:514                 *:*
```


### Starting with a non privileged user

> Version 0.10 will come with a mechanism to drop privileges at vSMTP startup.
> On current version (0.9) the vsmtp user must have the rights to bind to ports <1024.

The [kernel] must be updated to support network [ACL]. Add to these options to the KERNEL file and rebuild it.

[kernel]: https://docs.freebsd.org/en/books/handbook/kernelconfig/
[ACL]: https://docs.freebsd.org/en/books/handbook/mac/

``` 
options MAC
options MAC_PORTACL
```

```
cd /usr/src
make buildkernel KERNCONF=MYKERNEL
make installkernel KERNCONF=MYKERNEL
```

```shell
$ sysctl security.mac
security.mac.portacl.rules:
security.mac.portacl.port_high: 1023
security.mac.portacl.autoport_exempt: 1
security.mac.portacl.suser_exempt: 1
security.mac.portacl.enabled: 1
security.mac.mmap_revocation_via_cow: 0
security.mac.mmap_revocation: 1
security.mac.labeled: 0
security.mac.max_slots: 4
security.mac.version: 4
```

#### Add vSMTP user:group

```shell
pw groupadd vsmtp -g 999
pw useradd vsmtp -u 999 -d /noexistent -g vsmtp -s /sbin/nologin
chown -R vsmtp:vsmtp /var/log/vsmtp /etc/vsmtp/* /var/spool/vsmtp
```

```shell
$ sysctl security.mac.portacl.rules=uid:999:tcp:25,uid:999:tcp:587,uid:999:tcp:465
security.mac.portacl.rules: uid:999:tcp:25, -> uid:999:tcp:25,uid:999:tcp:587,uid:999:tcp:465
$ sysctl security.mac.portacl.rules
security.mac.portacl.rules: uid:999:tcp:25,uid:999:tcp:587,uid:999:tcp:465
$ sysctl net.inet.ip.portrange.reservedlow=0
net.inet.ip.portrange.reservedlow: 0 -> 0
$ sysctl net.inet.ip.portrange.reservedhigh=0
net.inet.ip.portrange.reservedhigh: 1023 -> 0
```

vSMTP user should now be enable to bind on standard SMTP ports (25, 587, 465).

## Adding a vSMTP as a system service

This feature has not been tested. 
Please note that version 0.10 will add a new startup mechanism that no longer require user ACLs.