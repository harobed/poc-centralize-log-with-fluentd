# Centralize log with fluentd

Proof of concept project to test log centralization with Fluentd:

* `server-log` use [fluentd](https://docs.fluentd.org/v1.0/articles/quickstart) server
* `client-log` use [fluentbit](http://fluentbit.io/documentation/current/) agent

All logs are centralized in json files on the server.

## Prerequisite

```
$ brew cask install virtualbox
$ vagrant plugin install vagrant-hostmanager
```

## Installation

Start `server-log` and `client-log`:

```
$ vagrant up
```

### Install and configure fluentd on server-log

```
$ vagrant ssh server-log
$ sudo su
# curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-xenial-td-agent3.sh | sh
```

```
# cat <<EOF > /etc/td-agent/td-agent.conf
<source>
  @type http
  @id input_http
  port 8888
</source>

<match **>
  @type file
  path /tmp/foobar.log
  flush_interval 1s
  append true
  <format>
    @type out_file
  </format>
</match>
EOF
# systemctl enable td-agent
# systemctl start td-agent
```

Display log content:

```
# tail -f /tmp/foobar.log.20180112.log
2018-01-12T09:24:39+00:00	fluent.info	{"worker":0,"message":"fluentd worker is now running worker=0"}
```

### Install and configure fluentd on server-log

[fluentbit](http://fluentbit.io/documentation/current/) installation:

```
$ vagrant ssh client-log
$ sudo su
# wget -qO - http://packages.fluentbit.io/fluentbit.key | sudo apt-key add -
# echo "deb http://packages.fluentbit.io/ubuntu xenial main" > /etc/apt/sources.list.d/fluentbit.list
# apt-get update
# apt-get install td-agent-bit
```

fluentbit configuration:

```
# cat <<EOF > /etc/td-agent-bit/td-agent-bit.conf
[SERVICE]
    Flush        5
    Daemon       Off
    Log_Level    info
    Parsers_File parsers.conf

[INPUT]
    Name systemd
    Systemd_Filter  SYSLOG_IDENTIFIER=foobar

[OUTPUT]
    Name file
    Match *
    Path /tmp/test-output.txt

[OUTPUT]
    Name http
    Match *
    Host server-log
    Port 8888
    URI /debug.test
    Format json
EOF
# systemctl enable td-agent-bit
# systemctl start td-agent-bit
```

## Send text to log

Send log message in `client-log`:

```
$ vagrant ssh client-log
$ sudo su
# echo 'hello' | systemd-cat -t foobar
```

Read this message on `server-log`:

```
$ vagrant ssh server-log
$ sudo su
# tail -f /tmp/foobar.log.20180112.log
2018-01-12T09:52:48+00:00	debug.test	{"date":1515750767.532173,"PRIORITY":"6","_UID":"0","_GID":"0","_CAP_EFFECTIVE":"3fffffffff","_SYSTEMD_SLICE":"-.slice","_BOOT_ID":"f4a3b91f81aa40faa2d2fdade88f4ea7","_MACHINE_ID":"73eeabe4d35f4c54af1cd88ffe22e352","_AUDIT_LOGINUID":"1000","_TRANSPORT":"stdout","_SYSTEMD_CGROUP":"/","_HOSTNAME":"client-log","_AUDIT_SESSION":"3","SYSLOG_IDENTIFIER":"foobar","MESSAGE":"hello","_PID":"2582","_COMM":"cat"}
```
