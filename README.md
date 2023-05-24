# GPDB 7 Docker Compose with systemd

This is a docker compose repository to build and orchestrate a GPDB 7X
multinode cluster for dev testing along with systemd enabled.
This can be useful when there is a
need to have multiple hosts and the single-node gpdemo cluster is
insufficient. The docker compose GPDB 7 cluster is ephemeral so the
cluster can be discarded and rebuilt for quick and clean testing. The
OS of choice here is Rocky Linux 8.

## Install Podman

Podman is a container engine with the same command-line interface (CLI) as Docker.
Pretty much every command you can run from the Docker command line you can execute with Podman.

For Mac, Podman is provided through Homebrew.

```
brew install podman
brew install podman-compose
```

Next, create and start your first Podman machine:

```
podman machine init
podman machine start
```

You can then verify the installation information using:

```
podman info
```

## Build the image

We need to build a local docker image that will be used for each
container in the multinode cluster. The image here is specifically
designed to work for GPDB 7 compilation and running the cluster.

```
pushd ./build/
podman build -t gpdb7-multinode-cluster/gpdb-image .
popd
```

Note: You only have to build the image once (unless there is an update
to the build directory). The image is stored locally.

## Set environment variables

The `docker-compose.yaml` uses three environment variables:
1. `$GPDB7_SRC` (the path to your GPDB 7X source code)
```
Example:
export GPDB7_SRC=/Users/jyih/workspace/gpdb7
```
Note: The `$GPDB7_SRC` path will be mounted and used for compilation
so it would be good to run `git clean -xfd` in the directory to rid of
anything (e.g. MacOS compiled binaries).

2. `$PWD` (the path to the top-level dir of this repository)
Note: You have to run `docker-compose` in the top-level dir anyways so
`$PWD` should always be correct.

3. `$GPDB_RPM` (the path to the GPDB RPM directory)
Note: This is optional and required only if you want to install GPDB using
an RPM instead of compiling it from source

## Run podman compose

This step will create the GPDB hosts (3 containers), build a network
bridge between the 3 containers, and mount a shared volume (the GPDB 7
source code and some orchestration scripts from this repository).

```
# In this repository's top-level dir
podman-compose up -d
```

## Run coordinate-everything.sh script

This step will install the compiled GPDB 7 onto each host and run
gpinitsystem to create the GPDB 7X cluster.

```
pushd ./scripts/
bash coordinate-everything.sh
popd
```

Alternatively, you can also use GPDB RPM to install GPDB. You would need
to set the `$GPDB_RPM` environment variable to the directory
where the RPM is located.

```
pushd ./scripts/
bash coordinate-everything.sh rpm
popd
```

Afterwards, you'll be able to connect to the coordinator node and
start your testing.

```
podman exec -it gpdb7-docker-compose_cdw_1 /bin/bash
su - gpadmin
source env.sh
```

## Stop and delete everything

This step will destroy everything we've created. Run this when you no
longer need the cluster.

```
# In this repository's top-level dir
podman-compose down -v
```
