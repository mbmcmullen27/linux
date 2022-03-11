### Add Units

```sh
# create unit files
cat <<EOF > test1.target
[Unit]
Description=test 1
EOF

cat <<EOF > test2.target
[Unit]
Description=test 2
Wants=test1.target
EOF

# copy unit files to system configuration directory 
cp test1.target test2.target /etc/systemd/system/

# start test2.target
systemctl start test2.target

# check status
systemctl status test1.target test2.target
```

- (147) If your unit file has an \[Install\] section, you need to "enable" the unit before activating it
    ```sh
    systemctl enable unit
    ``` 

### Remove Units

```sh
# Deactivate
systemctl stop test2.target
systemctl stop test1.target
```

- (147) If the unit has an \[Install\] section, disable the unit to remove any symbolic links created by the dependency system
    ```sh
    systemctl disable unit
    ```

```sh
# remove the files
rm /etc/systemd/system/test{1,2}.target
```