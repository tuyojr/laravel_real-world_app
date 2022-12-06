#

I had to redo the project in a more industry standard manner after corrections were made by our instructor Valentine Madu (@OdogwuVal on github).

![Altschool Africa](https://github.com/tuyojr/cloudTasks/blob/main/logos/AltSchool.svg)

![automation](https://github.com/tuyojr/cloudTasks/blob/main/logos/automation.gif)

> This project's instructions can be found [here](https://docs.google.com/document/d/1dhO--9fqxrvU5Y6xC52-0AIplQe23LyscqSp6F0_9SM/edit>#). The repository used for deployment is also [here](https://github.com/f1amy/laravel-realworld-example-app).

Below is my main ansible playbook used in deploying this project, which was live at [www.tuyojr.me](https://www.tuyojr.me/) at some point in October/November. All other files, variables and secrets used in the playbook are present in the directory above.

`main.yml`

```YAML
# This playbook was written by Originally Adedolapo Olutuyo ALT/SOE/022/4858. ©October-November 2022
# To apply and solidify my knowledge on everything taught during the 
# 2nd Semester of the Cloud Engineering Course at Altschool Africa, in the real world.
# If you're reading this, MAY THE FORCE BE WITH YOU!

- hosts: all
  pre_tasks:
    - name: update cache
      apt:
        update_cache: yes
  roles:
    - name: Install some necessary basic tools
      role: tools
    - name: Install and configure apache2
      role: apache2
    - name: Install and configure mysql
      role: mysql
    - name: Install php and its dependencies
      role: php
    - name: Add necessary ufw rules and enable ufw
      role: ufw
    - name: Clone, install project dependencies and deploy the project
      role: project
    - name: Install ssl certificate
      role: ssl
    - name: Install Postgresql, create user and database
      role: postgresql

# My configurations are a bit crude, I will commit to working better. Thank you for making it to the end.
# Again, MAY THE FORCE BE WITH YOU!
```

![thankYou](https://github.com/tuyojr/cloudTasks/blob/main/logos/thankYou.gif)
