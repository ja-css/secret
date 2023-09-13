Change a user password directly in `/etc/shadow` file  
  
Changing a password without using the `passwd` command directly would involve manually modifying the `/etc/shadow` file, which is not recommended and can be risky if not done correctly. However, if you have a specific use case that requires this approach, here are the general steps to manually change a password:  
  
### 1.   **Backup `/etc/shadow`:**   
Before making any changes, it's crucial to create a backup of the `/etc/shadow` file. This way, you can restore it in case anything goes wrong:  
      
`sudo cp /etc/shadow /etc/shadow.backup`   
      
### 2.  **Generate a New Password Hash:**  
You will need to generate a new password hash for the desired password. You can use the `openssl passwd` command to create the hash. For example, to create a hash for the password "newpassword," you can do:  
      
`openssl passwd -6 -salt <random_salt> <newpassword>`   
  
Replace `random_salt` with a random salt value (you can use a tool like `openssl rand -hex 8` to generate a random salt).  
  
### 3.  **Edit `/etc/shadow`:**   
Now, you'll need to edit the `/etc/shadow` file to replace the old password hash with the new one. Open the file with a text editor using sudo:  
  
`sudo nano /etc/shadow`   
  
Find the line corresponding to the user whose password you want to change, and replace the old password hash (the second field in the line) with the new one you generated in the previous step.  
  
For example, the line might look like this before the change:  
  
`username:$6$old_salt$old_hash:...`   
  
After the change, it should look like this:  
  
`username:$6$random_salt$new_hash:...`   
  
### 5.  **Save the File and Exit:**   
Save the changes in the `/etc/shadow` file and exit the text editor.  
  
### 6.  **Test the New Password:**   
Try logging in with the new password to make sure it works as expected.  
  
Please exercise extreme caution when manually editing system files like `/etc/shadow`, as any mistakes can render your system inaccessible. It's always recommended to use the `passwd` command or other standard user management tools for changing passwords, as they handle the necessary encryption and security checks automatically.  
  
Manually editing password hashes in the `/etc/shadow` file should be a last resort and should only be done by experienced system administrators who are aware of the risks involved.