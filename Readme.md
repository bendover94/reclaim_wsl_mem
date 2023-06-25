# This is the `reclaim_wsl_mem` Project
created by Lucas Schmirl on: 25.06.2023, last edit: 25.06.2023

<br>

Contact me: [E-Mail](mailto:info.hellgineer@gmail.com?subject=User%20Question&body=I%20like%20your%20code%20man,%20keep%20it%20up.)


Check out my Homepage: [Hellgineer](https://hellgineer.com)

---

For what reason ever WSL allocated the default mem size of 1TB on my C drive.

![allocated_mem](/IMGs/allocated_mem.png)

Unfortunately the 2 recommended tasks from the internet:
```bash
optimize-vhd -Path .\ext4.vhdx -Mode full
```

and 
```bash
diskpart
select vdisk file="C:...\ext4.vhdx"
compact vdisk
```

**Did not work for me.** (because of the missing defragmentation)

Altough the progress bar was visible and in both approaches, it finishes at ~30% with "sucess" / no error.

The wsl-allocated memory on my C drive was not gained back.

- I use windows-11 pro with WSL2
- my ssd is 2tb so wsl2 was taking:
    - virtual: 1024 GB
    - physical:  960 GB

even tough the actual size of WSL was just 29GB.

To reclaim that memory to be usable for windows again, you need to execute the following steps. (worked for me)

---
<br>

<br>

### 1. Just for information how much space is really used but spread over the 1TB type inside WSL:

```bash
du -h | sort -h
```
![real_mem](/IMGs/show_real_mem.png)

<br>

--- 

<br>

### 2. Defragment the memory that WSL is currently allocating. (set all empty blocks to 0) type inside WSL:
```bash
sudo dd if=/dev/zero of=/0bits bs=20M
```

then you should get a similar response after a while:

```bash
dd: error writing '/0bits': No space left on device
50261+0 records in
50260+0 records out
1054042726400 bytes (1.1 TB, 982 GiB) copied, 468.129 s, 2.3 GB/s
```

<br>

--- 

<br>

### 3. Shutdown WSL (from powershell)
```Powershell
wsl --shutdown
```

<br>

--- 

<br>

### 4. Optimize (Re-organize the memory)
as admin type in **powershell**: (replace your path to .vhdx file)
```powershell
Optimize-VHD C:\Users\lucas\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu20.04onWindows_79rhkp1fndgsc\LocalState\ext4.vhdx -Mode Full
```

When the progress bar is done, your mem on C should be reclaimed by windows again.

At the next start of WSL I had an **error:** `fallback read-only`

<br>

--- 

<br>

### 4. To resolve this error, follow this [Guide](https://learn.microsoft.com/en-us/windows/wsl/disk-space#how-to-repair-a-vhd-mounting-error)

During the guide I had an error when running
```powershell
wsl lsblk
```

**Error:** "mounting error, device is used by another process"

I did 
```powerhell
wsl --shutdown
```

which had no effect because WSL was already turned off.

Also I diconnected the device in Hyper-V-manager (maybe vodoo but after this, the guide worked somehow)

<br>

--- 

<br>

### 5. Restart WSL
I got another error, now the device was not read-only anymore but full.

**error:** "OSError: [Errno 28] No space left on device"

after running
```bash
df -h
```

I recogniced that the dev/sdc was full (because of the previous written 0bits)

![sdc_full](/IMGs/sdc_full.png)

So I did:
```bash
sudo rm /0bits
```

And checked it with
```bash
df -h
``` 

Now the device was freed again.

![sdc_freed](/IMGs/sdc_freed.png)

After restarting WSL again, it had no further errors.

For further safety I turn off SWAP 
```bash
sudo swapoff -a
```

