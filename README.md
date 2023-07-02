# Apache Zeppelin ENV
```bash
cd docker/

# Build image
# - docker build -t <tag name> .
docker build -t zeppelin .

# Run docker container
# - docker run -p 8080:8080 --rm -v <path to mount(local path)>:<path to be mounted(path in docker container)> --name <container name> <tag name>
docker run -p 8080:8080 --rm -v $PWD/notebook:/opt/zeppelin/notebook --name zeppelin zeppelin

# Enter docker container
# - docker exec -it <container name> /bin/bash
docker exec -it zeppelin /bin/bash
```
