# Interview_qna_Ansible
# Ansible & Ansible Tower – Interview Deep Dive

This document contains:

1. Top 50 Moderate-Level Ansible Questions  
2. Practical Explanations of Core Concepts  
3. Top 10 Practical Experience-Validation Questions  

This is designed to separate:
- Theory-level users
- Copy-paste engineers
- Real production DevOps engineers

---

# PART 1 – Top 50 Moderate-Level Ansible Questions

## Core Concepts & Architecture

1. What happens internally when you run `ansible-playbook`?
2. How does Ansible ensure idempotency? Give a real example.
3. Difference between `command` and `shell` module?
4. What is the difference between ad-hoc commands and playbooks?
5. How does Ansible handle parallelism and forks?
6. What is the purpose of `gather_facts`? When should you disable it?
7. How does Ansible determine task execution order?
8. Difference between `import_tasks` and `include_tasks`?
9. Difference between `import_role` and `include_role`?
10. Explain Ansible’s push execution model.

---

## Inventory & Variables

11. Difference between static and dynamic inventory?
12. Explain variable precedence in Ansible.
13. What are `host_vars` and `group_vars`?
14. How do you override variables at runtime?
15. Difference between `vars`, `vars_files`, and `set_fact`?
16. When would you use a dynamic inventory plugin?
17. How do you structure multi-environment inventories?
18. What happens when two variables share the same name?
19. How do you encrypt inventory variables?
20. How do you debug variable precedence problems?

---

## Playbooks & Task Control

21. What are handlers?
22. Difference between `notify` and `listen`?
23. Explain `block`, `rescue`, and `always`.
24. What are `changed_when` and `failed_when`?
25. What is check mode? Does it always work?
26. How do you run a task only once?
27. What is `run_once` and when can it cause issues?
28. Difference between `delegate_to` and `local_action`?
29. How do you control execution order across hosts?
30. What is `serial`?

---

## Roles & Project Structure

31. What is the standard role directory structure?
32. How do you manage role dependencies?
33. What is `meta/main.yml` used for?
34. How do you design reusable roles?
35. How do you version-control roles?
36. What is Ansible Galaxy?
37. How do you override role default variables?
38. Difference between `defaults` and `vars` in roles?
39. How do you test roles before production?
40. How do you structure a large Ansible repository?

---

## Modules & Plugins

41. Difference between module and plugin?
42. How does Ansible communicate with managed nodes?
43. How do you write a custom module?
44. What are callback plugins?
45. What is a filter plugin?
46. What is a lookup plugin?
47. What are facts? How do you create custom facts?
48. When would you use the `raw` module?
49. How do you manage cross-OS package installation?
50. What are collections and why were they introduced?

---

# PART 2 – Practical Concept Explanations

------------------------------------------------------------------------

# 1. Can We Change the Roles Directory Inside a Collection?

**Short Answer:** No.

A collection has a fixed, enforced structure:

    namespace/
      collection_name/
        roles/
        plugins/
        docs/
        tests/
        galaxy.yml

The `roles/` directory inside a collection cannot be renamed or
relocated. Ansible expects that structure when resolving Fully Qualified
Collection Names (FQCN).

Example usage:

    my_namespace.my_collection.my_role

If you need custom role paths for standalone roles (not inside
collections), modify `ansible.cfg`:

    [defaults]
    roles_path = ./roles:/opt/ansible/roles

**Important:**\
This affects only traditional roles, not roles packaged inside
collections.

------------------------------------------------------------------------

# 2. What is a Collection?

A collection is a versioned, namespaced bundle of Ansible content.

It can include:

-   Roles\
-   Modules\
-   Lookup plugins\
-   Filter plugins\
-   Callback plugins\
-   Documentation\
-   Tests

Example Fully Qualified Collection Name (FQCN):

    ansible.builtin.copy
    community.kubernetes.k8s

Why collections were introduced:

-   Prevent naming conflicts\
-   Enable versioning\
-   Allow modular distribution\
-   Improve content organization

Collection metadata is defined in:

    galaxy.yml

Collections are installed using:

    ansible-galaxy collection install namespace.collection

------------------------------------------------------------------------

# 3. What is Molecule?

Molecule is a testing framework for Ansible roles.

It allows you to:

-   Create temporary test environments\
-   Apply a role\
-   Verify results\
-   Destroy the environment

Typical workflow:

    molecule init role myrole
    molecule create
    molecule converge
    molecule verify
    molecule destroy

Molecule supports drivers such as:

-   Docker\
-   Podman\
-   Delegated\
-   EC2 (via plugins)

**Key benefit:**\
It validates idempotency and ensures roles work before production
deployment.

Common CI integration:

-   GitHub Actions\
-   Jenkins\
-   GitLab CI

If idempotency fails, Molecule will detect it during repeated converge
runs.

------------------------------------------------------------------------

# 4. What is Ansible Vault?

Ansible Vault encrypts sensitive data.

It can encrypt:

-   Variable files\
-   YAML files\
-   Entire playbooks\
-   Strings

Basic commands:

    ansible-vault encrypt secrets.yml
    ansible-vault decrypt secrets.yml
    ansible-vault edit secrets.yml
    ansible-vault view secrets.yml

