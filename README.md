# Project-13-Ansible-Dynamic-Assignments-and-Community-Role

## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

[Dynamic assignments](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments, where the values of variables are defined and fixed at the time of writing the playbook. In this project, we will continue configuring our UAT Servers whilst learning and practicing new Ansible concepts and modules. We shall discover the flexibility of Ansible with dynamic assignments (include) and  we shall leverage on Ansible pre-built solutons to expand our automation capabilities.

### Project Dependencies

In order to successfully execute this project, the following prerequisites need to be in place:

1. **`Jenkins-Ansible`** Server from [Project 12.](https://github.com/kalkah/Project-12-ANSIBLE-REFACTORING-AND-STATIC-ASSIGNMENTS-IMPORTS-AND-ROLES)
   
2. Two UAT Web Server Instances running Red Hat Enterprise Linux Distribution.

### <br>Introduction to Ansible Dynamic Assignments (include) and Community Roles<br/>

Dynamic assignments are useful in scenarios where the values of variables need to be determined based on certain conditions or inputs. From our work in [Project 12](https://github.com/kalkah/Project-12-ANSIBLE-REFACTORING-AND-STATIC-ASSIGNMENTS-IMPORTS-AND-ROLES) we can surmise that static assignments make use of the **`import`** ansible module. However, on the other hand, the module that enables dynamic assignments is the **`include`** ansible module.

`import = Static`    `include = Dynamic`

When the **`import`** module is used, all statements are pre-processed at the time playbooks are parsed. This means that when the **`site.yml`** playbook is executed, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when **`include`** module is used, all statements are processed only during execution of the playbook. This means that after the statements are parsed, any changes to the statements encountered during execution will be used.Although it is quite notable that static assignments are mostly used for playbooks (due to the fact that Dynamic assignments are less reliable and can be difficult to debug). Dynamic assignments have however proved useful for environment specific variables as will be introduced in this project. This project shall consist of two parts:

**1.** Making use of Dynamic Assignments.

**2.** Making use of Community Roles.

### <br>Making use of Dynamic Assignments<br/>

#### <br>Step 1: Introducing Dynamic Assignment Into the Structure<br/>

**i.** In our GitHub repository, we execute the following command to create and move into a new branch that we will call **`dynamic-assignments`**.

**`$ git checkout -b dynamic-assignments`**

**ii.** Next we create a folder and name it **`dynamic-assignments`**    **`$ mkdir dynamic-assignments`**

<img width="518" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/95f11579-d586-4a7c-92ed-cce3d155b4e7">

**iii.** We use the set of commands below to move into the dynamic-assignments folder and create a file named **`env-vars.yml`**

`$ cd dynamic-assignments`      `$ touch env-vars.yml`


<img width="527" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/7fb50bf1-bbc7-45a0-8bad-9295c6323547">

**iv.** At this point, our GitHub directory structure is as shown in the image below:

![image](https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/80818381-b4ac-4b50-9e73-ee04f0fd4c19)

**v.** Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment. For this reason, we proceed to create a folder **`env-vars`** to keep each environment’s variables file.

**`$ mkdir env-vars`**

<img width="500" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/869165f4-3134-4fd1-8d59-c8a2e1fb801d">

**vi.** Then, we move into the created **`env-vars`** folder, and for each environment, we create new **`YAML`** files which we will use to set variables.

`$ cd env-vars`      `$ touch dev.yml stage.yml uat.yml prod.yml`

<img width="584" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/80e6faff-cdc3-42e9-b0ce-dd47870d882d">

**vi.** Our directory layout now looks as shown in the image below:

![image](https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/392804ef-d60c-479b-8cc6-7aac4fe74b5e)

**vii.** Now satisfied with our folder structure, we move ahead and paste the following block of instruction into the **`env-vars.yml`** file.

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

<img width="439" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/3245c03c-f98d-4bbb-b6d6-565bdc826028">

**viii.** There are three things to note about the configuration above in **vii.**

1. We used **`include_vars`** syntax instead of **`include`**, this is because Ansible developers decided to separate different features of the module. From Ansible version **2.8**, the **`include module`** is deprecated and variants of **`include_*`** must be used. These are:

+ [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)
  
+ [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)
  
+ [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)

  
In the same version, variants of **`import`** were also introduced, such as:

+ [import_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)
  
+ [import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)
  
2. We made use of [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) **`{{ playbook_dir }}`** and **`{{ inventory_file }}`**. **`{{ playbook_dir }}`** will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. **`{{ inventory_file }}`** on the other hand will dynamically resolve to the name of the inventory file being used, then append **`.yml`** so that it picks up the required file within the **`env-vars`** folder.

3. We are including the variables using a loop. **`with_first_found`** implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

#### <br>Step 2: Update `site.yml` with Dynamic Assignments<br/>

The next step is to update the **`site.yml`** file to make use of the dynamic assignment. (It is however important to note that we cannot test it just yet. Rather, this is us preparing the stage for what is to come.) **`site.yml`** should look like this:

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

<img width="406" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/b3bff5a2-47b5-4b1c-8263-574255354ef2">

#### <br>Step 3: Update Git with Latest Code in `dynamic-assignments` Branch<br/> 

At this point, we we need to push all the changes we made locally to our remote Github repository.

**i.** We use the following commands to stage, commit and push our branch to GitHub:

`$ git status`      `$ git add <selected files>`      `$ git commit -m "commit message"`      `$ git push --set-upstream origin dynamic-assignments`

<img width="653" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/d6bb9df1-076d-4e78-b9bc-ab29863a48bf">

**ii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

+ This takes us to the **"Compare changes"** page where we choose the **`dynamic-assignments`** branch to set up a comparison with the **`main`** branch.

+ Once we set up the comparisons between the **`main`** and the **`dynamic-assignments`** branch, we then proceed to click on the **"Create pull request"** button.

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

**iii.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`dynamic-assignments`** branch and check for conflicts with the **`main`** branch.

**iv.** As we are satisfied and happy with the changes made in **`dynamic-assignments`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

**v.** This takes us to the next page which shows that **`dynamic-assignments`** has been successfully merged to **`main`** branch.

<img width="694" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/b9568198-b041-4c47-b865-f73ae0e8f640">

### <br>Making use of Community Roles<br/>

Now it is time to create a role for MySQL database which will perform the functions of installing the MySQL package, creating a database and configuring users. It is important to note that there are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. So rather than re-inventing the wheel, we simply make use of Ansible Galaxy again and we can simply download a ready to use ansible role.

#### <br>Step 1: Download Mysql Ansible Role<br/>

**i.** We proceed to browse available community roles [here](https://galaxy.ansible.com/ui/). We decide to use a [MySQL role developed by](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) **`geerlingguy`**.

**ii.** For now, we no longer need webhooks and Jenkins to update our codes on the **`Jenkins-Ansible`** server, so we disable it by clicking on **"Disable Project"** in the project status page in our Jenkins environment.

<img width="941" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/e1b9b692-03ae-4120-b74c-ea1aae5649b2">

**ii.** Next, we head to our Jenkins-Ansible server and then we go to the **`ansible-config-mgt`** directory and we checkout from the **`dynamic-assignments`** branch into the main branch, and then we pull down the latest changes.

`$ git checkout main`      `$ git pull`


<img width="448" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/606ffecb-da27-48a1-a99a-0907b1c59350">

**iii.** Then we execute the following command to create and move into a new branch that we will call **`roles-feature`**.

**`$ git checkout -b roles-feature`**

<img width="472" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/6187e988-3ed5-4cf9-9481-98d22f89585f">

**iv.** Inside the **`roles`** directory, we create a new MySQL role with ansible-galaxy and then we rename the folder to **`mysql`**:

`$ ansible-galaxy install geerlingguy.mysql`      `$ mv geerlingguy.mysql/ mysql`


<img width="604" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/32ae58d2-a97c-42e7-a1d2-7a183533f5c9">


#### <br>Step 2: Configure Mysql Ansible Role<br/>

**i.** The next thing we do is to go through the **README.md** file, and edit roles configuration to use the correct credentials for MySQL required for the **`tooling`** website. We navigate via VS Code explorer through **`roles > mysql > defaults > main.yml`** and we edit configuration in the **`main.yml`** file as shown in the image below:

<img width="458" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/fb4a3581-6f23-414e-a347-4227a11a203b">

NB: The IP Address in above image is the db private IP Address

**ii.** To reference, the database role, we do the following:

+ Go into the **README.md** file and copy the following block of code:

  ```
  ---
  - hosts: database
    roles:
    - role: geerlingguy.mysql
      become: yes
  ```

+ Then we move into the **`static-assignments`** folder and create a file we name **`db.yml`**

`$ cd static-assignments`      `$ touch db.yml`

<img width="491" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/9591914c-8105-4c8c-90dc-244992a38bf6">

+ Then we enter the file  and we paste in the configuration as shown in the image below:

<img width="219" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/14fd605a-9561-467a-99e9-ff94d7b163a5">

**iii.** We also need to reference the **`db.yml`** role inside **`site.yml`**. So, we put the following configuration inside site.yml:

```
-  hosts: db
- name: Installing db
  import_playbook: ../static-assignments/db.yml
```

<img width="377" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/24906c04-9919-4527-ac46-f6f07644d49f">

**iv.** We proceed to update the inventory **`ansible-config/inventory/dev.yml`** file with the Private IP address of our DB server using the following syntax:

```
[db]
<db-Server-Private-IP-Address> ansible_ssh_user=ubuntu
```

<img width="282" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/4dbab04c-b0a3-412d-a74d-383f439fb87d">


#### <br>Step 3: Commit and Test DB Playbook<br/>

In this step, we shall stage and commit our changes in the **`roles-feature`** branch. then we shall test run the db playbook. We decided to do this with the consideration that it would be cleaner and less cumbersome rather than executing it with the other roles we are going to download and configure later on in this project. It will also be easier to debug and investigate any potential errors our playbook might encounter.

**i.** Using the following set of commands, we add all untracked files to the staging area and then we commit the changes:

`$ git add .`      `$ git commit -m "saved all updates to role-features branch"`

<img width="638" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/0a6cbde6-cd33-4d5f-aa68-10325d6185f5">

**ii.** The next step is to ensure connection to our **`Jenkins-Ansible`** server via **`ssh-agent`** as we did in [Project 11](https://github.com/kalkah/Project-11-Ansible-Automate) and then with the commands below, we run the playbook against our **`dev inventory`**.

`$ cd /home/ubuntu/ansible-config-mgt`      `$ ansible-playbook -i /inventory/dev.yml playbooks/site.yml`


#### BLOCKER❗

![image](https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/0e9c30c7-3ee8-4921-bc40-50632f385ac7)

+ We got the error above when we ran the playbook, but after we examined the error message and did some investigations we reconfigure site.yml file

<img width="356" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/049c93b2-4b46-40ed-93bd-f347a527eb85">

+ So we fixed this issue and the playbook ran successfully as shown in the images below:

<img width="660" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/e10dc0cc-a5d3-4198-9b3d-2e8dfc18de82">


#### <br>Step 4: Load Balancer Roles<br/>

We want to be able to make a choice on which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively: Nginx and Apache.

**i.** We refer to what we did earlier with **`msql`** role and we do the same for both nginx and apache load balancer roles. Inside the **`roles`** directory, we create new roles for both nginx and apache load balancers with ansible-galaxy. And then we rename the folders to **`nginx-lb`** and  **`apache-lb`** respectively.

```
$ sudo ansible-galaxy install geerlingguy.nginx

$ mv geerlingguy.nginx/ nginx-lb

$ sudo ansible-galaxy install geerlingguy.apache

$ mv geerlingguy.apache/ apache-lb
```

<img width="611" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/4226ce54-e8b5-47b7-bf72-90a54d4a40f5">

**ii.** To make Nginx and Apache function as a loadbalancer, we need to update the **`default/main.yml`** file inside the Nginx and Apache roles with the private IP address of our two UAT webservers 


**iii.** Since we cannot use both Nginx and Apache as load balancer at the same time, we need to make use of variables to enable either one depending on preference. We proceed to declare the variables in the **`default/main.yml`** file inside the Nginx and Apache roles as follows:

+ We name each variable **`enable_nginx_lb`** and **`enable_apache_lb`** respectively.

+ We set both variable values to false as follows: **`enable_nginx_lb: false`** and **`enable_apache_lb: false`**

+ We declare another variable **`load_balancer_is_required`** and we set its value to **`false`** as well.

<img width="450" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/a9793f61-4c37-48ff-b303-62f86d04a056">

<img width="437" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/08c37643-b7af-4960-a497-4456d0a04969">

**iv.** Next, we update **`static-assignements`** by creating a **`loadbalancers.yml`**:

**`$ touch loadbalancers.yml`**

<img width="551" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/5644f3b9-b0ba-4238-a522-41439e862206">

**v.** And then, we subsequently paste the following block of configuration into the file we just created:

```
- hosts: lb
  roles:
    - { role: nginx-lb, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache-lb, when: enable_apache_lb and load_balancer_is_required }
  become: true
```

<img width="512" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/58e4c391-de99-4c56-9e4a-f001e892e536">


**vi.** The next step is to update the entry point for our playbooks, the **`site.yml`** file with the following configuration block:

```
-hosts: lb
- name: Loadbalancers assignment
- import_playbook: ../static-assignments/loadbalancers.yml
when: load_balancer_is_required 
```

<img width="362" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/38a8b61b-1aab-4d5e-bc96-791b686bb535">

**vii.** Afterwards, we update our **`inventory/dev.yml`** file with the private IP address of the load balancer server as shown below:

<img width="304" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/48ccbc8d-1ad1-4db6-aabc-8a9f543284a9">

**viii.** To complete our configuration, we need to decide whether it is Nginx or Apache that is used as a loadbalancer at any given time.  We make use of the **`env-vars/dev.yml`** file to define which load balancer to use in the UAT environment by setting the respective environmental variables to **`true`** or **`false`**.

+ By setting the variable below in the **`env-vars/dev.yml`** file, we activate the load balancer and enable **`nginx`**.

```
enable_nginx_lb: true
load_balancer_is_required: true
```
+ By setting the variable below in the **`env-vars/dev.yml`** file, we activate the load balancer and enable **`apache`**.

```
enable_apache_lb: true
load_balancer_is_required: true
```
+ **Tip**: *After testing the variable for the first load balancer, we must stop the load balancer service (e.g **`sudo systemctl stop apache2`** before setting the variable for the second load balancer and subsequently running the playbook against it. This is imperative to avoid errors as two loadbalancers cannot run simultaneously on the same server.*
  
**ix.** Next, we run our playbook by executing the command below:

**`ansible-playbook -i inventory/dev.yml playbooks/site.yml`**

+ The following images show the playbook output when we set the variable to enable **`nginx`** load balancer:

![image](https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/050ca57d-88e5-47dd-802d-bb15e48ada7e)

![image](https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/9c864c3c-f422-459e-9a40-31428fda4a77)

#### <br>Step 5: Update Git with Latest Code in `roles-feature` Branch<br/> 

At this point, we we need to push all the changes we made locally to our remote Github repository.

**i.** We use the following commands to stage, commit and push our branch to GitHub:

```
$ git status

$ git add <selected files>

$ git commit -m "commit message"

$ git push --set-upstream origin roles-feature
```

<img width="582" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/abad3169-b92a-409a-8daa-a6d7305628b5">

<img width="510" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/e56ebbbe-d109-46e3-a5e3-378193fb7fe3">

**ii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

+ This takes us to the **"Compare changes"** page where we choose the **`roles-feature`** branch to set up a comparison with the **`main`** branch.

+ Once we set up the comparisons between the **`main`** and the **`roles-feature`** branch, we then proceed to click on the **"Create pull request"** button.

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

**iii.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`roles-feature`** branch and check for conflicts with the **`main`** branch.

**iv.** As we are satisfied and happy with the changes made in **`roles-feature`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

**v.** This takes us to the next page which shows that **`roles-feature`** has been successfully merged to **`main`** branch.

<img width="545" alt="image" src="https://github.com/kalkah/Project-13-Ansible-Dynamic-Assignments-and-Community-Role/assets/95209274/1486c43e-6e6f-4f85-a018-de39fcda9803">


### <br>Conclusion<br/>

The output images from **ix.** above show that we have been able to successfully use Ansible Dynamic Assignments and Roles to prepare UAT environment for tooling web solution. The goal of our project was to demonstrate the flexibility of Ansible with dynamic assignments (include) and leverage on Ansible roles to how the automation capabilities demonstrated in Projects [11](https://github.com/kalkah/Project-11-Ansible-Automate) and [12](https://github.com/kalkah/Project-12-ANSIBLE-REFACTORING-AND-STATIC-ASSIGNMENTS-IMPORTS-AND-ROLES/edit/main/README.md) can be extended.

We began the project by creating and moving into a new branch **`dynamic-assignments`**. Here, we created the necessary folders and files and then inputted the necessary configuration to set our variables. Importantly, in our configuration, we used **`include_vars`** syntax instead of **`include`**, as the  **`include module`** has been deprecated since Ansible version **2.8**, and variants of **`include_*`** must be used. We made use of special variables that will help Ansible dynamically resolve the name of the inventory file being used and that will also help to determine the location of the running playbook, and from there navigate to other paths on the filesystem. We included the variables using a loop. **`with_first_found`** which implies that, looping through the list of files, the first one found is used. After, this we updated the **`site.yml`** file to make use of the dynamic assignment and then we staged, committed and pushed all the changes we made locally in our branch to our remote Github repository. In our repository, we initiated a pull request and merged the **`dynamic-assignments`** branch to our main branch.

In the next phase of this project, we began working with community roles. We created and moved into **`roles-feature`** branch. We then browsed available community roles in [ansible-galaxy](https://galaxy.ansible.com/ui/). We decided to use a [MySQL role](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) developed by **`geerlingguy`**.The next thing we did was to configure Mysql Ansible Role. We did this by creating **`db.yml`** file inside the **`static-assignments`** folder. After putting in the necessary configuration, we moved ahead to reference the **`db.yml`** role inside **`site.yml`**. With the configuration in place inside **`site.yml`**, we proceeded to update the inventory **`ansible-config/inventory/dev.yml`** file with the Private IP address of our DB server. Next, we staged and committed our changes and then we tested the playbook. We ran into a bit of a blocker at this point, but we were able to correct it by fixing a configuration issue and our playbook ran successfully.

The final phase of this project was to demonstrate the implementation of load balancer roles. We needed to be able to make a choice on which Load Balancer to use, Nginx or Apache, so we created two roles respectively: Nginx and Apache with by installing **`geerlingguy`** ansible-galaxy roles as we did earlier. To make Nginx and Apache function as a loadbalancer, we updated the **`default/main.yml`** file inside the Nginx and Apache roles with the private IP address of our two UAT webservers. And since we cannot use both Nginx and Apache as load balancer at the same time, we had to  declare the variables in the **`default/main.yml`** file (inside the Nginx and Apache roles) to enable either one depending on preference. Next, we updated **`static-assignements`** by creating and configuring the **`loadbalancers.yml`** file, we updated the entry point for our playbooks **`site.yml`** with the necessary configuration and we  we updated our **`inventory/dev.yml`** file with the private IP address of the load balancer server. We completed our configuration by using the **`env-vars/dev.yml`** file to define which load balancer to use in the UAT environment. We did this by switching the respective environmental variables settings for **`nginx`** and **`apache`** to **`true`** or **`false`** depending on which is in use. We tested this out for both load balancers and our playbook ran successfully. To wrap up the project in a nice tidy bow, we we staged, committed and pushed all the changes we made locally in the **`roles-feature`** branch to our remote Github repository. Here we created a pull requestand merged the changes to the main branch. Thank you. 
