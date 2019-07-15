bird [stable]
=========

Requirements
------------
Network should be alredy configured

Example Network Configuration
----------------
host_vars/host.yml
```yaml
  - device: ethXXX
    method: static
    address: X.X.X.1
    netmask: 255.255.255.240

  - device: lo2
    auto: true
    family: inet
    method: loopback
    address: X.X.X.2
    netmask: 255.255.255.255
    pre-up:
      - /sbin/ip link add lo2 type dummy
    up:
      - /sbin/ip link set lo2 address XX:XX:XX:XX:XX:X1
      - /sbin/ip addr add X.X.X.2/32 dev lo2
    down:
      - /sbin/ip addr del X.X.X.2/32 dev lo2
      - /sbin/ip link delete lo2 type dummy

  - device: lo3
    auto: true
    family: inet
    method: loopback
    address: X.X.X.3
    netmask: 255.255.255.255
    pre-up:
      - /sbin/ip link add lo3 type dummy
    up:
      - /sbin/ip link set lo3 address XX:XX:XX:XX:XX:X2
      - /sbin/ip addr add X.X.X.3/32 dev lo3
    down:
      - /sbin/ip addr del X.X.X.3/32 dev lo3
      - /sbin/ip link delete lo3 type dummy

  - device: lo4
    auto: true
    family: inet
    method: loopback
    address: X.X.X.4
    netmask: 255.255.255.255
    pre-up:
      - /sbin/ip link add lo4 type dummy
    up:
      - /sbin/ip link set lo4 address XX:XX:XX:XX:XX:X3
      - /sbin/ip addr add X.X.X.4/32 dev lo4
    down:
      - /sbin/ip addr del X.X.X.4/32 dev lo4
      - /sbin/ip link delete lo4 type dummy
```

Role Variables
--------------

```yaml
bird_rc_pre_actions:
  - ethtool -G ethXXX tx 4096 rx 4096
  - ethtool -A ethXXX tx off rx off
  - ethtool -L ethXXX combined 16
  - ethtool -K ethXXX ntuple on
```
```yaml
bird:
  router_id: "X.X.X.1"
  tables:
    - name: table_1 # should be defined for each element in tables
      bgp:
        peers: # should be defined for bgp
          - X.X.X.5
          - X.X.X.6
        local_as: "1234567890" # should be defined for bgp
        neighbor_as: "1023456789" # should be defined for bgp
        export: filter table_1_allowed_nets # should be defined for bgp
        bfd:
          interface_name: ethXXX # should be defined for bfd
      kernel:
        table: 10 # should be defined for kernel
      direct:
        interface: [ '"lo2"', '"lo3"', '"lo4"'] # should be defined for direct
      filter:
        - ipnet: X.X.X.2/32
          prepend: 1
        - ipnet: X.X.X.3/32
          prepend: 2
        - ipnet: X.X.X.4/32
          prepend: 3
```

Example Playbook
----------------

```yaml
  - hosts: all
    roles:
       - { role: bird }
```
