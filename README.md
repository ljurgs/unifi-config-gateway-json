ljurgs.unifi_config_gateway_json
=========

A role for managing config.gateway.json for site(s) on a Ubiquiti UniFi controller. 

https://help.ui.com/hc/en-us/articles/215458888-UniFi-USG-Advanced-Configuration-Using-config-gateway-json

Note
====

This role has been tested against a CloudKey Gen1 and a USG (with recent firmware). It should work with the USG Pro. I am not sure about the UDM/UDM Pro.
If some brave soul could try this on other versions after making backups and either letting me know or opening an issue, I'd love to work with you to iron out any bugs. 

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
        settings: {}

Explanation of variables
------------------------

    usg_controller_base_path: /usr/lib/unifi

The controller base path is required, this Ubiquiti doc explains how to find the unifi_base: https://help.ui.com/hc/en-us/articles/115004872967-UniFi-Where-is-unifi-base-

    usg_controller_process_user: unifi
    usg_controller_process_group: unifi
    
I'm not sure that the Unifi controller can run as any other user/group, in any case the configuration files must be owned by the same uid/gid as the process.

    usg_controller_base_url: https://unifi-cloudkey.local:8443
    usg_controller_username: ubnt
    usg_controller_password: ubnt

 The address/credentials used to log into the controller web interface. You will not need these if you don't opt to use the `provision: True` option for any the sites.

    usg_controller_sites:
      - name: default 
        provision: False 
        usg_mac: 78:xx:xx:xx:xx:xx
        settings: {}
        
This is the list of dicts that define the sites the controller manages. Do not list a site if it doesn't require a config.gateway.json file. 

The `name` variable can be obtained from the controller web url after login e.g. https://127.0.0.1:8443/manage/site/default/dashboard where in this case default is the site name. 
Sometimes it appears as a random string of alphabetical characters.

The `provision` variable defines if there should be a "forced" provision of the site USG/USG Pro device to apply the new settings files. Use this with caution until you are confident your file is valid. Services on the USG can stop working if invalid JSON is pushed to the device. Provisioning can also hang.

The `usg_mac` variable is the site lan facing MAC address of the USG/USG Pro device. You can find this by navigating to the properties pane for the device in the controller web interface.

The `settings:` variable is a dict of settings that will be converted to JSON in the config.gateway.json file for the site. You will need to refer to the Ubiqiti documentation for more information: https://help.ui.com/hc/en-us/articles/215458888-UniFi-USG-Advanced-Configuration-Using-config-gateway-json. A really useful tool for getting your current configuration is `mca-ctrl -t dump-cfg` this command must be run on the USG/USG Pro. So you'll need SSH access to that device.   

Examples
--------

Example of settings for an iPXE based chain loader that has the script target compiled in, architecture-type is collected and both UEFI and BIOS based booting is supported: 

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

Example of the settings for the DNAT rule as given on the Unifi documentation web page at: https://help.ui.com/hc/en-us/articles/215458888-UniFi-USG-Advanced-Configuration-Using-config-gateway-json

    settings: 
      service:
        nat:
          rule:
            1:
              destination:
                port: 53
              inbound-interface: eth0
              inside-address:
                address: 10.0.0.1
                port: 53
              protocol: tcp_udp
              type: destination 
        
Dependencies
------------

None

Example Playbook
----------------

    - name: Maintain the config.gateway.json file on the unifi-cloukey
      hosts: cloudkey
      vars_files:
        - vars/unifi.yml
      roles:
         - { role: ljurgs.unifi-config-gateway-json }

License
-------

BSD-3-Clause
