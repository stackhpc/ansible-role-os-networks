OpenStack Networks
==================

This role can be used to register networks, subnets and routers in neutron
using the os\_network, os\_subnet and os\_router modules.

Requirements
------------

The OpenStack neutron API should be accessible from the target host.

Role Variables
--------------

`os_networks_venv` is a path to a directory in which to create a virtualenv.

`os_networks_auth_type` is an authentication type compatible with the
`auth_type` argument of `os_*` Ansible modules.

`os_networks_auth` is a dict containing authentication information
compatible with the `auth` argument of `os_*` Ansible modules.

`os_networks_environment` is a dict containing OpenStack environment variables
to pass to the `os_*` Ansible modules.

`os_networks` is a list of networks to register. Each item should be a
dict containing the following items:

- `name`: Name of the neutron network.
- `provider_network_type`: Provider type of the neutron network.
- `provider_physical_network`: Provider physical network of the neutron
  network.
- `provider_segmentation_id`: Provider segmentation ID of the neutron network.
- `shared`: Whether the neutron network is shared.
- `external`: Whether the neutron network is external.
- `project`: Optionally create this network for a project other than the
  authenticating project.
- `state`: Optional state of the network, default is `present`.
- `subnets`: A list of subnets to create in this network. Each item should
   be a dict containing the following items:
   - `name`: Name of the neutron subnet.
   - `cidr`: CIDR representation of the neutron subnet's IP network.
   - `dns_nameservers`: A list of DNS nameservers for the subnet.
   - `gateway_ip`: IP address of the neutron subnet's gateway.
   - `enable_dhcp`: Whether to enable DHCP on the subnet.
   - `allocation_pool_start`: Start of the neutron subnet's IP allocation
     pool.
   - `allocation_pool_end`: End of the neutron subnet's IP allocation pool.
   - `host_routes`: A list of classless static routes to supply to hosts
     connected to this subnet. A list of dicts of `destination`
     (destination network in CIDR encoding) and `nexthop`
     (router IP on this subnet) must be supplied.
   - `ip_version`: Optional IP version for the subnet.
   - `ipv6_address_mode`: Optional IPv6 address mode for the subnet.
   - `ipv6_ra_mode`: Optional IPv6 router advertisement mode for the subnet.
   - `project`: Optionally create this subnet for a project other than the
     authenticating project.
   - `state`: Optional state of the subnet, default is `present`.

`os_networks_routers` is a list of routers to create. Each item should be a
dict containing the following items:

- `name`: Name of the neutron router.
- `interfaces`: List of names of subnets to attach to the router
  internal interface.
- `network`: Unique name or ID of the external gateway network.
- `project`: Optionally create this router for a project other than the
  authenticating project.
- `state`: Optional state of the router, default is `present`.


`os_networks_security_groups`: List of security groups to create.
Each item should be a dict containing the following items:
- `name`: Name of the security group.
- `description`: Optional description of the security group.
- `project`: Optional project in which to register the security group.
- `state`: Optional state of the security group, default is `present`.
- `rules`: Optional list of rules to add to the security group. Each item
  should be a dict containing the following items:
  - `direction`: Optional direction of the rule, default is `ingress`.
  - `ethertype`: Optional Ethertype of the rule, default is `IPv4`
  - `port_range_min`: Optional starting port.
  - `port_range_max`: Optional ending port.
  - `protocol`: Optional IP protocol of the rule.
  - `remote_group`: Optional name or ID of the security group to link.
  - `remote_ip_prefix`: Optional source IP address prefix in CIDR notation.
  - `state`: Optional state of the rule, default is `present`.

Dependencies
------------

This role depends on the `stackhpc.os-shade` role.

Example Playbook
----------------

The following playbook registers a neutron network, subnet and router.
A classless static route is defined to access another subnet through a
different gateway.

    ---
    - name: Ensure networks, subnets and routers are registered
      hosts: neutron-api
      roles:
        - role: os-networks
          os_networks_venv: "~/os-networks-venv"
          os_networks_auth_type: "password"
          os_networks_auth:
            project_name: <keystone project>
            username: <keystone user>
            password: <keystone password>
            auth_url: <keystone auth URL>
          os_networks:
            - name: net1
              provider_network_type: vlan
              provider_physical_network: physnet1
              provider_segmentation_id: 1234
              shared: true
              external: false
              subnets:
                - name: subnet1
                  cidr: 10.0.0.0/24
                  gateway_ip: 10.0.0.1
                  allocation_pool_start: 10.0.0.2
                  allocation_pool_end: 10.0.0.254
                  host_routes:
                    - destination: 10.0.1.0/24
                      nexthop: 10.0.0.254
          os_networks_routers:
            - name: router1
              interfaces:
                - subnet1
              network: net1
          os_networks_security_groups:
            - name: secgroup1
              rules:
                - protocol: icmp

Author Information
------------------

- Mark Goddard (<mark@stackhpc.com>)
