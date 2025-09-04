# Defining the Inventory

An **inventory** defines a collection of hosts that Ansible manages.  
- Hosts can be assigned to **groups**, which can be managed collectively.  
- Groups can contain **child groups**, and hosts can be members of multiple groups.  
- The inventory can also set **variables** that apply to hosts and groups.  

---

## Types of Inventories

There are two main ways to define host inventories:

1. **Static Inventory**  
   - Defined in a text file.  
   - Commonly written in **INI-style** or **YAML** format.  
   - Best suited for smaller or fixed environments.  

2. **Dynamic Inventory**  
   - Generated at runtime using an **Ansible plug-in**.  
   - Pulls host information from external sources such as cloud providers, CMDBs, or APIs.  
   - Useful for large, dynamic environments.

---

## Static Inventory

A **static inventory file** is a text file that specifies the managed hosts Ansible targets.  
- **INI-style format** is the most common (used in this course examples).  
- **YAML format** can also be used.  

In its simplest form, an INI-style static inventory file is a list of hostnames or IP addresses of managed hosts, each on a single line:

```ini
web1.example.com
web2.example.com
db1.example.com
db2.example.com
192.0.2.42
```

Normally, however, you organize managed hosts into host groups. Host groups allow you to more effectively run Ansible against a collection of systems. In this case, each section starts with a host group name enclosed in square brackets ([]). This is followed by the hostname or an IP address for each managed host in the group, each on a single line.

In the following example, the host inventory defines two host groups: webservers and db-servers.

```bash
[webservers]
web1.example.com
web2.example.com
192.0.2.42

[db-servers]
db1.example.com
db2.example.com
```

Hosts can be in multiple groups. In fact, the recommended practice is to organize your hosts into multiple groups, possibly organized in different ways depending on the role of the host, its physical location, whether it is in production or not, and so on. This allows you to easily apply Ansible plays to specific sets of hosts based on their characteristics, purpose, or location.

```bash
[webservers]
web1.example.com
web2.example.com
192.0.2.42

[db-servers]
db1.example.com
db2.example.com

[east-datacenter]
web1.example.com
db1.example.com

[west-datacenter]
web2.example.com
db2.example.com

[production]
web1.example.com
web2.example.com
db1.example.com
db2.example.com

[development]
192.0.2.42
```

Defining Nested Groups
Ansible host inventories can include groups of host groups. This is accomplished by creating a host group name with the :children suffix. The following example creates a new group called north-america, which includes all hosts from the usa and canada groups.

```bash
[usa]
washington1.example.com
washington2.example.com

[canada]
ontario01.example.com
ontario02.example.com

[north-america:children]
canada
usa
```

A group can have both managed hosts and child groups as members. For example, in the previous inventory you could add a [north-america] section that has its own list of managed hosts. That list of hosts would be merged with the additional hosts that the north-america group inherits from its child groups.

**Simplifying Host Specifications with Ranges**
You can specify ranges in the hostnames or IP addresses to simplify Ansible host inventories. You can specify either numeric or alphabetic ranges. Ranges have the following syntax:

```bash
[START:END]
```

Ranges match all values from START to END, inclusively. Consider the following examples:

192.168.[4:7].[0:255] matches all IPv4 addresses in the 192.168.4.0/22 network (192.168.4.0 through 192.168.7.255).

server[01:20].example.com matches all hosts named server01.example.com through server20.example.com.

[a:c].dns.example.com matches hosts named a.dns.example.com, b.dns.example.com, and c.dns.example.com.

2001:db8::[a:f] matches all IPv6 addresses from 2001:db8::a through 2001:db8::f.

If leading zeros are included in numeric ranges, they are used in the pattern. The second example above does not match server1.example.com but does match server07.example.com.

**Verifying the Inventory**

You can use the ansible-navigator inventory command to query an inventory file. You might do this to verify the presence of a host or a group in the inventory.

The following examples assume that a file named inventory exists in the current directory and that the file uses ranges to simplify the [usa] and [canada] group definitions:

```bash
[usa]
washington[1:2].example.com

[canada]
ontario[01:02].example.com
```

The following query matches a host in the inventory. Although this output displays empty braces, if the inventory file defined variables for the host, then the braces would contain those variables.
```bash
[user@controlnode ~]$ ansible-navigator inventory -i inventory \
> -m stdout --host washington1.example.com
{}
```

Because the range for the usa group does not include leading zeros, the following query does not match a host in the inventory:

```bash
[user@controlnode ~]$ ansible-navigator inventory -i inventory \
> -m stdout --host washington01.example.com
...output omitted...
ERROR! You must pass a single valid host to --host parameter
```

The following command lists all hosts in the inventory.
```bash
[user@controlnode ~]$ ansible-navigator inventory -i inventory \
> -m stdout --list
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "canada",
            "ungrouped",
            "usa"
        ]
    },
    "canada": {
        "hosts": [
            "ontario01.example.com",
            "ontario02.example.com"
        ]
    },
    "usa": {
        "hosts": [
            "washington1.example.com",
            "washington2.example.com"
        ]
    }
}
```

```bash
[user@controlnode ~]$ ansible-navigator inventory -i inventory \
> -m stdout --graph canada
@canada:
  |--ontario01.example.com
  |--ontario02.example.com

[user@controlnode ~]$ ansible-navigator inventory -i inventory
  Title             Description
0│Browse groups     Explore each inventory group and group members members
1│Browse hosts      Explore the inventory with a list of all hosts

```

**Overriding the Location of the Inventory**
The /etc/ansible/hosts file is considered the system's default static inventory file. However, normal practice is not to use that file but to specify a different location for your inventory files.

The ansible-navigator commands that you use to run playbooks can specify the location of an inventory file on the command line with the --inventory PATHNAME or -i PATHNAME option, where PATHNAME is the path to the desired inventory file.


