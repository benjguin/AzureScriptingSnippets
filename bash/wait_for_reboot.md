# Wait for a number of linux VMs to reboot

## What we want to achieve 

We want to reboot a few Linux VMs and wait for them to be ready before continuing.

## Sample variables initialization

```bash
nbServers=5
username=myuser
```

## Implementation

In this example, the VMs have IP addresses starting with `192.168.0.2`.

As we nay want to wait for reboots several times in the same script, it may be good to have this piece of code as an external file.

/tmp/wait4reboots_src.sh:
```bash
# this file is intended to be sourced
for mysrv in "${myservers[@]}"
do
    echo "waiting for $mysrv to reboot"
    stay="true"
    tries=0
    while [ "$stay" == "true" ]
    do
        ssh ${username}@${mysrv} whoami
        x=`ssh $mysrv whoami | grep $username | wc -l`
        if [ "$x" == "1" ]
        then
            stay="false"
        else
            if [ $tries -gt 10 ]
            then
                echo "Servers did not reboot correctly"
                exit 1
            fi
            echo "waiting for 30 seconds ..."
            sleep 30s
            ((tries=tries+1))
        fi
    done
done
```

The following script reboots the VMs and waits for them to reboot:

```bash
myservers=()
for (( i=0; i<$nbServers; i++ ))
do
    myservers+=(192.168.0.2$i)
done

# reboot servers
for mysrv in "${myservers[@]}"
do
    ssh $mysrv sudo shutdown -r now
done

# wait for the reboots to finish
source /tmp/wait4reboots_src.sh

echo "done"
```
