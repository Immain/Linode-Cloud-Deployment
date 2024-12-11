<p align="center">
  <a href="" rel="noopener">
 <img width=200px height=200px src="https://assets.linode.com/akamai-logo.svg" alt="Project logo"></a>
</p>

<h3 align="center">Akamai Linode Deployment</h3>



<p align="center">
Linode, now part of Akamai, provides powerful and reliable cloud hosting solutions. With a range of virtual private servers, Linode offers scalable compute platforms with additional storage, security, and monitoring features. 
</p>

## Getting Started

For this project, we will be deploying a Linode through Akamai using Ansible. The prerequisites for this project are as follows:

- Linode Account (Sign up at [Akamai](https://login.linode.com/signup))
- Linode [Pricing](https://www.linode.com/pricing/)
- Basic understanding of cloud computing concepts
- Familiarity with the Command Line Interface
- Ansible for Linux, macOS, or WSL



## Generate SSH Key and Personal Access Token:

**Step One:** Create your SSH Key
1. In your Linode account, click your name in the top-right corner and select ```SSH Keys```

2. In the top-right corner, click on ```Add SSH Key```

3. On your local or test machine, generate a new key using
   ```
   ssh-keygen
   ```

4. In your Linode Account, create a label for your SSH Key and then copy the  ```id_rsa.pub``` output using
   ```
   cat ~/.ssh/id_rsa.pub
   ```
5. Add the Public Key to your Linode account and click save. 

**Step Two:** Generate your Access Token
1.  Still in your Linode account, click your name in the top-right corner and select ```API Tokens```

2. Click ```Create a personal access token```, give your token a label and select your required attributes

##  Install Ansible:

> [!IMPORTANT]  
> Ansible is only available on macOS and Linux. If you are using Windows, you can use the Windows Subsystem for Linux (WSL) to install Ansible.

**Step One:**  Install Ansible on your local machine.

```
sudo apt install ansible -y
```

Verify the Ansible Version:
```
ansible --version
```

**Step Two:** Install the Akamai Linode module from Ansible Community, you can find the Ansible Linode documentation [here](https://galaxy.ansible.com/ui/repo/published/linode/cloud/docs/)

```
ansible-galaxy collection install linode.cloud
```

```
sudo pip3 install -r .ansible/collections/ansible_collections/linode/cloud/requirements.txt
```
## Ansible Vault
Ansible vault provides a way to encrypt and manage sensitive data such as passwords and tokens

### Vault Setup:

Create a new Ansible Vault password file:
```
sudo ansible-vault create environment/vault.yml
```

You will be prompted to enter and confirm a password:
```
New Vault password: 
Confirm New Vault password:
```

Add your ```oauth_token``` to the vault using a ```vault_``` prefix
```
vault_oauth_token: superdupersecrettoken
```
Ansible will encrypt the contents when you close the file. If you check the file, instead of seeing the words you typed, you will see an encrypted block:
```
sudo cat environment/vault.yml
```
<b>Output:</b>
```
$ANSIBLE_VAULT;1.1;AES256
343333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333
```

In your ```/group_vars/vars.yml``` file, reference the encrypted variables:
```
oauth_token: "{{ vault_oauth_token }}"
```

Reference the vault under ```vars_files``` in your playbook, include both the vars.yml and vault.yml files:
```
- hosts: localhost
  vars_files:
    - ./group_vars/vars.yml
    - ./group_vars/vault.yml
```
## Generating a Hashed Password:
Creating a hashed root password is required for Linode and provides better security rather than having your passwords in plain text

1. Make sure Python is installed

   ```
   sudo apt install python3
   ```
2. Crypt provides access to the Unix crypt() function, which is used to hash passwords.
  ```
  python3 -c 'import crypt,getpass;pw=getpass.getpass();print(crypt.crypt (pw) if (pw==getpass.getpass("Confirm: ")) else exit())'
  ```
**It will immediately ask you to type in a password and then confirm it (Like The Example Below)**
  ```
  Password: 
  Confirm: 
  $6$TRaOi6s1u.dPLtny$isRp.ZmY9XzF5EyFCw3Dq0HMVd4D/   uz0GnA1BLLY6Z64emsgKCtbfx51aWyUGExTpzTNka8lP4cFZx93MURKP0
  ```
3. Once this is completed, take the generated hash and add it to your ansible vault with 

   ```
   ansible-vault edit vault.yml
   ```

## Run the playbook
To run your playbook use
```
sudo ansible-playbook deploy.yml --ask-vault-pass
```

