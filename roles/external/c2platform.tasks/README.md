
# Ansible Role C2 Platform Core Tasks

Ansible role for reusable tasks. Current implementation of [Ansible Collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) do not allow tasks to be reused between collections.

So in order to prevent code duplication and ordinary role is used for this purpose.

Current this role has the following reusable tasks:

| File                  | Purpose |
|---------------------------|----|
| psql-terminate-block-sessions.yml | Block sessions to a PostgreSQL database |
| psql-database.yml | Manage a PostgreSQL database |
| psql-allow-sessions.yml | Allow sessions to a PostgreSQL database |

An example of this use can be found in __jira__ role e.g.

```yaml
- include_role:
    name: c2platform.tasks
    tasks_from: psql-database
  vars:
    lcm_role_upgrade: jira
  when: jira_database_type == 'postgresql'
```

