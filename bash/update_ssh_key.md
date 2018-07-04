# Update Authorized keys for a VM behind a jumpbox

## What we want to achieve 

We start with the following situation
```
 --------       ------------       ------------
| Laptop | --- | jumpbox    | ----| myVM       |
| ssh1   |     | ssh1.pub   |     | ssh1.pub   |
|        |     | ssh2       |     |            |
|        |     | $username1 |     | $username2 |
 --------       ------------       ------------
```

The local laptop has private ssh key `ssh1` which is authorized to access both Linux VMs `jumbox` and `myVM`.

Form a network perspective, `laptop` can access `myVM` only thru the jumpbox.

We don't want to copy the private `ssh1` key on the `jumbox`, and we want to be able to access `myVM` from `jumbox`.

so we want to have the following

```
 --------       ------------       ------------
| laptop | --- | jumpbox    | ----| myvm       |
| ssh1   |     | ssh1.pub   |     | ssh1.pub   |
|        |     | ssh2       |     | ssh2.pub   |
|        |     | $username1 |     | $username2 |
 --------       ------------       ------------
```

## Sample variables initialization

```bash
jumpbox="myjumpbox.sample.com"
myvm="myvm"
username1="john"
username1="jane"
```

## Implementation

From a first ssh session on `laptpop`, get the public SSH key, and create an ssh tunnel that will give access to `myvm` thru `jumpbox`:

```bash
scp $username1@$jumpbox:~/.ssh/ssh2.pub /tmp/ssh2_jumpbox.pub
ssh -L 127.0.0.1:8022:$myvm:22 $username1@$jumpbox
```

from another ssh client, also on `laptop`, 127.0.0.1 on port 8022 is `myvm`:

```bash
scp -P 8022 -o StrictHostKeyChecking=no /tmp/ssh2_jumpbox.pub $username2@127.0.0.1:~/.ssh/ssh2_jumpbox.pub
ssh -p 8022 -o StrictHostKeyChecking=no $username2@127.0.0.1 "cat ~/.ssh/ssh2_jumpbox.pub >> ~/.ssh/authorized_keys"
```

now one can connect to the monitoring VM from the jumpbox:

```bash
ssh $username2@$myvm
```
