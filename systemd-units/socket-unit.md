```sh
# copy unit files to system configuration directory 
cp echo.socket echo@.service /etc/systemd/system/

# start echo.socket
systemctl start echo.socket

# test socket
telnet localhost 22222

# enter some text...

# stop unit
systemctl stop echo.socket
```