# Sonatype Nexus 3+ Artifact Repository (Milestone 7+)

This is a Sonatype Nexus 3+ (Milestone 7+) container for Docker. It will survive for as long as there is no official container for Nexus 3 from Sonatype. This was shamelessly based on the official `sonatype/nexus` container, but soon it became a vastly different Dockerfile.

Usage is the same as ever for a container meant to be a service, port 8081 is exported by default:

```
$ docker run -d -p 8081:8081 --name nexus flisboac/docker-sonatype-nexus
```

`/var/lib/nexus/data` is Nexus' data folder, and `/etc/nexus` is its configuration folder. Both are equivalent to setting `-Dkaraf.data` and `-Dkaraf.etc`, respectively. You may volume-mount them if needed, but be sure to set the environment variable `NEXUS_UID` to avoid running into permission-related issues. For example:


```
$ docker run -d \
  -p 8081:8081 \
  -e NEXUS_UID=$(id -u) \
  -v /path/to/nexus/data:/var/lib/nexus/data
  -v /path/to/nexus/etc:/etc/nexus
  --name nexus flisboac/docker-sonatype-nexus
```

During its execution, the container will run a small setup script before executing the Nexus' server. This setup script will:

- Copy all of the configuration files in `${NEXUS_HOME}/etc` into `/etc/nexus`, but only if the folder is completely empty; this means that, when mounting `/etc/nexus` there is no need to copy configuration files if the mounted folder does not contain them, and that is especially useful if the configuration folder is being mounted for the first time.
- Checks if the environment variable `NEXUS_UID` is set, and if it is, the user on which Nexus will be running will have its UID changed to that indicated by `NEXUS_UID` (e.g. `usermod -u $NEXUS_UID $NEXUS_USER`); also, ownership will be reset for that user on some specific folders and variables, including 
`/var/lib/nexus/data` and `/etc/nexus`.
- Regenerate `nexus.vmoptions` (with all of the JVM flags present by default in the official distribution's `nexus.vmoptions`) and `nexus.rc` (`run_as_user` disabled at the moment because of [a bug](https://issues.sonatype.org/browse/NEXUS-9437), `piddir` is set to `/var/lib/nexus/data`)

As a note, Nexus will generate all of the file/folder structure on `/var/lib/nexus/data` on its first run if it was not previously set up.

A list of all the environment variables that may be set prior to execution:

- `JAVA_MAX_HEAP`: Used to define `-Xmx`.
- `JAVA_MIN_HEAP`: Used to define `-Xms`.
- `NEXUS_UID`: Nexus user's UID. The UID will be reset on every run, useful if using shared volumes. Unconventional, I admit.

There is no easy way to enable SSL/TLS yet. If needed, mount `/etc/nexus` and configure Nexus to use SSL/TLS, or use a reverse proxy.
