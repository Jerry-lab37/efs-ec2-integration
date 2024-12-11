### **1. Prerequisites**

- **EFS Setup:** Ensure you have created an EFS filesystem in the AWS Management Console.
- **Security Group Rules:** The security group associated with your EC2 instance and EFS must allow NFS traffic. Add an inbound rule to allow traffic on port 2049 (NFS) between your EC2 instance and EFS. 
- **IAM Role:** Your EC2 instance should have an IAM role with necessary permissions for EFS (e.g., `AmazonElasticFileSystemFullAccess`). 

---

### **2. Mount the EFS on EC2**

#### a. Install NFS Utilities

On your EC2 instance, install the necessary NFS client tools:

`sudo apt update && sudo apt install -y nfs-common`

#### b. Mount the EFS

Use the EFS DNS name to mount the file system. Replace `<file-system-id>` and `<mount-point>` with your EFS ID and the desired mount directory:

```
sudo mkdir -p /mnt/efs 
sudo mount -t nfs4 <file-system-id>.efs.<region>.amazonaws.com:/ /mnt/efs
```

#### c. Verify the Mount

Run the following command to check if the EFS is mounted:

`df -h`

You should see an entry for your EFS in the output.

---

### **3. Test EFS**

#### a. Create and Verify Files

- Create a test file in the EFS mount point:

    `sudo touch /mnt/efs/test-file`

- Verify the file exists:

    `ls -l /mnt/efs`


#### b. Test Read/Write

- Write some content to the file:

    `echo "EFS is working!" | sudo tee /mnt/efs/test-file`

- Read the content:

    `cat /mnt/efs/test-file`


---

### **4. Configure Moodle to Use EFS** 

To use EFS for Moodle, you can move directories like `moodledata` to the EFS mount. For example:

1. Stop your web server to prevent conflicts:

    `sudo systemctl stop apache2`
    
2. Move `moodledata` to the EFS mount:

    `sudo mv /var/moodledata /mnt/efs/` ##replace with your moodledata path

3. Create a symbolic link:

    `sudo ln -s /mnt/efs/moodledata /var/moodledata`  ##replace with your moodledata path

4. Restart the web server:

    `sudo systemctl start apache2`

 5.Adjust permissions to ensure the web server can access the directory:
 
      sudo chown -R www-data:www-data /mnt/efs/moodledata
