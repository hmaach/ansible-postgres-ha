# ansible-postgres-ha

Automated PostgreSQL high availability on RHEL with Patroni, etcd and Ansible, built for offline environments.

> Work in progress. Developed and tested in a local lab before use on real environments.

## Overview

This project deploys a 3-node PostgreSQL high availability cluster:

- **2 PostgreSQL nodes** managed by Patroni (one primary, one streaming replica)
- **1 witness node** running etcd only, providing the third vote for quorum

If the primary fails, Patroni promotes the replica automatically. The etcd quorum prevents split-brain: an isolated node cannot stay primary.

```
   +------------------+     +------------------+     +------------------+
   | node1            |     | node2            |     | witness          |
   | PostgreSQL 18    |====>| PostgreSQL 18    |     |                  |
   | Patroni          | WAL | Patroni          |     |                  |
   | etcd             |<--->| etcd             |<--->| etcd             |
   +------------------+     +------------------+     +------------------+
```

## Features

- Fully offline installation from local RPM repositories
- 3-member etcd quorum with a lightweight witness node
- Automatic failover and planned switchover with Patroni
- Old primary rejoins as a replica with `pg_rewind`
- Idempotent Ansible roles: safe to run multiple times
- Environment-specific settings kept in the inventory, roles stay generic
- Local lab with Vagrant to test everything before real deployments

## Requirements

| Component  | Version                         |
| ---------- | ------------------------------- |
| OS         | RHEL 9 (or compatible)          |
| PostgreSQL | 18                              |
| Ansible    | ansible-core 2.15+              |
| Lab        | Vagrant + VirtualBox or Hyper-V |

Local repositories must contain the PostgreSQL 18, Patroni and etcd RPMs with their dependencies.

## Repository structure

```
ansible-postgres-ha/
├── ansible.cfg
├── Vagrantfile
├── inventories/
│   └── lab/
│       ├── hosts.ini
│       └── group_vars/
│           └── all.yml
├── roles/
│   ├── common/          Local repos, time sync, hosts, firewall, storage
│   ├── etcd/            etcd install and quorum
│   ├── postgresql/      PostgreSQL 18 binaries
│   └── patroni/         Patroni install and cluster bootstrap
└── site.yml
```

## Quick start (lab)

```bash
git clone https://github.com/hmaach/ansible-postgres-ha.git
cd ansible-postgres-ha
vagrant up
ansible-playbook -i inventories/lab site.yml
```

Check the cluster:

```bash
vagrant ssh node1 -c "patronictl -c /etc/patroni/patroni.yml list"
```

## Roadmap

- [x] Lab VMs with offline repositories
- [ ] Roles: common, etcd, postgresql, patroni
- [ ] Full cluster build with `site.yml`
- [ ] Convert an existing standalone PostgreSQL to Patroni
- [ ] Add a replica to an existing cluster
- [ ] Failover test campaign (switchover, crash, network isolation, witness loss)
- [ ] Ansible Vault for secrets
- [ ] TLS for etcd and the Patroni REST API
- [ ] Backups with pgBackRest
- [ ] Monitoring (Patroni and etcd metrics)
- [ ] AWX compatibility
