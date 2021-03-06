---
layout: post
title: Kupe - getting access
permalink: /lessons/kupe-access/
chapter: kupe
---

You will learn how to set up your account on Kupe and how to login to the machine.

### Requirements

You will need a terminal program to login to Kupe:

- Windows: MobaXterm, Windows 10 bash, or Putty
- MacOS X: Terminal app, iTerm2
- Linux: Terminal app, xterm

In addition you will need:

 1. an [account](#account) on kupe
 2. [two factor authentication](#twofactor) set up if connecting from outside NIWA's network

### Connecting to kupe

#### Users outside of NIWA

Connecting to kupe is a two step process. First, connect to kupe's lander node
```
ssh -Y <myusername>@lander.nesi.org.nz
```
inside your terminal program, where ```<myusername>``` is your account user name. You will see the following prompt
```
First Factor:
```
Enter your password, followed by
```
Second Factor (optional):
```
Enter the 6-digit code from the mobile device. To execute and compile code, you will as a second step connect to the login node
```
ssh -Y login
```
using your password followed immediately by the 2nd factor token. E.g. mypassword345678 where 345678 is your 6-digit code from the mobile.

#### Users inside of NIWA's network

You can connect directly to kupe's login node
```
ssh -Y <myusername>@login.kupe.niwa.co.nz
```
using your password. **Note: if you have two-factor authentication set up then you need to provide mypassword345678 where 345678 is your 6-digit code from the mobile at the password prompt.**


### <a name="account"></a> Setting up an account on Kupe

If you are logging in for the first time to Kupe, you will need to set up your account. First, you will need to login to NeSI user portal. This populates NeSI database with your basic account information which will be used to set up your account.

1. Access [My NeSI  Portal](https://my.nesi.org.nz) via your browser.
 ![logging-in](../../assets/img/portal_login.png)
2. Log in using your institutional credentials via Tuakiri. See example below for logging in with NIWA credentals.
![logging-in](../../assets/img/tuakiri_credentials.png)

![logging-in](../../assets/img/niwa_turakiri.png)

3. After successful login, you should see a screen similar to the one below
![logging-in](../../assets/img/login_success.png)

4. Please click on ‘Reset Password’ button to proceed. It will send you an e-mail with temporary URL.


**Note** If you don’t see ‘Reset Password’ button and instead see error messages, it means your information on our database did not match your Tuakiri identity. Please see [troubleshooting](#trouble) section.

5. Clicking on the link on your e-mail will open up the following page that shows your temp password.
![logging-in](../../assets/img/temp_password.png)

6. During your first login with the temporary password you will be asked to change it.
![logging-in](../../assets/img/password_change.png)

Connecting to the HPC requires two-factor authentication at all times, your password, and an additional factor. These additional factors can be:
- A keycode provided by an external generator (e.g., via smartphone app)
- Connecting from NIWA's physical network (at a NIWA branch)
- Connecting through a NIWA VPN session

### <a name="twofactor"></a> Setting up two factor authentication

Note: You can skip this section if you log on from inside the NIWA network or via NIWA's VPN.

Please make sure you have a mobile device with a working camera and then install Google Authenticator app (free). The next step can only be done once.

Go back to My NeSI portal and click on Accounts or refresh the page and you will see a new option to ‘Link your mobile device’
![logging-in](../../assets/img/link_device.png)


Clicking on `Link your mobile device' will prepare your 2nd factor login so that you can login to our lander node from outside of the NIWA network. After clicking on ‘Link your mobile device’ you will be instructed to prepare your mobile device before proceeding.
![logging-in](../../assets/img/prepare_device.png)


Click ‘Continue’ and scan your QR code
![logging-in](../../assets/img/qr_code.png)


Open your Google Authenticator app and click on the add button and select ‘Scan a barcode’. Point your camera at the QR code displayed on the screen and it will be added to your phone.

Now logging in to the lander node will prompt you for ‘First factor’ where you enter your newly set password, and ‘Second factor’ which is the 6 digit code displayed on your Google Authenticator app. The 6 digit code rotates every 30 seconds, and it can only be used once. This means that you can only login to the lander node once every 30 seconds. Also the prompt says (optional), but it is not optional, and we are working to fix the message.

### Setting up access for connecting from outside of NIWA computer network (advanced)

On most Linux and MacOS machines the login process can be simplified to just a single SSH command, jumping across the lander node on the way to kupe. With the following lines in your `~/.ssh/config` file you can run the command `ssh kupe` on your machine and it will take you straight to kupe. Since we are using SSH ProxyCommand to jump first to the lander node, you will need to enter your ‘First factor’ (password) and then your ‘Second factor’ (from Google Authenticator) on the first jump and then a combination of your ‘First factor’+‘Second factor’ on the second jump when prompted for a password. 
```
Host kupe
   User your_username
   Hostname login.kupe.niwa.co.nz
   ProxyCommand ssh -W %h:%p lander
   ForwardX11 yes
   ForwardX11Trusted yes
   ServerAliveInterval 300
   ServerAliveCountMax 2

Host lander
   User your_username
   HostName lander.nesi.org.nz
   ForwardX11 yes
   ForwardX11Trusted yes
   ServerAliveInterval 300
   ServerAliveCountMax 2
```
The `ForwardX11` directives will enable X11 forwarding and are optional. This can be combined with the `Control` directives to make additional SSH logins and transferring data easier (see the [data transfer](009-data_transfer.md) page).

### <a name="trouble"></a> Troubleshooting

Please contact support@nesi.org.nz if you have any problems or questions. Also, let us know which of the following screen you see as this will enable us to address your issue more quickly. Thank you.

If your account is not ready, you may see:
![logging-in](../../assets/img/no_account.png)

If your project is not set up, you may see:
![logging-in](../../assets/img/no_project.png)

