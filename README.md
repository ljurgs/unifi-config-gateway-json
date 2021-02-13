ljurgs.unifi-config-gateway-json
=========

A role for managing config.gateway.json for site(s) on a Ubiquiti UniFi controller.

I tested this against a CloudKey Gen1 and a USG. It should work with the USG Pro. I am not sure about the UDM/UDP Pro.

Role Variables
--------------

    ---
    usg_controller_base_path: /usr/lib/unifi
    usg_controller_process_user: unifi
    usg_controller_process_group: unifi
    usg_controller_base_url: https://unifi-cloudkey.local:8443
    usg_controller_username: ubnt
    usg_controller_password: ubnt
    usg_controller_sites:
      - name: default
        provision: False
        usg_mac: 78:xx:xx:xx:xx:xx
        settings:
          service:
            dhcp-server:
              global-parameters:
                - "class &quot;denied&quot; { match substring (hardware, 1, 6); deny booting; } subclass &quot;denied&quot; 78:xx:xx:xx:xx:xx; subclass &quot;denied&quot; 78:xx:xx:xx:xx:xx; subclass &quot;denied&quot; 78:xx:xx:xx:xx:xx;"
                - "option architecture-type code 93 = unsigned integer 16;"
              shared-network-name:
                "net_LAN_eth1_192.168.0.0-24":
                  subnet:
                    "192.168.0.0/24":
                      bootfile-server: 192.168.0.2
                      subnet-parameters:
                        - "if option architecture-type != &quot;00:00&quot; { option bootfile-name &quot;ipxe.efi&quot;; filename &quot;ipxe.efi&quot;; } else { option bootfile-name &quot;undionly.kpxe&quot;; filename &quot;undionly.kpxe&quot;; }"


Dependencies
------------

None

Example Playbook
----------------

    - hosts: cloudkey
      roles:
         - { role: ljurgs.unifi-config-gateway-json }

License
-------

BSD-3-Clause
