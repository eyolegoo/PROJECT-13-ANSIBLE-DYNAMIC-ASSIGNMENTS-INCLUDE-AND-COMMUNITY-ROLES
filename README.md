## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

- IMPORTANT NOTICE: Ansible is an actively developing software project, so you are encouraged to visit [Ansible Documentation](https://docs.ansible.com/) for the latest updates on modules and their usage.

- Last 2 projects have already equipped you with some knowledge and skills on Ansible, so you can perform configurations using **playbooks, roles and imports**. Now you will continue configuring your UAT servers learning and practicing new Ansible concepts and modules.

- In this project we will [introduce dynamic](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) assignments by using ***include module***.

- Now you may be wondering, what is the difference between ***static*** and ***dynamic*** assignments?

- Well, from **Project 12**, you can already tell that static assignments use **import** Ansible module. The module that enables dynamic assignments is **include**.

- Hence,

```
import = Static
include = Dynamic
```

- When the **import** module is used, all statements are pre-processed at the time playbooks are [parsed](https://en.wikipedia.org/wiki/Parsing). Meaning, when you execute ***site.yml*** playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

- On the other hand, when **include** module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are **parsed**, any changes to the statements encountered during execution will be used.

- Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.

## ***INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE***

- ***Introducing Dynamic Assignment Into Our structure***

- In your **https://github.com/<your-name>/ansible-config-mgt** GitHub repository start a new branch and call it **dynamic-assignments**.
  
- Create a new folder, name it ***dynamic-assignments***. Then inside this folder, create a new file and name it ***env-vars.yml***. We will instruct ***site.yml*** to ***include*** this playbook later. For now, let us keep building up the structure.
  
- Your GitHub shall have following structure by now.
  
- Note: Depending on what method you used in the previous project you may have or not have **roles** folder in your GitHub repository – if you used **ansible-galaxy**, then **roles** directory was only created on your **Jenkins-Ansible** server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case – it is up to you.
  
```
  ├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
```

- Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as **servername**, **ip-address** etc., we will need a way to set values to variables per specific environment.
  
- For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new **YAML** files which we will use to set variables.
  
- Your layout should now look like this.
  
```
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml  
```
  
- Now paste the instruction below into the **env-vars.yml** file.
  
```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

- Notice 3 things to notice here:
  
1. - We used **include_vars** syntax instead of include, this is because Ansible developers decided to separate different features of the module. From **Ansible version 2.8**, the **include** module is deprecated and variants of **include_*** must be used. These are:
  
  - [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)
  
  - [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)
  
  - [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)
  
- In the same version, variants of **import** were also introduces, such as:
  
  - [import_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)
  
  - [import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)
  
2. - We made use of a [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) **{ playbook_dir }** and **{ inventory_file }**. **{ playbook_dir }** will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. **{ inventory_file }** on the other hand will dynamically resolve to the name of the inventory file being used, then append **.yml** so that it picks up the required file within the **env-vars** folder.
  
  
3. - We are including the variables using a loop. **with_first_found** implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.
