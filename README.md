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

By default, the collections go into .ansible/collections/ansible_collections and this is **shared across your virtual environments** .  You can however, change the collections path.

#### tarbal
From the tarbal that you created earlier during build, you can install the collection:
Note that I think the "force" parameter should not be necessary if I install a newer version, but it is at the moment.

```
ansible-galaxy collection install /home/tbosmans/ansible/collections/custom_isam_collection/custom-isam-1.0.3.tar.gz --force
```

#### git
You can also install the collection directly from git, and that makes the most sense, definitely during development.
Same remark about 'force'.
```
ansible-galaxy collection install git@github.com:tombosmansibm/custom_isam_collection.git --force
```

**_TIP_**  If you recieve an error, it means your ansible(-galaxy) version is not up to date.
Install ansible (pip install --upgrade ansible) into the virtual environment and log out (deactivate) and back in.

> Process install dependency map
> ERROR! Invalid collection name 'git@github.com', name must be in the format <namespace>.<collection>. 
> Please make sure namespace and collection name contains characters from [a-zA-Z0-9_] only.

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
An alternative is to create a link to the location (that's what is actually proposed in the **IBM Security Verify Access - IBM automation cookbook** https://www.securitylearningacademy.com/enrol/index.php?id=5578)
Note that, specifically for playbooks, there is not much value at this moment to put them in your own collection until Ansible 2.11 is available .

List the installed collection.
```
ansible-galaxy collection list custom.isam
```

This shows you the path where the collection is actually installed
```
# /home/tbosmans/.ansible/collections/ansible_collections
Collection  Version
----------- -------
custom.isam 1.0.6
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

### Modify the python code for ibmsecurity in site-packages directory
Install the ibmsecurity python package into your environment, where it will reside by default in the site-packages location.
Perform your changes there.

This approach obviously has the disadvantage that your changes will be overwritten again if you install a newer version of the ibmsecurity pip package.

### Create a python package
This is a more robust approach, that also easily allows you to switch back to code in the original "ibmsecurity" package.
#### Create a similar structure as the ibmsecurity package
Under docs/samples/site-packages/ , I've added a custom package "tbosmans" (you find it under "src").
I've created it by copying the ibmsecurity package folder, and then stripping away everything I didn't need.

In this example, I'm overriding the isam.base.date_time package, with my own code.
This example only contains the **single function** that I want to override, although I did start from a copy of the original **data_time.py**.

The timezones retrieved from the appliance are ignored, and overriden with a bogus timezone named "Home" .

#### Pip
The sample structure here also contains the structure necessary to upload this to pip.
See https://packaging.python.org/tutorials/packaging-projects/ for more information on how to do that.

For this demo , I uploaded the package that is in this repo already to the test index of test.pypi.org : https://test.pypi.org/manage/project/tbosmans-isam-demo, so it is available .

So first, create a new, clean virtual environment (for example with python 3.9)
```
virtualenv ~/venvtmp --python=python3.9
. ~/venvtmp/bin/activate
```

Then install the python prerequisites in that virtual environment:
```
pip install ansible
pip install ibmsecurity
```

Then install this demo package from test.pypi.org.
Use --no-deps, so pip doesn't try to resolve dependencies on test.pypi.org (you want the dependencies from the "production" environment, so in this case it means you have to install "ibmsecurity" prior to running this).
```
pip install --upgrade --index-url https://test.pypi.org/simple/ --no-deps tbosmans-isam-demo
```
**_NOTE_** In real life, you probably don't want to upload your overrides to (public) pypi.org, but if you build the package, you can still install it using pip with the tarball in the *dist* directory.  You could obviously distribute this tarball any way you like to your systems.

```
pip install --upgrade <path to this git repo on your disk>/docs/samples/site-packages/dist/tbosmans-isam-demo-1.0.0.tar.gz 
```

The custom.isam collection also has to be installed, but we did that earlier, and it's shared accross virtual environments.

#### Ansible modules
The Ansible module that is provided in the "ibm.isam" collection, contains some code that dynamically looks up the package and functions.
Unfortunately, it contains a check to only process "ibmsecurity.isam" code.

```
   # Dynamically process the action to be invoked
    # Simple check to restrict calls to just "isam" ones for safety
    if action.startswith('ibmsecurity.isam.'):
    ...
```
So we need to provide our own copy of this module, that also works for our custom package "tbosmans.isam".
Copy the plugins/modules/isam.py file from the ibm.isam collection into this structure.

```
└── plugins
    └── modules
        └── isam.py
```

In this module, I added an elif statement to also look in my own Python package.

```
    elif action.startswith('tbosmans.isam.'):
        # This allows me to call my own modules
        ...
```


#### Role
The custom role ties the custom module to the plays.

There's 2 things to notice here:
- module name: if you use a namespace, it should be the "custom.isam" namespace, not the "ibm.isam" namespace.  In this case, I remove the namespace, so it simply reads "isam".  This is necessary, so the module in this collection is used, where we modified the dynamic lookup of the acton.
- action: the action needs to point to the python package, so in this case "tbosmans.isam" (instead of "ibmsecurity.isam")


<pre>
...
- name: Retrieve a bogus list of timezones
  <b>isam</b>:
    log: "{{ log_level | default(omit) }}"
    force: "{{ force | default(omit) }}"
    action: <b>tbosmans.isam</b>.base.date_time.get_timezones
    isamapi: 
  register: timezones_obj
...
</pre>

It's obviously not hard to simply change "tbosmans.isam" to "ibmsecurity.isam" again, to start using the "official" code again.

#### Playbook
In the playbook, by not using the full namespace , the first match will work (so first look in this collection, then look in the ibm.isam collection)

```
    - name: Get timezones
      tags: ["ntp"]
      include_role:
       name: base.get_timezones
```

The result is then the single, bogus timezone named "Home" (instead of a full list of timezones):

To run the playbook:

```
ansible-playbook /home/tbosmans/.ansible/collections/ansible_collections/custom/isam/playbooks/dev-base-setup-module-demo.yml -i <inventory>
```

> PLAY [all] 
> ************************
> TASK [Get timezones] 
> ************************
> TASK [custom.isam.base.get_timezones : Help INFO (-e help=true)] 
> ************************
> skipping: [192.168.151.128]
> 
> TASK [custom.isam.base.get_timezones : Retrieve a bogus list of timezones] 
> ************************
> [WARNING]: THIS IS CUSTOM CODE
> ok: [192.168.151.128]
> 
> TASK [custom.isam.base.get_timezones : debug] 
> ************************
> ok: [192.168.151.128] => {
>     **"msg": "[\n    {\n        \"id\": \"Home\",\n        \"name\": \"UTC+00:00 Home\"\n    }\n]"**
> }
> 
> PLAY RECAP 
> ************************
> 192.168.151.128            : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   


# Ansible tower

**_NOTE_** : I HAVE NOT VERIFIED THIS YET

## Ansible Galaxy

There's support for ansible galaxy in Tower in the later version (2.9+).

It basically works the same for collections as it already did for roles.

You would have to create a requirements.yml file in a 'collections' directory in your playbooks directory that contains the collections you need.
The requirements.yml should look something like this:

```
---
collections:
# ibm.isam
- ibm.isam

# Custom collection, for example from an internal git repo
# This does not work with the namespace (name: custom.isam fails)
- name: git@github.com:tombosmansibm/custom_isam_collection.git
  version: '*'
  type: git
  source: 'git@github.com:tombosmansibm/custom_isam_collection.git'
```  

## Custom collection

I think that in general (for ISAM) a custom collection only brings value for roles and custom modules/packages at this moment for Tower.
Playbooks are better maintained in there separate Git repository, and inventories also should be in a separate git repository.




