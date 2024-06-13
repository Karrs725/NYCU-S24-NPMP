# Project 4

## Table of Contents

- [Project 4](#project-4)
  - [Table of Contents](#table-of-contents)
  - [Basic](#basic)
  - [Channel group](#channel-group)
  - [ACLs](#acls)
  - [FHRP](#fhrp)
  - [GRE tunnel](#gre-tunnel)

## Basic

## Channel group

```text
int ran fa 0/11-12
channel-pro lacp
channel-group 3 mode passive
exit
int port-channel 3
sw m t
sw t a v 10,316,324
```

## ACLs

## FHRP

```text
int vlan 316
standby version 2
standby 0 ip 140.113.16.254
standby 0 priority 200
standby 0 preempt
```

## GRE tunnel

```text
int tunnel 69
ip addr 192.168.88.69 255.255.255.252
tunnel source g 0/0/0
tunnel destination 140.113.69.3
```
