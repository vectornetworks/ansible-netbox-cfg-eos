# Overview
This project provides a basic example of how Netbox data can be converted to network device configuration and applied by Ansible.  It was built as a proof-of-concept using Arista CEOS devices to illustrate how Netbox and Ansible could work together to realize a Network as Code configuration model.  Currently it comprises two playbooks, one that configures the devices' interface IP addressing and another that builds their BGP sessions.  The playbooks aren't long or complex, but we believe they may have some value to others that are looking for a starting point for how Netbox and Ansible could be used in tandem.

## Playbook Information and Requirements

The playbooks in this repo aren't necessarily meant to be run and used; they are just an example of how one might pull data from Netbox and push it using Ansible.  However, they have been run in test environments so if you are looking to give them a spin here's what you need to know.

There are two playbooks that actually perform the work:

* *cfg_eos_ints.yml* - configures interfaces as L3 (no switchport) and applies IP addresses
* *cfg_eos_bgp.yml* - builds BGP sessions, configures router IDs and Loopback originations

The other playbooks (e.g., query_) will illustrate some basic ways to query Netbox using Ansible and output the results.

These playbooks were tested against a 2 spine, 4 leaf Arista CEOS test topology built using [docker-topo](https://github.com/networkop/docker-topo), but could theoretically be used on any devices that are in a similar configuration.  The docker-topo files have been included in this repo in case you would like to reproduce (bring your own CEOS image).

The Netbox data should model whatever IP addressesing, interfaces, and BGP sessions you are looking to configure.  One quirk of the interfaces is that they require a tag applied called **l3base** if they are going to be configured as native L3.  If you are looking for a quick way to bring up Netbox and load sample data, check out [here](https://github.com/vectornetworks/netbox-vagrant-baselab).  The Ansible inventory hostnames should match the Netbox hostnames.

### In summary, here's what you need
* An Ansible control node (from which to run the playbooks)
* An Ansible inventory with hostnames that match the Netbox device names
  - Bonus points if this is pulled from Netbox itself
  - A copy of the inventory we used for the docker-topo CEOS test is also included
* A Netbox instance populated with devices, interfaces, IP address data and, optionally, BGP sessions using the netbox-bgp plugin
  - A way to load sample Netbox data can be found [here](https://github.com/vectornetworks/netbox-vagrant-baselab) 
* An Arista EOS network test bed to which the configurations will be deployed
  - We use [docker-topo](https://github.com/networkop/docker-topo) with CEOS. Configuration for both the topology and device bases included in the 'docker-topo' folder.

## Running the Playbooks
Prior to running the playbooks, you'll need to set your Netbox URL and API key.  This can be accomplished by setting the `NETBOX_URL` and `NETBOX_API_TOKEN` environment variables, respectively. For example:

```
export NETBOX_URL=http://mynetboxhostname/
export NETBOX_API_TOKEN=mynetboxapitoken
```
You could also directly modify the playbooks' variables `nb_api_endpoint` and `nb_api_token` if you prefer to hard set those values.

After the URL and token have been specified and assuming connectivity to the test devices from the Ansible control node, the playbooks can be run with Ansible playbook:

```
ansible-playbook -i <yourinventoryfile> cfg_eos_ints.yml
ansible-playbook -i <yourinventoryfile> cfg_eos_bgp.yml
```

