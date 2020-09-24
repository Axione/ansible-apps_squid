# Ansible Role: ansible-apps_squid


## Description

[![Build Status](https://travis-ci.com/lotusnoir/ansible-apps_squid.svg?branch=master)](https://travis-ci.com/lotusnoir/ansible-apps_squid)[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)[![Ansible Role](https://img.shields.io/badge/ansible%20role-apps__squid-blue)](https://galaxy.ansible.com/lotusnoir/ansible-apps_squid/)[![GitHub tag](https://img.shields.io/badge/version-latest-blue)](https://github.com/lotusnoir/ansible-apps_squid/tags)

Deploy [squid](http://www.squid-cache.org/) proxy system using ansible.


## Role variables

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `squid_port` | 3128 | squid port |
| `squid_user` | proxy | |
| `squid_group` | proxy |  |
| `ulimit` | 65636 | max_open_files |
| `squid_dns_v4_first` | on | prefer to resolve ipv4 first |
| `squid_core_dump_dir` | /var/spool/squid |  |
| `squid_log_access` | /var/log/squid/access.log |  |
| `squid_log_cache` | /var/log/squid/cache.log |  |
| `squid_log_store` | /var/log/squid/store.log |  |
| `squid_snmp_allow` | true | activate snmp |
| `squid_snmp_port` | 3401 | snmp port to expose |
| `squid_cache_allow` | false |  |
| `squid_cache_dir` | /var/spool/squid |  |
| `squid_cache_mem` | 16 MB |  |
| `squid_cache_obj_size` | 15 MB |  |
| `squid_acl_subnets` | [] |  |
| `squid_acl_ports` | [] |  |
| `squid_auth_rules` | [] |  |
| `squid_auth_domain` | false |  |
| `squid_auth_rules_domain` | [] |  |

## Examples

	---
	- hosts: apps_squid
	  become: yes
	  become_method: sudo
	  gather_facts: yes
	  roles:
	    - role: ansible-apps_squid
	  environment: 
	    http_proxy: "{{ http_proxy }}"
	    https_proxy: "{{ https_proxy }}"
	    no_proxy: "{{ no_proxy }}

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.
