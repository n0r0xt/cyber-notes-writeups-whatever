# Spaghetti writeup
## this is my writeup for the spaghetti malware challenge in huntress CTF 2025

The first thing I did was try to identify each of the files AYGIW.tmp and Spaghetti. They were both ASCII files so I opened them in notepad and then vscode.  
I couldn't immediately identify what AYGIW.tmp was used for, but I identified Spaghetti as a powershell script which was obfuscated by filling it with comments and random host writes and arithmetic operations.  
To clear the file of comments, I saved it as spaghetti.txt and used the following python script:  
```python
put the script here
```
From the cleanspaghetti.txt file, I just opened it in vscode and copied any important looking parts over to a new file which I called a.ps1:
```powershell
a.ps1
```
I found the first flag by watching the exection of the script and the output of the resultant binary that was decoded after the set-mppreference operations and others I assume were meant to disable or weaken defender: flag{copy over when I get home}

For the second flag...
