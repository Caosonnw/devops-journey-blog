---
title: "CI/CD For Beginners #1 | Install GitLab EE on Ubuntu 22.04: Step-by-Step Guide"
date: 2025-09-07T20:00:00+07:00
series: ["CI-CD"]
categories: ["CI-CD"]
tags: ["ubuntu", "devops", "ci-cd"]
author: "Cao Sơn"
draft: false
showToc: true
tocOpen: false
cover:
  image: "ci-cd/GitLab_logo.png"
  alt: "Install GitLab server"
  caption: "Install GitLab server"
  relative: true
  hiddenInList: true
  hiddenInSingle: false
---

> **Target**: This guide is for beginners and self-learners who want to practice DevOps by installing _GitLab EE on Ubuntu 22.04_. I'll use VMware to run a virtual machine so you don't need to spend money on renting servers or cloud resources — perfect for building your own lab at home.

---

# Introduction ✨

**What is GitLab EE?**

GitLab EE (Enterprise Edition) is the commercial version of GitLab, offering advanced features and professional support for large enterprises and complex organizations. It includes enhanced user management, enterprise-grade security, high scalability, and dedicated technical support to help companies meet compliance and security requirements.

Key features of GitLab EE:

- **Enterprise-focused**: Designed for large organizations with complex needs and high security requirements
- **Professional support**: Direct support from GitLab Inc. for quick resolution of installation and usage issues

---

# Preparation 💻

First, we need to prepare 2 Ubuntu 22.04 servers. You can use VMware Fusion (macOS) or VMware Workstation (Windows).

{{< figure src="/ci-cd/1.png" alt="Two Ubuntu servers setup" caption="Images 1: Two Ubuntu servers setup">}}

## Server Requirements

| Hostname      | OS           |     IP Address | RAM (Minimum) | CPU (Minimum) |
| ------------- | ------------ | -------------: | :-----------: | :-----------: |
| ubuntu-server | Ubuntu 22.04 | 172.16.177.110 |      4GB      |    2 cores    |
| gitlab-server | Ubuntu 22.04 | 172.16.177.100 |      4GB      |    2 cores    |

### Server Purposes

- **ubuntu-server**: This will be our main server for continuing the CI/CD project deployment series
- **gitlab-server**: This will be our dedicated GitLab server

> **Note**: Make sure both servers have stable network connectivity and can communicate with each other.

#### Network Configuration

- **Desktop PC**: Use **Bridge** network mode for better performance and direct network access
- **Laptop/MacBook**: Use **NAT** network mode for stable connectivity, especially when switching between different networks (home, office, coffee shop, etc.)

> **Tip**: NAT mode is more reliable for mobile devices as it doesn't depend on the host's network configuration changes.

# Getting Started 🚀

## Step 1: Download GitLab EE Package 📦

First, open your web browser and search for "gitlab ee package". Then navigate to the official GitLab package repository as shown in the image below:

{{< figure src="/ci-cd/2.png" alt="GitLab EE package repository" caption="Image 2: GitLab EE package repository">}}

Here you'll see many different versions available for various operating systems. While many enterprises still use older GitLab versions, we'll use the latest version (18.1.5) to familiarize ourselves with GitLab's newest interface. The latest version doesn't differ significantly from older versions in terms of core functionality.

{{< figure src="/ci-cd/3.png" alt="GitLab version selection" caption="Image 3: Selecting GitLab version and operating system">}}

After selecting the correct version and operating system, click on the script section to copy the installation commands as highlighted in the image:

{{< figure src="/ci-cd/4.png" alt="Copy installation script" caption="Image 4: Copying the GitLab installation script">}}

## Step 2: Install GitLab EE 🔧

Now we'll SSH into the GitLab server and switch to the root user. Paste the script we just copied and run it. The GitLab EE packages will then be downloaded and installed.

```bash
# SSH into the GitLab server
ssh [user]@[IP your server Gitlab]

# Switch to root user
sudo -i
# Enter password

# Run the installation script (paste the copied script here)
# The script will install dependencies and GitLab EE package
```

> **Note**: The installation process may take several minutes depending on your internet connection speed and server performance.

After copying the script, proceed with the installation command:

```bash
# Install GitLab EE version 18.1.5
sudo apt-get install gitlab-ee=18.1.5-ee.0
```

The installation process will begin downloading and configuring GitLab EE on your server. This may take several minutes to complete.

