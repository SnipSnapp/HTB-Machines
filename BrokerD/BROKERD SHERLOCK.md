# Q1
activemq is the target, we know this from the lab description
# Q2
Going to /opt/apache-activemq-5.15.15 we know the version is 5.15.15
# Q3
Googling active MQ and the version we get the CVE
# Q4
Enumerating the directories we find that the file was dropped under the bin directory. 
Finding what was done, we can do that by looking at the /var/log/audit directory.
# Q5
Answer is combo of args in the audit log. (argc fields)
# Q6
Look at the files that are created by the poc-linux.xml exploit you performed in Broker. under bean id="pb" we can see the java.lang.ProcessBuilder start 
This is what was performed. 
# Q7
This one refers to the exploit the attacker ran. So in order to get the packet, we need to recreate it using the exploit!
# Q8
going to the home directory we can see the bad guy was messing with nginx, trivially we can insinuate from this the attacker abused it.  to verify, run
```sh
su activemq
sudo -l
```
And wow, we see nginx!
# Q9 
Look for audit logs usage of wget inside of /var/log/audit/ and then regrepping for .so.
```sh
grep 'wget' /var/log/audit/* | grep '\.so'
```
# Q10
This one is hard, but we essentially do the same thing as before by grepping the 'tmp' in /var/log/laurel.
```sh
grep 'tmp' /var/log/laurel* | grep '\.conf'
```
# Q11
This one is also ahrd. Downloaded so we need to analyze laurel.
```sh
cat /var/log/laurel/* | grep 'libhax\.so'
```
curl localhost:1337/tmp/libhax.so