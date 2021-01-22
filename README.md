# Ansible Collection - custom.isam

This is an example collection, how to deploy your own collections, while using the ibm.isam collection as a dependency.

Although it is possible to push your own modifications back to the master ibm.isam collection, this may be a lenghtly process.


It allows you to 
- override roles in the ibm.isam collection 
- add roles specific to
- use Ansible Tower with your custom collection, while using ibm.isam collection

You should separate your inventories from your playbooks , so they should be in a separate git repo (although you can bundle them in the collection)

Note that making a collection may be a little bit too much.
You can also use a requirements.yml file to import the ibm.isam collection in your playbooks instead.

