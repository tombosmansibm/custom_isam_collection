# Ansible Collection - custom.isam
## Description
This is an example collection, how to deploy your own collections, while using the ibm.isam collection as a dependency.

Although it is possible to push your own modifications back to the master ibm.isam collection, this may be a slow process.

This approach allows you to 
- override roles in the ibm.isam collection (in playbooks)
- add roles specific to this implementaion, or alternative roles
- use Ansible Tower with your custom collection, while using ibm.isam collection

You should separate your inventories from your playbooks , so they should be in a separate git repo (although you can bundle them in the collection)

Note that making a collection may be a little bit too much for your use case.  
You can also use a requirements.yml file to import the ibm.isam collection in your playbooks instead.

Anyway, the end result is a collection that is specific to *your* environment, that you can host on your Git repository of your choice (internal or external) and that you can deploy.

## Links
You do need to prepare your Ansible environment to run playbooks for ISAM (and you need an ISAM or IBM Security Verify Access machine).
- IBM Security isam-ansible-collection: https://github.com/IBM-Security/isam-ansible-collection
- Ansible Collections: https://docs.ansible.com/ansible/latest/user_guide/collections_using.html
- ISAM Automation cookbook: https://www.ibm.com/support/pages/node/6348868

## Create a collection
### Initialize the collection
```
ansible-galaxy collection init custom.isam
```
This creates the directory structure.

### Edit Galaxy.yml
In galaxy.yml, you can define the dependency to the ibm.isam collection (or any other collection you need)

You can and should define a version, but "*" means "any version". 
```
dependencies: {
  ibm.isam: "*"
}
```

To define a specific version, you can use something like this:
```
dependencies: {
  ibm.isam: ">=1.0.0"
}
```

Update the version as well, when you make larger changes.

### Add your roles and playbooks
At this point, the actual work begins.
Playbooks go in a "playbooks" directory, Roles go in the "roles" directory.

Since this is an enhancement of the ibm.isam collection, you can start from there and copy the roles as starting point for new roles or to enhance them.

#### roles

- handlers: the handlers are modified to show that they're executed (although they may have no effect whatsoever)
- base/get_static_routes: a role that does not exist in ibm.isam
- base/configure_time: a role that exists in ibm.isam, but is modified (it will pause)

#### playbook

The playbook references :
- a  role from ibm.isam
- an overriden role in custom.isam that also exists in ibm.isam
- a new role in custom.isam that does not exist (yet) in ibm.isam
Notice the "collections" only contains "custom.isam"

```
- hosts: "all"
  gather_facts: no
  collections:
    - custom.isam
  tasks:
    - name: Perform First Steps
      tags: ["first", "steps"]
      include_role:
        name: ibm.isam.base.first_steps
      vars:
        first_steps_admin_pwd: True
#
# This is a role without FQCN
#   It will first look in this collection and then in the ibm.isam collection
#
    - name: Configure NTP
      tags: ["ntp"]
      include_role:
       name: base.configure_time
#
# This uses a role from the "custom.isam" collection, that does not exist in ibm.isam
#
    - name: Show configured static routes (from custom.isam)
      tags: ["ipv4"]
      include_role:
        name: custom.isam.base.get_static_routes
```

### Push to git
You should push your collection to your Github repository (Git, Gitlab, Bitbucket, .... ), to make it available.

### Build
In the collection's directory, you can build the collection

```
ansible-galaxy collection build --force
```

Building generates a tarball that contains the collection.
### Publish to an Ansible Galaxy server
You could publish your collection to an Ansible Galaxy server, but in the scope of this use case, that is not necessary.
So if your company has an Ansible Galaxy server, by all means , set it up.

### Install
Installing a collection is the easiest if your collection is published on an Ansible Galaxy server, but it's not a must.

#### tarbal
From the tarbal that you created earlier during build, you can install the collection:
Note that I think the "force" parameter should not be necessary if I install a newer version, but it is at the moment.

```
ansible-galaxy collection install /home/tbosmans/ansible/collections/custom_isam_collection/custom-isam-1.0.3.tar.gz --force
```

#### git
You can also install the collection directly from git, and that makes the most sense, definitely during development.
```
ansible-galaxy collection install git@github.com:tombosmansibm/custom_isam_collection.git
```

# Advantages
## Override roles in playbooks
The way to override roles, requires you to drop the namespace from the roles.

Although the recommendation is to always use the FQCN (Fully Qualified Collection Name), not using it allows you to easily override roles in your playbooks.

**_NOTE:_**  this does not always work as expected, that's why it's adviced to use the FQCN!    I refer to the Ansible documentation.

## Add your custom roles
This is the main usecase for building your own collection.  
You can add your own roles here .
Again, do not "namespace" your roles, so you can easily and automatically switch to a different version later.

**_NOTE:_** that there are differences between using collections in roles vs. playbooks ! 

## Override handlers
Overriding handlers does not seem to be possible.  If you add the same handlers, all handlers in both namespaces are executed.
This may have the expected result for you , but it may just as well not do what you want.

In this collection, I have copied in the common_handlers role, and it is executed.  But at the same time, the common_handlers from "ibm.isam" are also executed.   Since the "ibm.isam" handlers are executed *before* the "custom.isam" handlers, this likely dos not do anything.

## Run playbooks directly from your collection
### Future
**_NOTE:_** This is not available yet, this will come starting with Ansible 2.11

```
ansible-playbook custom.isam.playbooks.dev-base-setup -i <your inventory>
```

### Today
A bit clunky, but you can run the playbooks from the collection.   It depends on where you install your collections (there's defaults and options).

List the installed collection.
```
ansible-galaxy collection list custom.isam
```

This shows you the path where the collection is actually installed
```
# /home/tbosmans/.ansible/collections/ansible_collections
Collection  Version
----------- -------
custom.isam 1.0.3
```

Add the path of the collection to the base path (replacing '.' with '/' ).
Now you can access the playbook in the collection like a normal playbook.

```
ansible-playbook /home/tbosmans/.ansible/collections/ansible_collections/custom/isam/playbooks/dev-base-setup.yml -i <your inventory>
```
**_NOTE:_** To run the ibm.isam collection, you do need to install the *ibmsecurity* pip package.  See the documentation:  https://github.com/IBM-Security/isam-ansible-collection


## Custom modules
Creating your own custom collection also enables you to include any custom modules you want, in the plugins directory.
This enables you to add your own custom filter plugins, or dynamic inventory plugins that are really specific to your situation.

It's a bit trickier to override or add to the ibmsecurity python code, however.  This is because the plugins in ibm.isam collection dynamically lookup modules from the ibm.security package, and you guessed it, it's hardcoded to ibm.security .

So there's still multiple ways to add your own code.
### Modify the python code in site-packages
Install the ibmsecurity python package into your environment, where it will reside by default in the site-packages location.
Perform your changes there.

This approach obviously has the disadvantage that your changes will be overwritten again if you install a newer version of the ibmsecurity pip package.

### Add a custom path to your PYTHON_PATH



### Update ibmsecurity


## Ansible tower
TODO:  So far I haven't tried this out on Tower, it may be necessary to also add a "requirements.yml" .
