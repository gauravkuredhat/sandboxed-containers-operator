
# RVPS (Reference Value Provider Service) Usage

The RVPS (Reference Value Provider Service) values are used for remote attestation.

It is responsible for verifying, storing, and providing reference values. RVPS receives and verifies inputs from the software supply chain, stores the measurement values, and generates reference value claims for the Attestation Service.

This operation is performed based on the evidence verified by the Attestation Service (AS).

## RVPS Values

The values are:

1. `image_phkh`
2. `image_tag`
3. `se.version`
4. `se.tag`
5. `se.attestation_phk`

## Script Options

The script will help retrieve the RVPS via the following two options:

1. **Calculate the RVPS values based on the SE PODVM image stored locally** on the user’s machine where the script is being executed. The script will expect the absolute path of the SE PODVM image.

2. **Calculate the RVPS values based on the SE PODVM image uploaded to a libvirt volume**. The script will expect the following inputs:
    - Libvirt Pool Name
    - Libvirt URI Name
    - Libvirt Volume Name

## Output

After successful execution, you will get `se-message` and `ibmse-policy.rego` in a directory called `output-files`. These files will contain the RVPS parameters.

## Prerequisites
1. The user needs to check if Network Block device ('nbd3') is available or not because the script is written on the basis of 'nbd3'.
Then can check using below and make sure it's size is 0 bytes :- 

```
# lsblk | grep nbd3
# nbd3         43:96   0    0B  0 disk
```

```
In case if it is allocated, try to disconnect it and then progress further.
If it can't be deallocated due to other usage , we can check which 'nbd' is available
:-
[root@a3elp61 ~]# lsblk | grep nbd3
nbd3         43:96   0    0B  0 disk
[root@a3elp61 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
loop0         7:0    0  7.6G  0 loop  /var/www/html/bastioniso
sda           8:0    0    2T  0 disk
`-mpatha    253:0    0    2T  0 mpath
  `-mpatha1 253:1    0    2T  0 part  /var/lib/containers/storage/overlay
                                      /
sdb           8:16   0    2T  0 disk
`-mpatha    253:0    0    2T  0 mpath
  `-mpatha1 253:1    0    2T  0 part  /var/lib/containers/storage/overlay
                                      /
nbd0         43:0    0    0B  0 disk
nbd1         43:32   0    0B  0 disk
nbd2         43:64   0    0B  0 disk
nbd3         43:96   0    0B  0 disk
nbd4         43:128  0    0B  0 disk
nbd5         43:160  0    0B  0 disk
nbd6         43:192  0    0B  0 disk
nbd7         43:224  0    0B  0 disk
nbd8         43:256  0    0B  0 disk
nbd9         43:288  0    0B  0 disk
nbd10        43:320  0    0B  0 disk
nbd11        43:352  0    0B  0 disk
nbd12        43:384  0    0B  0 disk
nbd13        43:416  0    0B  0 disk
nbd14        43:448  0    0B  0 disk
nbd15        43:480  0    0B  0 disk

In this case ,any nbd from  nbd0 to nbd15 can be used. The same 'nbd' user can replace in the script(GetRvps.sh) .  In maximum cases , it will not be required as 'nbd3' will be available always.
```

2. The user needs to copy the script and associated files in the respective lpar. They can follow below steps. 

```bash
Step 1. Create the directory
#mkdir -p Rvps-Extraction/static-files

Step 2. Naviate to the source directory and copy the script
# cd Rvps-Extraction/
# wget https://github.com/openshift/sandboxed-containers-operator/raw/devel/hack/Rvps-Extraction/GetRvps.sh -O $PWD/GetRvps.sh
# chmod +x GetRvps.sh

Step 3. Naviate to the child directory and copy the python extraction script along with pvextract

# cd static-files/
# wget https://github.com/openshift/sandboxed-containers-operator/raw/devel/hack/Rvps-Extraction/static-files/pvextract-hdr -O $PWD/pvextract-hdr
# chmod +x pvextract-hdr
# wget https://github.com/openshift/sandboxed-containers-operator/raw/devel/hack/Rvps-Extraction/static-files/se_parse_hdr.py -O $PWD/se_parse_hdr.py

Step 4. copy HKD.crt for the respective lpar from local to the same directory
# cp ~/path/to/<hkd_cert.crt> .

So After step 4 completion, the 'static-files' directory will contain below files :- 

HKD.crt
pvextract-hdr
se_parse_hdr.py
```

