# Access Control systems

* DAC: discrete access based on groups/users permissions. The unix file attributes user/group/all is DAC.
* MAC *(Mandatory AC)*: constraints on doing certain operations or accessing resources. SELinux and AppArmor to fit that description. cgroups seems too, but I'm not sure.
* RBAC *(role based AC)*: based on 3 components: permissions, privileges *(just a grouping of permissions)* and a role *(a list of users and a list of privileges)*. Example: there can be 2 permissions: read/write/delete mounts and managing an FTP service. They can be grouped into "FTP admins" privilege. Then it may be added to "admin" role. The privilege may be added to many roles and a role may have multiple privileges.
* ABAC *(attribute-based AC)*: authorizes a user operation based on attributes assigned to the user, the object it wants to operate upon, the operation and potentially environmental characteristics, such as the current time.
