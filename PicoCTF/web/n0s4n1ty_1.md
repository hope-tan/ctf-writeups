# 🎯 PicoCTF Writeup: n0s4n1ty 1

**Category:** Web Exploitation  
**Author:** hope-tan  
**Date Completed:** July 9, 2025  
**Link:** [PicoCTF n0s4n1ty 1](https://play.picoctf.org/practice/challenge/482?category=1&page=1)


## 🧠 Challenge Description
"A developer has added profile picture upload functionality to a website. However, the implementation is flawed, and it presents an opportunity for you. Your mission, should you choose to accept it, is to navigate to the provided web page and locate the file upload area. Your ultimate goal is to find the hidden flag located in the /root directory."

## 📁 Resources Provided

- Sample Web Application
- Hint 1: "File upload was not sanitized"
- Hint 2: "Whenever you get a shell on a remote machine, check sudo -l"


## 🔍 Solution Steps

### 1. Initial Observations   
I opened this CTF on my personal Kali VM. The provided web app was a simple HTML file with a profile photo upload form.
<img width="1919" height="734" alt="image" src="https://github.com/user-attachments/assets/787e9656-dc87-4151-807e-4f5a8b717f98" />

My first step was to look at the source code:
<img width="1547" height="731" alt="image" src="https://github.com/user-attachments/assets/55a31e41-6853-497a-9622-5ad5e00d894d" />

I noticed 2 things. First, the file upload was handled by a PHP file, called upload.php. Second, there didn't seem to be file input sanitization on the front end.  
***
### 2. Testing Upload Functionality  
To exploit a web application, it's important to understand how the app was intended to work. To test the upload functionality normally, I tried uploading a sample file:
<img width="1919" height="715" alt="image" src="https://github.com/user-attachments/assets/bc1d4002-ad45-428b-9759-bff3452a8a8e" />

Interesting... upon form submission, the website tells me exactly where the file was uploaded to:
<img width="1919" height="249" alt="image" src="https://github.com/user-attachments/assets/20da4328-1a67-40b1-9640-6864c38390c4" />

Let's see if this file is stored somewhere publicly accessible by visiting adding /uploads/pikachu.jpg to the URL. 

Yikes. There it is, and on the public internet no less.
<img width="2472" height="1073" alt="image" src="https://github.com/user-attachments/assets/5ea10ebd-3db9-4ecd-8f68-fb50710e416b" />
***
### 3. PHP Script
Since I didn't see any file input sanitization, I decided to create a PHP script to upload and pop a web shell. I left it empty, with "cmd" as the command placeholder for now.
<img width="1191" height="138" alt="image" src="https://github.com/user-attachments/assets/8c75a407-556d-4667-9dc6-15e1d75e82d7" />

Time to see if the website lets me upload this script.
<img width="1919" height="700" alt="image" src="https://github.com/user-attachments/assets/5439f353-23a6-4511-bb1e-87a71729d74e" />

Cool, the script uploaded successfully.
<img width="1919" height="180" alt="image" src="https://github.com/user-attachments/assets/fad8e90b-0f45-40c4-9f6b-1164da102469" />

I figured I could access my PHP script by adding /uploads/shell.php to the URL.

When I visited the URL, I came across this error message. This tells me that the script was trying to execute commands. My original script was empty, so the error makes sense.  This is very bad news for whoever owns this site but very good news for me.
<img width="1919" height="313" alt="image" src="https://github.com/user-attachments/assets/4ce1f0c6-67a1-46dc-acee-0b65af8362c1" />

***
### 4. Web Shell Exploit!
Time for the fun part! I'm going to replace the "cmd" placeholder from my original script by adding "?cmd=id" to the end of the URL. If this works and returns my ID, then I'm definitely executing commands against the actual web server.
<img width="1919" height="182" alt="image" src="https://github.com/user-attachments/assets/9f7f09b9-ca2c-439b-aeff-d92d3f076eb1" />

Nice, I'm the www-data user! Now let's try listing the contents of the root directory with "?cmd=ls /root".
<img width="1919" height="678" alt="image" src="https://github.com/user-attachments/assets/db858ad6-af41-4fe5-889c-6df7073e7d90" />

I'm blocked from seeing anything, likely because I'm not the root user. 

Let's take a look at my permissions with "?cmd=sudo -l". Now this is interesting... www-data user can run any commands on challenge (ALL)NOPASSWD: ALL, meaning I can run any command using sudo *without entering a password*. 
<img width="1919" height="362" alt="image" src="https://github.com/user-attachments/assets/926c3223-a3f3-4268-b6e9-6d681c16f518" />

Let's try looking at the root directory again, but this time with "?cmd=sudo ls /root". Usually, adding sudo to a command triggers a password prompt to ensure that whoever is using sudo has been authorized to do so, but that didn't happen because of my NOPASSWD privilege.
<img width="1919" height="259" alt="image" src="https://github.com/user-attachments/assets/4473cdd8-ab53-4998-a50f-b6e83b2985ff" />

We can see the root directory now, and there's our flag! One last "?cmd=sudo cat /root/flag.txt" to read the flag:
<img width="1919" height="200" alt="image" src="https://github.com/user-attachments/assets/21867a12-f039-4924-9a1e-95f2e373123e" />

Ta-da! There's our flag. 
***
## 💡Lessons Learned
This CTF demonstrates the importance of file upload sanitization. Otherwise, a simple malicious script upload can grant attackers full control over the hosting web server. Companies must take a defense-in-depth approach to guard against web app vulnerabilities like this one.
