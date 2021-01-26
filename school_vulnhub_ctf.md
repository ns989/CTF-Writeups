# School CTF Write Up
"School" is a vulnerable virtual machine from https://vulnhub.com. This write up goes over the security issues found in it and how they can be exploited.

## School CTF
Date: January 26, 2021

CTF Link: https://www.vulnhub.com/entry/school-1,613/

## Technical
 The vulnerable system's IP was 192.168.0.130. First I did a port scan of all 65,535 TCP ports. The system had ports 22, 23, and 80. Port 22 had OpenSSH 7.9p1 running. Port 23 had an unknown, telnet-like service prompting for a verification code. Port 80 had an Apache 2.4.38 web server. Both SSH and the web server revealed the operating system type, with SSH also revealing the exact version (Debian 10). This is an information disclosure vulnerability. I recommended that exact version numbers of software and OS name/version are removed from publicly accessible services. SSH was up to date and there were no publicly available exploits that would give access to the system. The telnet-like service could invite some kind of brute forcing of the verification code, but since I didn't know what format the code could be in I moved on the the web server.
 ![alt text](https://i.imgur.com/MyOJPWR.png "Port Scan")
 I opened my firefox web browser and made navigated to http://192.168.0.130/. The web server redirected to http://192.168.0.130/student_attendance/login.php, a login page. The page's file extension indicates this web server is using PHP. I tried a few credentials to see how the website would respond. "Username or password is incorrect" was returned with a few test password attempts. Next, I tried to find any hidden directories with the directory brute forcing tool "dirb" using the default directory list. The tool discovered a few files and directories including http://192.168.0.130/student_attendance/assets/, http://192.168.0.130/student_attendance/index.php, and http://192.168.0.130/student_attendance/database/. I browsed to the "database" directory, which contained a single file named "student_attendance_db.sql." This looked promising, so I downloaded the file. It contained an entire backup of the system's database. Included were two sets of credentials. One for a staff member, another for the site administrator. The passwords for both were MD5 hashed. I did a dictionary attack on both passwords using hashcat and the rockyou wordlist. The admin account's password was cracked with a password of "admin123". The other account wasn't cracked however. Using the admin credentials I logged in to the website. I had full control over the website's functionality including the ability to change grades, add and remove users, add and remove courses and subjects, and change passwords. 
 
![alt text](https://i.imgur.com/UqR8wgB.png "Admin Panel")

 This compromise was already serious enough, but I was looking to gain access to the underlying operating system. Interestingly, the application didn't handle missing or invalid HTTP GET and POST parameters very well. For example, leaving out the username key and value in a login attempt caused an error like this:
 ```
  Notice: Undefined variable: username in /var/www/html/student_attendance/admin_class.php on line 20
  ```
  This discloses filesystem information and information about the structure of the application itself. However it didn't seem to be anything worse than information disclosure. This behavior was true across the entire application. 
  
  # Not Finished Yet
