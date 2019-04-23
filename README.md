# Qubes-LogVM
This is a Proof of Concent project, implementing a central log collector (LogVM) for Qubes OS.

## Preparation
Read the [docuemntation](https://www.qubes-os.org/doc/qrexec3/) about qrexec

### dom0

- Create a LogVM 

  Choose a nice name like: "sys-log" :)
  
  (2vCPU, 256/512 RAM, and a reasonable amount of private storage)

- Qubes RPC

  Create a qubes-rcp action:
  
  /etc/qubes-rpc/qubes.LogCollect
  ```
  /usr/local/bin/LogCollector
  ```
  
  and it's policy:
  
  /etc/qubes-rpc/policy/qubes.LogCollect
  ```
  @anyvm	sys-log	allow
  ```

### Log Colletor (sys-log)
  
- create a minimalistic LogCollector script: `/usr/local/bin/LogCollector`
  ```
  #!/bin/bash
  cat >> /tmp/syslog
  ```
- check if it's working:
  ```
  tail -f /tmp/syslog
  ```

**WARNING**: this is just a simple exaple, to see if the solution is working.

If it is, You will receive all the logs from all the VM's into a single file. But that file will be lost if you reboot your LogVM.

**TODO**: 
  - Create a basic working solution, with permanent log storage.
  - Create a more advanced soluton by passing the logs to syslog-ng
  
  
### Log Source
  
The log source can be any VM (qube) inside your Qubes OS. In case of Debian/Fedora, the default system logger is the `systemd-journald` service. Journald is just a workaround for the desktop users, and it is not prepared for sending logs to a central log collector, so we need a real log manager application: 
  
- install the 'syslog-ng' package (and it's dependecies)
- create a [config file](syslog-ng.conf) for syslog-ng
- modify the journald configuration: /etc/systemd/journald.conf
  ```
  [Journal]
  #Storage=auto
  Storage=volatile
  ```
- restart the systemd-journald service
- (re)start the syslog-ng service

**WARNING**: If you do the modification in a running AppVM, you will loose all after rebooting the VM. If you do the modification in a template, all your VM's (based on that template) will be affected.


After this stage, you should see all the logs in sys-log VM :)