{{< figure src="/ci-cd/6.png" alt="GitLab EE installation in progress" caption="Image 5: GitLab EE installation process">}}

> **Important**: During installation, GitLab will automatically configure itself. Make sure you have sufficient disk space and memory available before proceeding.

## Step 3: Configure Domain Access 🌐

Once the installation is completed successfully, we'll proceed to set up a domain. Since I don't have a real domain and want to set up a virtual domain on the machine so I can access GitLab without using the IP address (which would be inconvenient), we'll modify the hosts file.

We will edit the hosts file to map our GitLab server's IP to a custom domain name.

{{< figure src="/ci-cd/10.png" alt="Editing hosts file" caption="Image 6: Accessing the hosts file for domain configuration">}}

Add the GitLab server IP address and the domain name we want to use:

{{< figure src="/ci-cd/11.png" alt="Adding domain mapping" caption="Image 7: Adding IP and domain mapping to hosts file">}}

> **Note**: This local domain configuration allows you to access GitLab using a friendly domain name instead of remembering the IP address. The domain will only work on the machine where you modify the hosts file.

Similarly, on your local machine, we need to make the same modification:

{{< figure src="/ci-cd/7.png" alt="Local hosts file location" caption="Image 8: Locating the hosts file on your local machine">}}

{{< figure src="/ci-cd/8.png" alt="Adding domain mapping on local machine" caption="Image 9: Adding the same IP and domain mapping to your local hosts file">}}

For Mac users, after making the changes, run the following commands to flush the DNS cache:

```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

{{< figure src="/ci-cd/9.png" alt="Flushing DNS cache on Mac" caption="Image 10: Flushing DNS cache on macOS">}}

> **Note**: The DNS cache flush ensures that your system immediately recognizes the new domain mapping without requiring a restart.

## Step 4: Configure External URL 🔗

Next, we need to configure the external URL for GitLab. We'll edit the GitLab configuration file:

```bash
vi /etc/gitlab/gitlab.rb
```

{{< figure src="/ci-cd/12.png" alt="Editing GitLab configuration file" caption="Image 11: Opening the GitLab configuration file">}}

Once you open the file, you'll see the `external_url` line at the top. Change this to the domain you just set up. In my case, I'll change it to `gitlab.caosonnw.tech`:

{{< figure src="/ci-cd/13.png" alt="Setting external URL" caption="Image 12: Configuring the external URL with your custom domain">}}

> **Important**: Make sure to uncomment the `external_url` line by removing the `#` symbol and replace the default URL with your custom domain name.

After making the changes, run the command:

```bash
sudo gitlab-ctl reconfigure
```

This command applies the configuration from the `/etc/gitlab/gitlab.rb` file, initializes or updates GitLab services, and ensures system consistency.

If you modify the configuration in `gitlab.rb` without running this command, the changes will not take effect.

Every time you make changes (such as GitLab domain, HTTP/HTTPS ports, SMTP email configuration), you must run this command again to apply the changes.

## Step 5: Access GitLab Web Interface 🌐

After GitLab has been successfully reconfigured, you can access the web interface by navigating to the domain you set up in your web browser. You should see the GitLab sign-in page as shown below:

{{< figure src="/ci-cd/16.png" alt="GitLab sign-in page" caption="Image 13: GitLab sign-in page after successful installation">}}

> **Success!** If you can see this sign-in page, it means GitLab EE has been successfully installed and configured on your Ubuntu 22.04 server.

### Default Administrator Access

To log in for the first time, you'll need the default administrator credentials:

- **Username**: `root`
- **Password**: Located in `/etc/gitlab/initial_root_password`

To retrieve the initial password, run:

```bash
sudo cat /etc/gitlab/initial_root_password | grep Password
```

You will see the password output as shown in the image below:

{{< figure src="/ci-cd/17.png" alt="GitLab initial root password" caption="Image 14: Retrieving the initial root password">}}

> **Security Note**: Make sure to change the default password immediately after your first login for security purposes.

Now paste the password you just retrieved into the sign-in form and log in:

{{< figure src="/ci-cd/18.png" alt="GitLab login form" caption="Image 15: Entering credentials to log into GitLab">}}

After successfully logging in, you'll see a notification prompting you to set up some important settings. Click on it to configure these settings.

