# Dockerfile Instructions

A comprehensive list of Dockerfile instructions reorganized by category, with usage descriptions and multiple examples.

## Categories
* **Base & Meta:** FROM, LABEL
* **Variables:** ARG, ENV
* **Execution Control:** CMD, ENTRYPOINT, RUN
* **Filesystem & Copying:** COPY, ADD, WORKDIR, VOLUME
* **Networking & User:** EXPOSE, USER
* **Advanced:** ONBUILD, STOPSIGNAL, HEALTHCHECK, SHELL

## Base & Meta
Instructions used to initialize the build and add metadata.

### FROM
**Use:** Initializes a new build stage and sets the Base Image for subsequent instructions. A valid Dockerfile must start with a `FROM` instruction.

**Examples:**
*Basic usage:*
```dockerfile
FROM ubuntu:20.04
```
*Using an alias for multi-stage builds:*
```dockerfile
FROM node:14 AS builder
```
*Using a digest for immutable builds:*
```dockerfile
FROM alpine@sha256:ac6b...
```

### LABEL
**Use:** Adds metadata to an image. A LABEL is a key-value pair.

**Examples:**
*Single label:*
```dockerfile
LABEL version="1.0"
```
*Multiple labels in one instruction (preferred for layer optimization):*
```dockerfile
LABEL version="1.0" \
      description="My App" \
      maintainer="srikanth@example.com"
```

## Variables
Instructions for defining variables used during build or runtime.

### ARG
**Use:** Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag. These variables are only available during the build process, not in the final container.

**Examples:**
*With a default value:*
```dockerfile
ARG VERSION=latest
FROM alpine:$VERSION
```
*Without a default value (must be passed at build time):*
```dockerfile
ARG BUILD_DATE
LABEL build_date=$BUILD_DATE
```

### ENV
**Use:** Sets the environment variable `<key>` to the value `<value>`. These variables persist in the final container and are available at runtime.

**Examples:**
*Setting a single environment variable:*
```dockerfile
ENV APP_ENV=production
```
*Setting multiple environment variables:*
```dockerfile
ENV APP_HOME=/usr/src/app \
    PORT=8080
```
*Using an ENV variable in subsequent instructions:*
```dockerfile
WORKDIR $APP_HOME
```

## Execution Control
Instructions that define what runs during access and how the container starts.

### RUN
**Use:** Executes any commands in a new layer on top of the current image and commits the results. The resulting committed image is used for the next step in the Dockerfile.

**Examples:**
*Shell form (runs in /bin/sh -c):*
```dockerfile
RUN apt-get update && apt-get install -y python3
```
*Exec form (avoids shell string munging, preferred):*
```dockerfile
RUN ["/bin/bash", "-c", "echo hello"]
```

### CMD
**Use:** Provides defaults for an executing container. There can only be one CMD instruction in a Dockerfile. If you provide more than one, only the last one takes effect. It can be overridden by arguments passed to `docker run`.

**Examples:**
*Exec form (preferred):*
```dockerfile
CMD ["python3", "app.py"]
```
*Shell form:*
```dockerfile
CMD python3 app.py
```
*As default parameters to ENTRYPOINT:*
```dockerfile
ENTRYPOINT ["/bin/echo"]
CMD ["Hello world"]
```

### ENTRYPOINT
**Use:** Configures a container that will run as an executable. Command line arguments to `docker run <image>` will be appended after all elements in an exec form ENTRYPOINT, and will override all elements using CMD.

**Examples:**
*Exec form (preferred):*
```dockerfile
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
```
*Shell form (ignores any CMD or docker run arguments):*
```dockerfile
ENTRYPOINT /usr/sbin/nginx -g "daemon off;"
```

## Filesystem & Copying
Instructions for manipulating files and directories.

### COPY
**Use:** Copies new files or directories from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

**Examples:**
*Copying a single file:*
```dockerfile
COPY requirements.txt /tmp/
```
*Copying multiple files to a directory:*
```dockerfile
COPY src/ /app/src/
```
*Copying from a previous build stage (Multi-stage build):*
```dockerfile
COPY --from=builder /app/build /usr/share/nginx/html
```
*Changing ownership during copy:*
```dockerfile
COPY --chown=user:group files/ /app/files/
```

### ADD
**Use:** Similar to COPY, but has additional features: it can extract local tar archives and fetch files from remote URLs. START with COPY, use ADD only if you need these features.

**Examples:**
*Extracting a local tarball:*
```dockerfile
ADD resources.tar.gz /usr/share/resources/
```
*Downloading a remote file:*
```dockerfile
ADD https://example.com/config.json /app/config.json
```

### WORKDIR
**Use:** Sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it. If the directory doesn't exist, it will be created.

**Examples:**
*Setting an absolute path:*
```dockerfile
WORKDIR /path/to/workdir
```
*Setting a relative path (relative to previous WORKDIR):*
```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
# Output: /a/b/c
```

### VOLUME
**Use:** Creates a mount point and marks it as holding externally mounted volumes. It allows data to persist even if the container is removed.

**Examples:**
*Defining a single volume:*
```dockerfile
VOLUME ["/data"]
```
*Defining multiple volumes:*
```dockerfile
VOLUME ["/var/log", "/var/db"]
```

## Networking & User
Instructions for network settings and user permissions.

### EXPOSE
**Use:** Informs Docker that the container listens on the specified network ports at runtime. It does NOT actually publish the port; you must still use `-p` when running the container.

**Examples:**
*Exposing a default TCP port:*
```dockerfile
EXPOSE 80
```
*Exposing UDP and TCP:*
```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

### USER
**Use:** Sets the User name (or UID) and optionally the User Group (or GID) to use for the remaining instructions and when running the container.

**Examples:**
*Switching to a specific user:*
```dockerfile
USER patrick
```
*Switching via UID:GID:*
```dockerfile
USER 1001:1001
```

## Advanced
Instructions for special build behaviors and health checks.

### ONBUILD
**Use:** Adds to the image a trigger instruction to be executed when the image is used as the base for *another* build.

**Examples:**
*Adding files in a child image:*
```dockerfile
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```

### STOPSIGNAL
**Use:** Sets the system call signal that will be sent to the container to exit. Default is SIGTERM.

**Examples:**
*Setting a custom stop signal:*
```dockerfile
STOPSIGNAL SIGKILL
```

### HEALTHCHECK
**Use:** Tells Docker how to test a container to check that it is still working.

**Examples:**
*Checking a web server:*
```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
*Disabling healthcheck inherited from base image:*
```dockerfile
HEALTHCHECK NONE
```

### SHELL
**Use:** Allows the default shell used for the shell form of commands to be overridden. Linux default is `["/bin/sh", "-c"]`.

**Examples:**
*Using bash with strict mode:*
```dockerfile
SHELL ["/bin/bash", "-c"]
RUN echo "This runs in bash"
```
*Using PowerShell on Windows:*
```dockerfile
SHELL ["powershell", "-command"]
RUN Write-Host "This runs in powershell"
```
