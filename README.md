# Ansible Collection - custom.isam

This is an example collection, how to deploy your own collections, while using the ibm.isam collection as a dependency.

Although it is possible to push your own modifications back to the master ibm.isam collection, this may be a lenghtly process.


It allows you to 
- override roles in the ibm.isam collection 
- add roles specific to this implementaion, or alternative roles
- use Ansible Tower with your custom collection, while using ibm.isam collection

You should separate your inventories from your playbooks , so they should be in a separate git repo (although you can bundle them in the collection)

Note that making a collection may be a little bit too much.
You can also use a requirements.yml file to import the ibm.isam collection in your playbooks instead.

## Galaxy.yml
In galaxy.yml, you can define the dependency to the ibm.isam collection (or any other collection you need)

You can and should define a version, but "*" means "any version". 
```
dependencies: {
  ibm.isam: "*"
}
```

## Override roles
The way to override roles, requires you to drop the namespace from the roles.

The roles shouldn't be namespaced anyway, but it can make sense to use the namespace (to make sure you are using the correct version).

## Override handlers
It's possible to override the handlers, but it's a bit trickier.  It appears the