First, uncheck the two boxes: "Sign-up enabled" and "Require admin approval for new sign-ups" as shown below:

{{< figure src="/ci-cd/19.png" alt="GitLab sign-up settings" caption="Image 16: Disabling sign-up settings for security">}}

Next, navigate to CI/CD settings and uncheck the box: "Default to Auto DevOps pipeline for all projects" as shown below:

{{< figure src="/ci-cd/20.png" alt="GitLab CI/CD settings" caption="Image 17: Disabling Auto DevOps pipeline default setting">}}

## Step 6: Change Root Password 🔐

After configuring the basic settings, you should change the default root password for security reasons.

Click on your avatar in the top-right corner and select "Edit profile":

{{< figure src="/ci-cd/21.png" alt="Edit profile menu" caption="Image 18: Accessing the edit profile option">}}

Then click on "Password" in the left sidebar:

{{< figure src="/ci-cd/22.png" alt="Password settings" caption="Image 19: Navigating to password settings">}}

Enter your current password and set a new, secure password:

{{< figure src="/ci-cd/23.png" alt="Change password form" caption="Image 20: Changing the root user password">}}

The system will automatically sign you out after the password change. Sign in again with your new password:

{{< figure src="/ci-cd/24.png" alt="Sign in with new password" caption="Image 21: Signing in with the updated password">}}

## Step 7: Create New User Account 👤

Now let's create a new user account for better security practices. It's recommended to use a dedicated user account instead of the root account for daily operations.

Navigate to the Admin area by clicking on the "Admin" button in the top navigation bar, then click on "New user":

{{< figure src="/ci-cd/25.png" alt="Admin area new user" caption="Image 22: Creating a new user in Admin area">}}

Fill in the basic information for the new user including name, username, and email address:

{{< figure src="/ci-cd/31.png" alt="New user creation form" caption="Image 23: Filling in new user details">}}

You can optionally check "Administrator" to grant admin privileges to this user. After filling in all the required information, click "Create user" to proceed.

Once the user is created successfully, you'll see the user in the users list. Click "Edit" next to the newly created user to set a password:

{{< figure src="/ci-cd/26.png" alt="User list with edit option" caption="Image 24: Editing the newly created user to set password">}}

In the user edit page, navigate to the password section and set a secure password for the new user:

{{< figure src="/ci-cd/27.png" alt="Set user password" caption="Image 25: Setting password for the new user">}}

After saving the password, sign out from the root account to test the new user login:

{{< figure src="/ci-cd/28.png" alt="Sign out option" caption="Image 26: Signing out to test new user account">}}

When you sign in with the new user for the first time, the system will require a password update for security reasons:

{{< figure src="/ci-cd/29.png" alt="Password update required" caption="Image 27: System requiring password update for new user">}}

Set a new password that meets the security requirements and confirm the change.

After successfully updating the password, sign in again with the new user credentials:

{{< figure src="/ci-cd/30.png" alt="New user login" caption="Image 28: Successfully logging in with the new user account">}}

> **Best Practice**: Using a dedicated user account instead of the root account provides better security and audit trails for your GitLab instance.

# Conclusion 🎉

Congratulations! You have successfully installed and configured GitLab EE on Ubuntu 22.04. Let's recap what we've accomplished:

## What We've Achieved ✅

- ✅ **Installed GitLab EE 18.1.5** on Ubuntu 22.04 server
- ✅ **Configured custom domain** for easy access instead of IP addresses
- ✅ **Set up external URL** and applied configuration changes
- ✅ **Secured the installation** by changing default passwords
- ✅ **Created a dedicated user account** following security best practices
- ✅ **Configured basic settings** for optimal security

## Next Steps 🚀

Now that your GitLab server is up and running, you can:

1. **Create your first project** and start version controlling your code
2. **Set up GitLab Runners** for CI/CD automation
3. **Configure SSH keys** for secure repository access
4. **Explore GitLab's DevOps features** like issue tracking, merge requests, and CI/CD pipelines
5. **Integrate with external tools** and services for your DevOps workflow

## Important Security Reminders 🔒

- **Regularly update GitLab** to get security patches and new features
- **Use strong passwords** and consider enabling two-factor authentication
- **Regular backups** of your GitLab data and configuration
- **Monitor system resources** to ensure optimal performance

Your GitLab EE server is now ready to support your DevOps journey and CI/CD pipeline projects. Happy coding! 🎯
