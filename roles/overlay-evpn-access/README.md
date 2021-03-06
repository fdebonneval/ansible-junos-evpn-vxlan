
# 'overlay-evpn-access' role  
Generate configuration for access ports of the EVPN/VXLAN fabric
 - Aggregate interface, with or without ESI
 - Aggregate interface, with or without LACP
 - All interfaces in Access, trunk or normalized mode

Template can be found in [roles/overlay-evpn-access/templates/main.conf.j2 ](templates/main.conf.j2)

> Current roles/templates do not support to have normalized interface and non-normalized interface on the same device
> Junos support it, the limitation is coming from these templates

## Variables needed by the template

```yaml
access:
    nbr_ae:          # How many aggregate interface are present
    interfaces:      # List of interfaces
      - name:		          # Interface name. Interface connected to the customer device
	    server_role:          # Role of the servers for that include access,trunk and vlans in centeral file "prod-server-roles.yaml"
        mode: 			      # Interface mode. [trunk, access, aggregate ,normalize in case of use of server_role use role]
        vlans: 			      # List of vlan-id to configure on this interface. Structure is [ 100, 101, 102]
        description:	    # Interface description
        esi: 			        # ESI value configured manually
        lacp_system_id:   # LACP ID configured on both leaves. It must be the same to emulate a normal LACP LAG
        esi_neighbor: switch1 # If esi and/or lacp system-id are omitted use the id of esi_neighbor and the local id (from the inventory host)
							  # to generate the esi 8 and 9 octets (they are sorted and converted to hex) the 10 octets is the ae number this way it is uniq per pair of switches and ae number
        lacp_mode: 		    # LACP mode (active|passive)
        parent:			      # Aggregated interface (aeX)
		mtu:				# MTU for the access interface
        description: 	    # Interface description
        vni_vlans_mapping:    # VNI to VLAN mapping table used for Normalized interface
          <vni>:  <vlan_id>

```

### Example

> Unlike other roles in this project, this one is not designed to use merge hash.

**For Device 'qfx5100-03'**
```yaml
# host_vars/qfx5100-03/overlay-access.yaml
access:
    nbr_ae:   2
    interfaces:
      ## Interface single attached in trunk mode
        -   name: ge-0/0/1
            description: Peer hostname
            mode: trunk
			mtu: 9216
            vlans: [ 100, 101, 102, 103, 104, 105, 106, 107, 108 ]
      ## Interface single attached in normalized mode
        -   name: ge-0/0/1
            description: Peer hostname
            mode: normalize
            vni_vlans_mapping:
              1000:  10
              1001:  11
      ## Interface part of an ESI (multi-home) with LACP  in trunk mode      
        -   name: ae0
            mode: trunk
            esi: 00:01:01:01:01:01:01:01:01:01
            vlans: [ 100, 101, 102, 103, 104, 105, 106, 107, 108 ]
            lacp_system_id: 00:00:00:01:01:01
            lacp_mode: active
            description: "customer XX - server YY"
        -   name: ge-0/0/2
            description: Port Hostname
            mode: aggregate
            parent: ae0
            description: "customer XX - server YY"
      ## Interface part of an ESI (multi-home) without LACP in trunk mode      
        -   name: ae2
            mode: trunk
            esi: 00:01:01:01:01:01:01:01:01:01
            vlans: [ 100, 101, 102, 103 ]
            description: "customer XX - server YY"
        -   name: ge-0/0/10
            description: Port Hostname
            mode: aggregate
            parent: ae2
            description: "customer XX - server YY"
      ## Interface part of an ESI (multi-home) with LACP  in trunk mode and auto generate the esi  
	  ## For example ae144 where local id is 10 and leaf3 id is 122 will generate 01:02:03:04:05:06:07:0a:7a:90 (0a=10, 7a=122 90=144)  
        -   name: ae144
            mode: trunk
            esi_neighbor: leaf3
            vlans: [ 100, 101, 102, 103, 104, 105, 106, 107, 108 ]
            lacp_mode: active
            description: "customer XX - server YY"
        -   name: ge-0/0/2
            description: Port Hostname
            mode: aggregate
            parent: ae0
            description: "customer XX - server YY"
```

**For Device 'qfx5100-01'**
```yaml
# host_vars/qfx5100-01/overlay-access.yaml
access:
    nbr_ae:   2
    interfaces:
    ## Interface single attached in normalized mode
        -   name: ge-0/0/1
            description: Peer hostname
            mode: normalize
            vni_vlans_mapping:
              1000:  10
              1001:  11
```
