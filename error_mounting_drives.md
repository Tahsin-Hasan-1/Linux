# Solution to Error mounting of drives in linux system.

## Works only on debian or debian based distros.

```bash
sudo fdisk -l
```
sudo apt-get install nfs-common

sudo apt-get install cifs-utils

sudo ntfsfix -d /dev/nvme0n1p5


### note: replace dev/nvme0n1p5 in the last command with your desired disk or volume.