To run a playbook with vault:

    ansible-playbook site.yml --ask-vault-pass

Or use a vault password file:

    ansible-playbook site.yml --vault-password-file vault_pass.txt

In Ansible Tower / AAP:

-   Vault passwords are stored as Vault Credential types\
-   No need to expose password files in SCM

**Best Practice:**\
Never store secrets in plain text in Git repositories.

------------------------------------------------------------------------

# 5. What is Dynamic Inventory in Ansible Tower?

Dynamic inventory automatically pulls infrastructure data from external
systems.

Common sources:

-   AWS EC2\
-   Azure\
-   GCP\
-   VMware\
-   ServiceNow\
-   Custom inventory scripts

In Ansible Tower / AAP:

1.  Create Inventory\
2.  Add Inventory Source\
3.  Select cloud provider plugin\
4.  Attach credentials\
5.  Configure sync options

Important Tower concepts:

-   Inventory sync jobs\
-   Update on launch\
-   Cache timeout\
-   Overwrite variables option

Example:\
An AWS EC2 dynamic inventory can automatically group hosts based on
tags.

Dynamic inventory eliminates the need to manually maintain static host
files.

------------------------------------------------------------------------ 

# 20 Practical Interview Questions (Experience Detection Mode)

This document contains 20 practical, real-world interview questions
designed to identify whether a candidate has genuinely worked with
Ansible and Ansible Tower (AAP), or only theoretical exposure.

These questions focus on operational behavior, troubleshooting, scaling,
and production realities.

------------------------------------------------------------------------

# 1. You updated a role in Git, but Tower is still running old code. Why?

What would you check? - Project sync status - Correct branch selection -
SCM update on launch enabled - Execution Environment version -
Collection version mismatch

------------------------------------------------------------------------

# 2. A play fails on 5 out of 100 servers. What happens next?

What controls this behavior? - serial - max_fail_percentage -
any_errors_fatal - strategy plugins

------------------------------------------------------------------------

# 3. You must run a DB migration only once but deploy application on all hosts. How do you design it?

Expected concepts: - run_once - delegate_to - Separate play - Ordering
concerns

------------------------------------------------------------------------

# 4. Secrets are required in Tower. How do you pass them securely?

Expected: - Credential types - Vault credential - Avoid plaintext in
SCM - Avoid extra_vars for secrets

------------------------------------------------------------------------

# 5. How do you structure repositories for dev, qa, prod environments?

Expected structure discussion: - inventory segregation - group_vars -
environment isolation - branching strategy

------------------------------------------------------------------------

# 6. How do you test roles before production deployment?

Expected: - Molecule - ansible-lint - CI pipeline validation - Staging
inventory

------------------------------------------------------------------------

# 7. How do you debug variable precedence issues?

Expected tools: - ansible-inventory --graph - debug module - -vvv
verbosity - hostvars inspection

------------------------------------------------------------------------

# 8. Difference between Project Sync and Inventory Sync in Tower?

Project Sync: - Pulls SCM repository

Inventory Sync: - Updates hosts from dynamic source

------------------------------------------------------------------------

# 9. What breaks idempotency in production?

Examples: - shell without condition - timestamp-based file creation -
unmanaged templates - missing changed_when

------------------------------------------------------------------------

# 10. How do you scale Ansible in enterprise?

Expected: - Execution Environments - Controller clustering - External
database - Redis - Job slicing

------------------------------------------------------------------------

# 11. How do you handle rolling deployments safely?

Expected: - serial - health checks - pause module - load balancer
deregistration

------------------------------------------------------------------------

# 12. A dynamic inventory is not updating new hosts. What would you check?

Expected: - Inventory sync status - Credential validity - Cache
timeout - Update on launch setting

------------------------------------------------------------------------

# 13. How do you manage different OS families in one playbook?

Expected: - ansible_facts - when conditions - package abstraction -
include_tasks based on OS

------------------------------------------------------------------------

# 14. What is Execution Environment in AAP 2.x and why is it important?

Expected: - Containerized runtime - Dependency isolation - Collection
packaging - Version consistency

------------------------------------------------------------------------

# 15. How do you ensure playbook safety before execution?

Expected: - check mode - diff mode - syntax check - approval workflows
in Tower

------------------------------------------------------------------------

# 16. What are common reasons Tower jobs get stuck in pending?

Expected: - No available execution node - Instance group limits -
Credential issues - Project sync lock

------------------------------------------------------------------------

# 17. How do you implement blue-green deployment using Ansible?

Expected: - Separate inventory groups - Traffic switching logic - Health
verification - Controlled promotion

------------------------------------------------------------------------

# 18. How do you handle long-running tasks without blocking entire play?

Expected: - async - poll - async_status module

------------------------------------------------------------------------

# 19. What is the difference between linear and free strategy?

Linear: - Host-by-host task execution

Free: - Hosts execute tasks independently

------------------------------------------------------------------------

# 20. How do you audit what ran in Tower?

Expected: - Job history - Activity stream - API usage - Logging
integration (Splunk/ELK)

------------------------------------------------------------------------

# Conclusion

If a candidate can explain these scenarios with real examples, failure
cases, and production decisions --- they have real operational exposure.

If answers are generic or theoretical, they likely lack hands-on
experience.
