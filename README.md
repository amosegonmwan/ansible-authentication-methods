# Ansible Authentication Methods

This guide covers three primary authentication methods used in Ansible:

1. **Ansible PasswordLess Authentication**
2. **Ansible Password Authentication**
3. **Ansible Vault**

---

## 1. Ansible PasswordLess Authentication

Ansible PasswordLess Authentication allows you to securely manage remote nodes without the need to enter a password for every operation.

### Step 1: Generate SSH Key Pair

On the Ansible control node, generate an SSH key pair if you don't already have one:

- **Enhanced Security (4096-bit RSA keypair):**

  ```bash
  ssh-keygen -t rsa -b 4096
  ```
- **Default SSH Keypair:**

  ```bash
  ssh-keygen -t rsa -b 4096
  ```

  ### Step 2: Copy SSH Public Key to Managed Nodes
Copy your public SSH key to each managed node using the `ssh-copy-id` command:

  ```bash
  ssh-copy-id ansible@10.0.1.213
  ssh-copy-id ansible@10.0.1.50
  ssh-copy-id ansible@10.0.1.67
  ```
Note: Replace the IP addresses with your actual managed node IPs.

### Step 3: Update Ansible Inventory File
Update the inventory file to list your managed nodes:
  ```bash
  [db]
  10.0.1.213  
  10.0.1.50  
  
  [web]
  10.0.1.67 
  ```
Note: Create groups of your choice.

### Step 4: Test SSH Connection
Test the SSH connection by logging into a remote node:
  ```bash
  ssh ansible@10.0.1.213
  ```
Tip: If SSH login is successful without prompting for a password, your setup is correct.


## 2. Ansible Password Authentication
Password-based authentication can be used as an alternative to SSH keys, particularly useful in environments where SSH keys have not been set up.

### Option 1: Specify Password in the Inventory File
Specify the username and password directly in the inventory file:
  ```ini
  [db]
  10.0.1.213 ansible_user=ansible ansible_pass=password
  10.0.1.50 ansible_user=ansible ansible_pass=password
  
  [web]
  10.0.1.67 ansible_user=mark ansible_pass=password
  ```

### Option 2: Use a Variable File
Store passwords in a separate variables file (`vars.yaml`) to avoid hardcoding them in the inventory file:
  ```yaml
  # vars.yaml
  users:
    ansible:
      ansible_ssh_pass: admin123
    mark:
      ansible_ssh_pass: mark123
  ```

Modify the inventory file to reference this variable file:
  ```ini
  [db]
  10.0.1.213 ansible_user=ansible 
  10.0.1.50 ansible_user=ansible 
  
  [web]
  10.0.1.67 ansible_user=mark
  ```
Example Playbook:
  ```yaml
  - hosts: all
    vars_files:
      - vars.yml
    tasks:
      - name: Install a package
        yum:
          name: httpd
          state: latest
  ```

### Option 3: Prompt for Password at Runtime
You can prompt for the password during playbook execution using the --ask-pass option:
  ```bash
  ansible-playbook playbook.yaml --ask-pass
  ```

### Option 4: Use Environment Variables
Set the password as an environment variable:
  ```bash
  export ANSIBLE_PASSWORD=ansible@123 
  ansible-playbook playbook.yaml --extra-vars "ansible_ssh_pass=$ANSIBLE_PASSWORD"
  ```
Note: Using environment variables can help keep your playbooks clean and reduce the risk of exposing sensitive information.

---

## 3. Ansible Vault
Ansible Vault is a powerful tool for securing sensitive data, such as passwords, API keys, and other confidential information in your Ansible playbooks.

### Common Ansible Vault Commands

### 1. Create a New Encrypted File

Create a new encrypted file to store sensitive information:

```bash
ansible-vault create credentials.yaml
```
### 2. Encrypt an Existing File
Encrypt an existing file:

```bash
ansible-vault encrypt credentials.yaml
```
This will prompt for a password and encrypt the file without opening an editor.

### 3. Decrypt an Encrypted File
To decrypt an encrypted file:

```bash
ansible-vault decrypt credentials.yaml
```
This will prompt for the password and decrypt the file.

### 4. View the Contents of an Encrypted File
To view the contents of an encrypted file without decrypting it:

``` bash
ansible-vault view credentials.yaml
```
This will prompt for the password and display the content.

### 5. Edit an Encrypted File
To edit an encrypted file:

``` bash
ansible-vault edit credentials.yaml
```
This will prompt for the password and then open the file in an editor for you to make changes.
