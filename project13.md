## DoOCUMENTATION OF PROJECT 13
---

**ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES**
---
NOTE: This is for personal use .  Ansible is an actively developing software project, you are therefore encouraged to visit [Ansible Documentation](https://docs.ansible.com/) for the latest updates on modules and their usage.
In this project I will introduce dynamic assignments by using include module. Remember we worked on `static-assignments` through `import` ansible module, in this project, we will focus on dynamic-assignments through the `include` ansible module.
-  When the `import` module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when `include` module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

Take note that in most cases it is recommended to use `static assignments` for `playbooks`, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project. 

----
**INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE**
-
- In your <https://github.com/<your-name>/ansible-config-mgt> GitHub repository start a new branch and call it dynamic-assignments.(Mine is `ansible-config-artifact`). Create a new folder, name it `dynamic-assignments`. Then inside this folder, create a new file and name it `env-vars.yml`. We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment. For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables. Your file structure should like the one below:
![File structure](./images/File%20structure.PNG)

Inside the `dynamics folder/env-vars.yml file`, paste the following commands:
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
if you observed closely, you'd see that I used `include_vars` syntax instead of mere include, this is because Ansible developers decided to separate different features of the module right from Ansible version 2.8, the include module is deprecated and variants of i`nclude_*` must be used. These are:
`include_role` `include_tasks` `include_vars`
In the same version, variants of import were also introduces, such as: `import_role` `import_tasks`
Following the documentation, I made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.
I included the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values just in case an environment specific env file does not exist.

---

**LET'S UPDATE OUR SITE.YML WITH DYNAMIC ASSIGNMENTS**
I
--
In `playbooks/site.yml`, paste the follwingsnippets of codes:
```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```
*Community Roles*
-
it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going just like i did in this project. 
- Hint: To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ‘ansible-config-mgt’ directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it – we will be using Jenkins later for a better purpose. I believe by now we have an idea on how to go about that. So, let's proceed!
I used the [MysQl role](https://galaxy.ansible.com/geerlingguy/mysql) developed by geerlingguy. Click that link to access the command to run on your terminal to install the whole `roles` package. After a successful installation, inside `roles` directory create your new `MySQL role` with `ansible-galaxy install geerlingguy.mysql` and rename the folder to `mysql` with this command `mv geerlingguy.mysql/ mysql` Read `README.md` file, and edit roles configuration to use correct credentials for MySQL required for the tooling website and once you're satisfued with the configuration, create a pull request on Github and merge it on the main branch.

---
**LOAD BALANCER ROLES**
-
We want to be able to choose which Load Balancer to use, `Nginx` or `Apache`, so we need to have two roles respectively. Install the two roles just the same way we installed mysql. Check the images below:
![Nginx](./images/Installation%20of%20nginx.PNG)
![Apache](./images/installation%20of%20apache.PNG)
You can decide to develop new roles or find some in the community. Configiure the two roles respectively, update the neccessary folders and files in other to have a smooth  connection anytime you run the ansible-playbook command.

- Hint: tweaking the folders and files in these two roles maybe a little hectic and frustrating, you just have to read through the README.md file to understand what to do and what not to do. I had a very chanleging experience in the process of tweaking the respective roles but i channelled those frustration to gaining more insights and understanding hence it was  a plus for me and itb should be for you too.
-
After you've succesfully installed the two roles, your roles folder should now look like this:
![roles folder](./images/roles%20folder.PNG)
 - while the entire repo structure should have the following. Check the images below to compare mine with yours to be sure you are the right track.
 ![Directory](./images/Directory%20tree.PNG)
 ![Directory](./images/Directory%20tree%202.PNG)
 ![Directory](./images/tree%203.PNG)
 ![Directory](./images/tree%204.PNG)

  **Hints**
 ---
Since we cannot use both Nginx and Apache load balancer, we need to add a condition to enable either one – this is where variables are essential

  - Now declare a variable in `defaults/main.yml` file inside the `Nginx` and `Apache` **roles** and name each variables `enable_nginx_lb` and `enable_apache_lb` respectively.

  - Let's set both values to false like this `enable_nginx_lb: false` and e`nable_apache_lb: false`.
  - Let's now declare another variable in both roles `load_balancer_is_required` and set its value to `false` as well just as we did above.

- It's time to update both `assignment` and `site.yml` files respectively.
***
If you don't have the loadbalancer and database files already created inside the static-assignments, create the files and name them `lb.yml` and `db.yml` respectively. Inside the `lb.yml` paste the following the codes:
```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```
Inside the `db.yml` file, paste the following codes:
```
---
 - hosts: db
   roles:
      - mysql
   become: true
```
Good!
- Now, let's update our `site.yml` file with the below codes:
```
- name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
```
Note that you make use of `env-vars\uat.yml` file to define which loadbalancer to use in UAT environment by setting respective environmental variable to `true`.
You will have to activate `load balancer`, and enable `nginx` by setting these in the respective environment’s `env-vars file`. Paste the following codes inside your `env-vars\uat.yml`
```
enable_nginx_lb: true
load_balancer_is_required: true
```
It's time to run the `ansible-playbook` command but before we do that, take a moment to compare your set up, folders and files externally and those in the `roles` directory with mine. You can access the complete repo [here](https://github.com/Lordchancellorr/ansible-config-artifact).
Check below to see the output i got after running the playbook command:
![ansible playbook](./images/ansible%20playbook%20output%201.PNG)
![output](./images/Output%202.PNG)
![output 3](./images/Output%203.PNG)
![output 4](./images/Output%204.PNG)
![output 5](./images/Output%205.PNG)
![output 6](./images/Output%206.PNG)
![output 7](./images/Output%207.PNG)
![output 8](./images/Output%208.PNG)
![output 9](./images/Output%209.PNG)

 **This brings us to the end of this project. Thank you and have a nice day!**
-
