On host:
```
 nc -l  127.0.0.1 1235 > out.png
```

On guest:
```
nc -w 2 10.0.2.10 1235 < in.png
```

SSH to guest:
```
ssh -i image/stretch.id_rsa -p 10021 -o "StrictHostKeyChecking no" root@localhost
```
