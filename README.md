POSTGRESQL
==========
**COPYRIGHT** 2022-2023 Valtion tieto- ja viestintätekniikkakeskus Valtori  
**LICENSE** GPL-2.0-or-later  
**AUTHORS**
- Arsi Atomi <arsi.atomi@valtori.fi>  

Overview
========

This Ansible Role is meant for easier and more flexible PostgreSQL management.

TL;DR: Just install and run: **[here](#TLDR)**

Requirements
------------

- Operating systems (one mandatory)
  - Fedora 36 or higher
  - Red Hat Enterprise Linux 8 or higher

- Other components
  - Ansible 2.13 or higher

Definitions
-----------

PostgreSQL documentation defines cluster as a collection of databases. This can be confusing, since clusters are commonly
used for collection of host computers. In this document PostgreSQL cluster has been replaced with "instance". Host can 
include 1 or more instances.

```plantuml
hide circle

entity "postgresql" {
  * pg_version = "15"
}
entity "postgresql-instance" {
  * pg_instance = "data"
  * pg_version = "15"
--
  pg_port = "5432"
}
entity "postgresql-database" {
  * pg_dbname
  * pg_instance = "data"
  * pg_version = "15"
--
  pg_dbowner = "postgres"
}
postgresql ||-|{ "postgresql-instance"
"postgresql-instance" ||-|{ "postgresql-database"
```

Tasks
-----

| Description                                               | State                      | Mandatory variables[^1]         |
|:----------------------------------------------------------|:---------------------------|:--------------------------------|
| [Install PostgreSQL packages](#present)                   | present                    |                                 |
| [Remove PostgreSQL packages](#absent)                     | absent                     |                                 |
| [Install PostgreSQL instance](#instance_present)          | instance_present           |                                 |
| [Remove PostgreSQL package](#instance_absent)             | instance_absent            |                                 |
| [Start PostgreSQL instance](#instance_start)              | instance_started           |                                 |
| [Stop PostgreSQL instance](#instance_stop)                | instance_stopped           |                                 |
| [Create database](#database_present)                      | database_present           | pg_dbname                       |
| [Remove database](#database_absent)                       | database_absent            | pg_dbname                       |
| [Remove all PostgreSQL related](#all_absent)              | all_absent                 |                                 |
| [User in database present](#user_in_database_present)     | user_in_database_present   | pg_dbuser, pg_dbname            |
| [User in database absent](#user_in_database_absent)       | user_in_database_absent    | pg_dbuser, pg_dbname            |


Install PostgreSQL packages (<a name="present">present</a>)
===========================================================

Description
-----------

Setups PostgreSQL repository if needed and installs PostgreSQL packages.


Playbook examples
-----------------

For installing with defaults

```yaml
---
- name: "Install PostgreSQL"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: present
```

For installing PostgreSQL version 14

```yaml
---
- name: "Install PostgreSQL"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: present
    pg_version: "14"
```

<a name="TLDR">For installing single PostgreSQL with defaults, initializing database and starting it</a>

```yaml
---
- name: "Install single PostgreSQL with defaults"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: present

  - role: postgresql
    state: instance_present
    
  - role: postgresql
    state: instance_started

```

Tasks overview
--------------

- Checking if the PostgreSQL server packages are already available
- Installing the repository RPM key for the target distribution
- Installing the PostgreSQL repository for the target distribution
- Installing the PostgreSQL server and contrib packages
- Installing the python3-psycopg2 library, which is required for connecting to the PostgreSQL server using Python

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_version**       | "15"                            | PostgreSQL version                      |


Remove PostgreSQL packages (<a name="absent">absent</a>)
========================================================

Description
-----------

Removes PostgreSQL packages, but not postgres user or data dir.


Playbook examples
-----------------

For removing default version packages.

```yaml
---
- name: "Remove PostgreSQL packages"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: absent
```

For removing PostgreSQL version 14 packages.

```yaml
---
- name: "Remove PostgreSQL packages"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: absent
    pg_version: "14"
```

Tasks overview
--------------

- Stopping postgresql-{{ *pg_version* }}-{{ *pg_instance* }} services and disabling them from starting on boot
- Uninstalling PostgreSQL {{ *pg_version* }} packages

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_version**       | "15"                            | PostgreSQL version                      |


Install PostgreSQL instance (<a name="instance_present">instance_present</a>)
=============================================================================

Description
-----------

Installs PostgreSQL instance defined in {{ *pg_instance* }}.

Two instances cannot be at the same port and started at the same time (defined in {{ *pg_port* }}).

Playbook examples
-----------------

Target directory in this example will be `/var/lib/pgsql/15/mydata/`.

```yaml
---
- name: "Add PostgreSQL instance"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: instance_present
    pg_instance: mydata
```

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_port**          | 5432                            | Port address                            |
| **pg_version**       | "15"                            | PostgreSQL version                      |


Remove PostgreSQL instance (<a name="instance_absent">instance_absent</a>)
==========================================================================

Description
-----------

Stops and removes instance data dir and systemd files.

Playbook examples
-----------------
```yaml
---
- name: "Remove PostgreSQL instance"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: instance_absent
    pg_instance: mydata
```

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version                      |


Start PostgreSQL instance (<a name="instance_started">instance_started</a>)
===========================================================================

Description
-----------

Start PostgreSQL instance {{ *pg_instance* }} service.

Playbook examples
-----------------
```yaml
---
- name: "Start PostgreSQL instance"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: instance_started
    pg_instance: mydata
```

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version


Stop PostgreSQL instance (<a name="instance_stopped">instance_stopped</a>)
==========================================================================

Description
-----------

Stop PostgreSQL instance {{ *pg_instance* }}

Playbook examples
-----------------

```yaml
---
- name: "Stop PostgreSQL instance"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: instance_stopped
```

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version                      |



Create database (<a name="database_present">database_present</a>)
================================================================

Description
-----------

Create database {{ *pg_dbname* }} to instance {{ *pg_instance* }}

{{ *pg_version* }} is mandatory as it is part of the directory hierarchy (default: `/var/lib/pgsql/15/data` for example)

Playbook examples
-----------------

```yaml
---
- name: "Create database"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: database_present
```

Tasks overview
--------------

- Sets the value of {{ *pg_dbowner* }} to {{ *pg_user* }} if {{ *pg_dbowner* }} is undefined.
- Creates PostgreSQL database with name {{ *pg_dbname* }} and sets {{ *pg_dbowner* }} as owner.

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_dbname**        |                                 | Database name                           |
| pg_dbowner           | "postgres"                      | Database owner                          |
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version                      |


Remove database (<a name="database_absent">database_absent</a>)
===============================================================

Description
-----------

Remove database {{ *pg_dbname* }} from instance {{ *pg_instance* }}

{{ *pg_version* }} is mandatory as it is part of the directory hierarchy (default: `/var/lib/pgsql/15/data` for example)

Playbook examples
-----------------

```yaml
---
- name: "Remove database"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: database_absent
```

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_dbname**        |                                 | Database name                           |
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version                      |


Remove all PostgreSQL related (<a name="all_absent">all_absent</a>)
===================================================================

Description
-----------

Remove all postgresql data, packages and postgres user.

Playbook examples
-----------------

```yaml
---
- name: "Remove all PostgreSQL packages, user and directories"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: all_absent
```

Variables
---------

No variables.

Add user to database (<a name="user_in_database_present">user_in_database_present</a>)
======================================================================================

Description
-----------

Adds user access to database.

Playbook examples
-----------------

```yaml
---
- name: "Add user(s) to database(s)"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: user_in_database_present
```

Tasks overview
--------------

- Adds {{ *pg_dbuser* }} to `pg_hba.conf`
- Grants all privileges to user {{ *pg_dbuser* }} in database {{ *pg_dbname* }}

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_dbname**        |                                 | Database name                           |
| **pg_dbuser**        |                                 | Database user
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version                      |

Remove user from database (<a name="user_in_database_absent">user_in_database_absent</a>)
=========================================================================================

Description
-----------

Removes user access to database.

Playbook examples
-----------------

```yaml
---
- name: "Remove user(s) from database(s)"
  hosts: postgresql
  become: yes

  roles:
  - role: postgresql
    state: user_in_database_absent
```

Tasks overview
--------------

- Removes {{ *pg_dbuser* }} from `pg_hba.conf`
- Revoke all privileges to user {{ *pg_dbuser* }} in database {{ *pg_dbname* }}

Variables
---------

Mandatory variables are bolded.

Default value is used if possible for undefined variables.

| Variable             | Default value                   | Description                             |
|:---------------------|:--------------------------------|:----------------------------------------|
| **pg_dbname**        |                                 | Database name                           |
| **pg_dbuser**        |                                 | Database user
| **pg_instance**      | "data"                          | Instance name (data dir)                |
| **pg_version**       | "15"                            | PostgreSQL version                      |


Footnotes
=========

[^1]: Mandatory variables without default values (from defaults/main.yml for example)

