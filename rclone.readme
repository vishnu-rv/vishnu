# README: Using rclone with AWS S3 and E2E Storage

## Prerequisites
- Installed `rclone`
- AWS S3 bucket with proper permissions
- E2E storage configured
- Required credentials (Access Key, Secret Key)

---

## **Step 1: Configure rclone for AWS S3**
1. Run the following command to start the configuration:
   ```bash
   rclone config
   ```
2. Create a new remote:
   - Choose `n` for a new remote.
   - Enter a name (e.g., `AWS`).
   - Select storage type: **Amazon S3**.
   - Set `provider` to `AWS`.
   - Enter your **AWS Access Key** and **Secret Key**.
   - Choose `default` or the desired AWS region.
   - Choose the desired endpoint (leave empty for default AWS S3).
   - Set `acl` to `private` to avoid permission conflicts.
   - Save and exit.

### **Verify AWS Connection**
To list S3 buckets:
```bash
rclone lsd AWS:
```

---

## **Step 2: Configure rclone for E2E Storage**
1. Run `rclone config` again.
2. Create a new remote:
   - Choose `n` for a new remote.
   - Enter a name (e.g., `E2E`).
   - Select storage type: **S3 Compatible Storage**.
   - Enter **E2E Access Key** and **Secret Key**.
   - Set the custom endpoint (e.g., `https://your-e2e-storage-endpoint`).
   - Choose the desired region.
   - Set `acl` to `private`.
   - Save and exit.

### **Verify E2E Connection**
To list buckets in E2E storage:
```bash
rclone lsd E2E:
```

---

## **Step 3: Copy Files to AWS S3**
### **Default Copy (Failed due to ACL Issue)**
```bash
rclone copy /home/tonystark/Downloads/pod.yaml AWS:migratee2e
```
- **Error:** `InvalidBucketAclWithObjectOwnership`
- **Reason:** AWS bucket has `ObjectOwnership: BucketOwnerEnforced`, which **disables ACLs**.

### **Fixed Copy Command**
To fix the issue, use `--s3-acl private`:
```bash
rclone copy /home/tonystark/Downloads/pod.yaml AWS:migratee2e --s3-acl private
```

---

## **Step 4: Copy Files to E2E Storage**
```bash
rclone copy /home/tonystark/Downloads/pod.yaml E2E:migratee2e
```
- No additional ACL configuration is required unless specified by the E2E storage provider.

---

## **Step 5: Automating ACL Setting (Optional)**
To avoid using `--s3-acl private` every time, configure the remote settings:
1. Open `rclone config`.
2. Edit `AWS` remote.
3. Set:
   ```
   s3_acl = private
   ```
4. Save and exit.

---

## **Additional Commands**
### **List Files in a Bucket**
```bash
rclone ls AWS:migratee2e
rclone ls E2E:migratee2e
```

### **Sync Entire Folder**
```bash
rclone sync /path/to/local/dir AWS:migratee2e --s3-acl private
rclone sync /path/to/local/dir E2E:migratee2e
```

### **Delete a File from S3/E2E**
```bash
rclone delete AWS:migratee2e/pod.yaml
rclone delete E2E:migratee2e/pod.yaml
```

---

## **Troubleshooting**
### **1. Permission Issues**
- Ensure your AWS/E2E credentials have write permissions.
- Check if the bucket has `BucketOwnerEnforced` ACL restrictions.

### **2. Slow Transfers**
- Use `--progress` to monitor real-time copying:
  ```bash
  rclone copy /home/tonystark/Downloads/pod.yaml AWS:migratee2e --s3-acl private --progress
  ```
- Increase parallel transfers:
  ```bash
  rclone copy /home/tonystark/Downloads/ AWS:migratee2e --transfers 10 --s3-acl private
  ```

---

## **Conclusion**
You have successfully configured `rclone` for both AWS S3 and E2E storage. By setting the correct ACL (`private`), you can avoid errors related to `BucketOwnerEnforced` settings. For automation, modify the `rclone` configuration to set the default ACL.

For more details, refer to the [rclone documentation](https://rclone.org/).

