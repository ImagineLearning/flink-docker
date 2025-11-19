# flink-docker / dev-master

## Building a custom docker image

The commands shown during these steps serve as an example and assume that you have checked out `flink` and `flink-docker`
in the same folder and version 1.11. Please substitute your folder structure and version.

1. Compress Flink in `flink/flink-dist/target/flink-1.11-SNAPSHOT-bin`: `tar czf flink-1.11.tgz flink-1.11-SNAPSHOT`
2. Copy the compressed distro to this project's root: `cp flink/flink-dist/target/flink-1.11-SNAPSHOT-bin/flink-1.11.tgz flink-docker`
3. Start web server ``docker run -it -p 9999:9999 -v `pwd`:/data python:3.7.7-slim-buster python -m http.server 9999``
4. Generate `Dockerfile` `./add-custom.sh -u http://localhost:9999/data/flink-1.11.tgz -n flink-1.11`
    (If you are on a Mac or Windows, use `host.docker.internal` instead of `localhost`)
    * If you want to build the docker image inside Minikube, then you have to specify the resolved `host.minikube.internal` which you can look up via `minikube ssh "cat /etc/hosts"`.
5. Generate docker image (in `flink-docker/dev/flink-1.11-debian`): `docker build -t flink:1.11-SN .`
6. Run custom Flink docker image: `docker run -it flink:1.11-SN jobmanager`

### Build without running a containerized web server (skip step 3, above)

4. Generate the `Dockerfile` via: `./add-custom.sh -u file:///build-ctx/flink-1.11.tgz -n flink-1.11`.  
    Note that the URL must begin with `file:////build-ctx/`
    Next, copy the tgz file into the build context directory created for your custom build: `cp flink-1.11.tgz -n dev/flink-1.11-debian/.`  
    Proceed with step 5, above.

#### Build with Podman and without running a containerized web server (skip step 3, above)

Podman doesn't complain about `RUN --mount ...` but then triggers permission denied when accessing files in the mounted volume.  Here's the workaround...

4. Generate the `Dockerfile` via: `./add-custom.sh -u file:///build-vol/flink-1.11.tgz -n flink-1.11`.  
    Next, copy the tgz file into the build dir created for your custom build: `cp flink-1.11.tgz -n dev/flink-1.11-debian/.`  
5. Generate docker image (in `flink-docker/dev/flink-1.11-debian`): `docker build -v $PWD:/build-vol -t flink:1.11-SN .`    
