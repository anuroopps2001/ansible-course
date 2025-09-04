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
