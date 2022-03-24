```sh
# copy unit files to system configuration directory 
sudo cp loggertest.* /etc/systemd/system/

# enable the timer
systemctl enable loggertest.timer

# start the service
systemctl start loggertest

# view status
systemctl status loggertest
```