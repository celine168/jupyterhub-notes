# An attempt to add a registered, but wiped node

Running `systemctl status kubelet` on chick10 revealed an error code 255. 

Examining the logs using `journalctl --since <time> | grep "kubelet"` shows that
kubelet `config.yaml` file in `/var/lib/kubelet` could not be found. 
The file was not in the directory.

`systemctl restart kubelet.service` didn't do anything.

We copied the config file from another node. 

We looked at the logs again, another file was missing. So, we ran
`sudo scp /etc/kubernetes/bootstrap-kubelet.conf 10.0.0.110:/tmp`
