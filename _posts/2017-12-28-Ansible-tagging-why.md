---
layout: post
status: preview
published: true
title: Ansible tagging - why?
author:
  display_name: iguy
  login: iguy
  email: iguy@ionsphere.org
  url: ''
author_login: iguy
author_email: iguy@ionsphere.org
date: 2017-12-28
syntax_highlighting:
  enabled: true
  css: main.css
categories:
- ansible
tags:
- scripts
comments: []
---
In recent times I have been utilizing Ansible nearly constantly for a variety of automated deployments from Openstack, AWS and Azure.
Through this there have been a variety of different approaches I've run into on how to structure Ansible Playbooks.  Some have unique playbooks for each different status change by having a playbook to start the enviornment and one to stop it.  Others have a single extra_var that is passed in to determine the state of the environment and that handles starting and stopping the space.   I've even seen the state of the enviornment be split out into different playbooks in different repos.   

The one that I simply don't get is using tagging in ansible for starting/stopping/installing the environment.  In this design there is a single playbook that is used for all the different states an environment can be in.   Example of it would be something like this from a command line:

```bash
ansible-playbook -i inventory MyApp.yml --tags start,common,assigntags,security
```
In this you run everything that is marked with start, common, assigntags and security.  Along with that the un-tagged items always run.

Pros to using Tagging:

1. Allow skipping over a given sections (plays or tasks) of the playbook

Cons to using Tagging:

1. Easy to create spagetti code
1. Difficult to debug as more tags are introduced and their interactions

Example:  

```yaml
[...]
- name: My first action
  hosts: appserver
  roles: app-make-it-go
  tags: 
      - common, security, start
      
- name: My second action
  hosts: appserver
  roles: app-configure
  tags:
      - common
```

Now this says if you pass in the `security` tag, only the first play will execute.   

What if the role has some tasks that are marked with `common` and you didn't pass that in to that first task?   

How does the caller know that some steps won't run while others will?   

I spent 6 hours debugging one odd issue I was having working with an existing playbook that utilized tags and ultimately I ended up removing all the tags and creating purpose built playbooks to run instead.   This allowed me to follow the logic clearly and find that the issue was one caused because the role had tags and was expecting a certain state when that task ran.  Because of the tags used, that state was skipped in a variety of situations.   

I highly encourage folks who build ansible playbooks to think two, three, four, five times about using tags.  It might make it easy when the playbook is first created when everything and every flow is in your head.  It will more likely make whoever has to fix have to unwind the complexity added to the playbook to save a couple files.  

Please.. think of the maintainers.  





