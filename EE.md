## DOCKER EE



### Create a Wordpress application which utilizes secrets and define RBAC for the application.

- Create a secret.
  swarm>> secret >> Create
- Create a network `wordpress-network` for service stack.
    - swarm >> network >> Create  (use default settings)
- Create a MySql service with name `wordpress-db`:
  - Swarm >> Services >> Create Service. Select Task Template and use the “mysql:5.7” image.
  - Use default `collection set`
  - Network >> attach network >> selecet the network we previously created.
  - Environment >> Secret >> select the secret previously created.
                >> environment variable >> "MYSQL_ROOT_PASSWORD_FILE=/run/secrets/wordpress"

- Create a `wordpress` service.
  - Swarm >> Services >> Create Service. Select Task Template and use the “wordpress:latest” image.
  - Use default `collection set`
  - Network >> attach network >> selecet the network we previously created.
  - Environment >> Secret >> select the secret previously created.
                >> add environment variable >> "WORDPRESS_DB_PASSWORD_FILE=/run/secrets/wordpress"
                                            >> "WORDPRESS_DB_HOST=wordpress-db:3306"
                                            
###  RBAC in Docker Swarm.

-  **collection** is a group of swarm resources(resource collection), like services, containers, volumes, networks, and secrets. Access to collections goes through a directory structure that arranges a swarm's resources. To assign permissions, administrators create grants against directory branches. DOccker EE enables controlling access to swarm resources by using collections and Kubernetes resources with namespaces.
- **Grants** defines how users and organizations can access resource sets. A grant defines who has how much access to what resources. Each grant is a 1:1:1 mapping of subject, role, and resource set.
- **Users**  similar to SA , user can be admin or nona admin user.
- **Teams**  Group of users
- **Oragnization** Collection of teams
- **Roles** Permission levels 


- Add and configure subjects (users, teams, and service accounts).
- Define custom roles (or use defaults) by adding permitted operations per type of resource(collection).
- Group cluster resources into resource sets of Swarm collections.
- Create grants by combining ***subject + role + collection***


- Create a non admin user `apptester`
- Create a team `appteam`
- Create an organization `app`.
- Add `apptester` to team `appteam`.
- Create a collection `appresource`
- Move one of service to `appresource`
- Check if `apptester` can see service.
- Create a grant `viewonly` for team `appteam` to `appresource`
- Check if `apptester` can see service.
- Try to edit the service.



 