Once copied, the script can be executed as follows:

```bash
Step 1. Navigate to source directory where script has been copied
#cd Rvps-Extraction

Step 2. Execute the script. The script will install the required packages and will ask for RVPS extraction options. Below will be a sample output.
# ./GetRvps.sh

$$$$$Sample output starts
***Installing necessary packages for RVPS values extraction ***
Updating Subscription Management repositories.
Last metadata expiration check: 0:20:12 ago on Fri Nov 15 07:45:26 2024.
Package python3-3.9.18-3.el9_4.6.s390x is already installed.
Package python3-cryptography-36.0.1-4.el9.s390x is already installed.
Package kmod-28-9.el9.s390x is already installed.
Dependencies resolved.
=========================================================================================
 Package                   Arch   Version          Repository                       Size
=========================================================================================
Upgrading:
 kmod                      s390x  28-10.el9        rhel-9-for-s390x-baseos-rpms    129 k
 python-unversioned-command
                           noarch 3.9.19-8.el9_5.1 rhel-9-for-s390x-appstream-rpms  11 k
 python3                   s390x  3.9.19-8.el9_5.1 rhel-9-for-s390x-baseos-rpms     30 k
 python3-devel             s390x  3.9.19-8.el9_5.1 rhel-9-for-s390x-appstream-rpms 249 k
 python3-libs              s390x  3.9.19-8.el9_5.1 rhel-9-for-s390x-baseos-rpms    7.9 M

Transaction Summary
=========================================================================================
Upgrade  5 Packages

Total download size: 8.3 M
Downloading Packages:
(1/5): python3-3.9.19-8.el9_5.1.s390x.rpm                 52 kB/s |  30 kB     00:00
(2/5): python3-libs-3.9.19-8.el9_5.1.s390x.rpm           8.8 MB/s | 7.9 MB     00:00
(3/5): kmod-28-10.el9.s390x.rpm                          142 kB/s | 129 kB     00:00
(4/5): python-unversioned-command-3.9.19-8.el9_5.1.noarc  32 kB/s |  11 kB     00:00
(5/5): python3-devel-3.9.19-8.el9_5.1.s390x.rpm          903 kB/s | 249 kB     00:00
-----------------------------------------------------------------------------------------
Total                                                    7.0 MB/s | 8.3 MB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                 1/1
  Upgrading        : python3-libs-3.9.19-8.el9_5.1.s390x                            1/10
  Upgrading        : python-unversioned-command-3.9.19-8.el9_5.1.noarch             2/10
  Upgrading        : python3-3.9.19-8.el9_5.1.s390x                                 3/10
  Upgrading        : python3-devel-3.9.19-8.el9_5.1.s390x                           4/10
  Upgrading        : kmod-28-10.el9.s390x                                           5/10
  Cleanup          : python3-devel-3.9.18-3.el9_4.6.s390x                           6/10
  Cleanup          : python3-3.9.18-3.el9_4.6.s390x                                 7/10
  Cleanup          : python-unversioned-command-3.9.18-3.el9_4.6.noarch             8/10
  Cleanup          : python3-libs-3.9.18-3.el9_4.6.s390x                            9/10
  Cleanup          : kmod-28-9.el9.s390x                                           10/10
  Running scriptlet: kmod-28-9.el9.s390x                                           10/10
  Verifying        : kmod-28-10.el9.s390x                                           1/10
  Verifying        : kmod-28-9.el9.s390x                                            2/10
  Verifying        : python3-3.9.19-8.el9_5.1.s390x                                 3/10
  Verifying        : python3-3.9.18-3.el9_4.6.s390x                                 4/10
  Verifying        : python3-libs-3.9.19-8.el9_5.1.s390x                            5/10
  Verifying        : python3-libs-3.9.18-3.el9_4.6.s390x                            6/10
  Verifying        : python-unversioned-command-3.9.19-8.el9_5.1.noarch             7/10
  Verifying        : python-unversioned-command-3.9.18-3.el9_4.6.noarch             8/10
  Verifying        : python3-devel-3.9.19-8.el9_5.1.s390x                           9/10
  Verifying        : python3-devel-3.9.18-3.el9_4.6.s390x                          10/10
Installed products updated.

Upgraded:
  kmod-28-10.el9.s390x                python-unversioned-command-3.9.19-8.el9_5.1.noarch
  python3-3.9.19-8.el9_5.1.s390x      python3-devel-3.9.19-8.el9_5.1.s390x
  python3-libs-3.9.19-8.el9_5.1.s390x

Complete!
***Installation Finished ***
1) Generate the RVPS From Local Image from User pc
2) Generate RVPS from Volume
3) Quit
Please enter your choice: 1
Enter the Qcow2 image with Full path
/root/gaurav-testing-rvps/se-podvm-b7e5e2a-gaurav-s390x.qcow2
mount: /mnt/myvm: special device /dev/nbd3p1 does not exist.
Error: Failed to mount the image. Retrying...
Mounting on second attempt passed
/dev/nbd3 disconnected
SE header found at offset 0x014000
SE header written to '/root/gaurav-testing-rvps/Downloadables/Rvps-Extraction/output-files/hdr.bin' (640 bytes)
se.tag:  a8a938f4ea3f9453da005574e0e75434
se.image_phkh:  92d0aff6eb86719b6b1ea0cb98d2c99ff2ec693df3efff2158f54112f6961508
provenance = ewogICAgInNlLmF0dGVzdGF0aW9uX3Boa2giOiBbCiAgICAgICAgIjkyZDBhZmY2ZWI4NjcxOWI2YjFlYTBjYjk4ZDJjOTlmZjJlYzY5M2RmM2VmZmYyMTU4ZjU0MTEyZjY5NjE1MDgiCiAgICBdLAogICAgInNlLnRhZyI6IFsKICAgICAgICAiYThhOTM4ZjRlYTNmOTQ1M2RhMDA1NTc0ZTBlNzU0MzQiCiAgICBdLAogICAgInNlLmltYWdlX3Boa2giOiBbCiAgICAgICAgIjkyZDBhZmY2ZWI4NjcxOWI2YjFlYTBjYjk4ZDJjOTlmZjJlYzY5M2RmM2VmZmYyMTU4ZjU0MTEyZjY5NjE1MDgiCiAgICBdLAogICAgInNlLnVzZXJfZGF0YSI6IFsKICAgICAgICAiMDAiCiAgICBdLAogICAgInNlLnZlcnNpb24iOiBbCiAgICAgICAgIjI1NiIKICAgIF0KfQo=
-rw-r--r--. 1 root root 640 Nov 15 08:11 /root/gaurav-testing-rvps/Downloadables/Rvps-Extraction/output-files/hdr.bin
-rw-r--r--. 1 root root 446 Nov 15 08:11 /root/gaurav-testing-rvps/Downloadables/Rvps-Extraction/output-files/ibmse-policy.rego
-rw-r--r--. 1 root root 561 Nov 15 08:11 /root/gaurav-testing-rvps/Downloadables/Rvps-Extraction/output-files/se-message
Please enter your choice: 3
[root@a3elp61 Rvps-Extraction]# cat /root/gaurav-testing-rvps/Downloadables/Rvps-Extraction/output-files/ibmse-policy.rego

[root@a3elp61 Rvps-Extraction]# cat /root/gaurav-testing-rvps/Downloadables/Rvps-Extraction/output-files/ibmse-policy.rego
package policy
import rego.v1
default allow = false
converted_version := sprintf("%v", [input["se.version"]])
allow if {
    input["se.attestation_phkh"] == "92d0aff6eb86719b6b1ea0cb98d2c99ff2ec693df3efff2158f54112f6961508"
    input["se.image_phkh"] == "92d0aff6eb86719b6b1ea0cb98d2c99ff2ec693df3efff2158f54112f6961508"
    input["se.tag"] == "a8a938f4ea3f9453da005574e0e75434"
    input["se.user_data"] == "00"
    converted_version == "256"
}
$$$$$Sample output ends

Step 3. User can get the RVPS values inside the file 'ibmse-policy.rego' and use it. There is one additional file also called 'se-message' which has 'provenance' values. It can be used in few cases. But the main file is 'ibmse-policy.rego' here. 
```


## Static Files

Some static files will also be used to generate the RVPS. These include:

- **`pvextract-hdr`**: This is used to extract the SE header from the PODVM SE image (input). It generates an intermediate file, `hdr.bin`, which will be used for further extraction.
- **`se_parse_hdr.py`**: A Python parser used to generate the actual RVPS values.
- **`HKD.crt`**: This certificate will vary between labs. The user needs to copy the same `HKD.crt` used to generate the uploaded PODVM SE image into this path.
