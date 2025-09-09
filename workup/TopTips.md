# Tutorial FAX / Top Tips

## docker name conflict

If you get a container name conflict between examples you might get an error:

``
Error response from daemon: Conflict. The container name "/database" is already in use by container
```
Answer:

```
# List all the running and stopped containers on your system using the following command:

docker ps -a

CONTAINER ID   IMAGE                          COMMAND                  CREATED        STATUS                    PORTS      NAMES
4f2cce0d13b2   opennms/horizon:33.1.6         "/entrypoint.sh -s"      7 days ago     Exited (137) 7 days ago              horizon
6c85026265a5   postgres:15                    "docker-entrypoint.s…"   7 days ago     Exited (255) 7 days ago   5432/tcp   database
2e7426518669   opennms/minion:33.1.6          "/entrypoint.sh -f"      7 days ago     Exited (0) 7 days ago                minion1

# Use the following command to remove the conflicting container

On your project make sure the project is shutdown

docker compose down -v

# you may also need to stop the container
docker stop <container_name>

# remove the container
docker remove <container_name>

```

