# Using FTP to send Files to Landing Zone

**Step 1**: Install HPCC Cluster (Ignore if already installed)<br>
In my case, I have downloaded the VM file to run the HPCC Cluster. The last version of HPCC cluster available is v8.2.0-1. You can download it from here: https://d2wulyp08c6njk.cloudfront.net/releases/CE-Candidate-8.2.0/bin/vm/HPCCSystemsVM-amd64-8.2.0-1.ova
Also download and install Oracle VM Virtual Box.
Open Virtual Box and click on File, and then Import Appliance, then select the path where you have downloaded the HPCCSystemsVM-amd64-8.2.0-1.ova file, then click Next and Finish.
Then select the installed VM and click on settings. Go to Network Tab. In Adapter 1, select Attached to “NAT” and for Adapter 2, select Attached to “Host-only Adapter”. Click OK and double click to start the VM.
<br><br>
**Step 2**: Install FTP Server, an NANO text editor<br>
For installing, run the following codes:<br>
```
sudo apt install vsftpd
sudo apt install nano
```
<br><br>
**Step 3**: Configure vsftpd<br>
Run the following command:
```
sudo nano /etc/vsftpd.conf
```
Now NANO text editor will open, use your arrow key and navigate to very end of file and then write this:
```
write_enable=YES
local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES
user_sub_token=$USER
local_root=/var/lib/HPCCSystems/mydropzone/$USER
```
Next click ctrl+O to write, ctrl+X to exit
<br><br>
**Step 4**: Allow ports from firewall to access FTP server<br>
Run the command:
```
sudo ufw allow from any to any port 20,21 proto tcp
```
<br><br>
**Step 5**: Create a New User to access this FTP Server<br>
Run the command:
```
sudo adduser user
```
Username you can keep anything, I am keeping it as user, also give a password.
<br><br>
**Step 6**: Create a folder in Landing Zone so that the user can access this Folder to upload the files<br>
Run these commands:
```
sudo mkdir /var/lib/HPCCSystems/mydropzone/user
sudo chown nobody:nogroup /var/lib/HPCCSystems/mydropzone/user
sudo chmod a-w /var/lib/HPCCSystems/mydropzone/user
sudo mkdir /var/lib/HPCCSystems/mydropzone/user/upload
sudo chown user:user /var/lib/HPCCSystems/mydropzone/user/upload
echo "My FTP Server" | sudo tee /var/lib/HPCCSystems/mydropzone/user/upload/demo.txt
```
Just check in the ecl watch dropzone if demo.txt is available
<br><br>
**Step 7**: Restart FTP Server<br>
Run these commands:
```
sudo systemctl restart vsftpd
sudo service vsftpd status
```
Check if the FTP Server Status is Active. If it isn't, run these commands:
```
sudo apt purge vsftpd
sudo apt install vsftpd
sudo nano /etc/vsftpd.conf
```
Now nano text editor will open, use your arrow key and navigate to very end of file and then write this
```
write_enable=YES
local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES
user_sub_token=$USER
local_root=/var/lib/HPCCSystems/mydropzone/$USER
```
Next click ctrl+O to write, ctrl+X to exit
<br>
Now run these commands to check the Status again:
```
sudo systemctl restart vsftpd
sudo service vsftpd status
```
Now the status will be Active<br>
Next run:
```
hostname -I
```
Note down the IP Address of the FTP Server.
<br><br>
**Step 8**: Install FileZilla in windows to access FTP Server<br>
Download and Install FileZilla from this link:<br>
https://filezilla-project.org/download.php?type=client
<br>
After installing, open it, enter Host as the IP address, Username as user which you created, Password as the password which you set in installation, then click QuickConnect button, you will see the upload folder, inside it you can see the demo.txt, here you can upload any file from Windows to FTP server, which adds it to landing zone of HPCC cluster. You can go to ecl watch and verify the same.
