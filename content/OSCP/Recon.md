> [!IMPORTANT]
> ALWAYS TRY TO ADD TARGET TO `/etc/hosts`
# Port Scanning (`nmap`)
```
sudo nmap -sC -sV -A --stats-every 0 target.com
```
# Fuzzing (`ffuf`)
`-w` is our wordlist.

```
FILTER OPTIONS:
  -fc                 Filter HTTP status codes from response. Comma separated list of codes and ranges
  -fl                 Filter by amount of lines in response. Comma separated list of line counts and ranges
  -fmode              Filter set operator. Either of: and, or (default: or)
  -fr                 Filter regexp
  -fs                 Filter HTTP response size. Comma separated list of sizes and ranges
  -ft                 Filter by number of milliseconds to the first response byte, either greater or less than. EG: >100 or <100
  -fw                 Filter by amount of words in response. Comma separated list of word counts and ranges
```
## Subdirectories
```
ffuf -u http://target.com/FUZZ -w ~/Downloads/common.txt
```
## Subdomains
This uses vhost fuzzing
```
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w ~/Downloads/common.txt
```
