[root@localhost public_folder]# sudo ss -tlnp | grep LISTEN
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=774,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=774,fd=4))
