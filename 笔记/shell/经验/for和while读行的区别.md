```
$ cat file  
aaaa  
bbbb  
cccc dddd  
  
$ cat file | while read line; do echo $line; done  
aaaa  
bbbb  
cccc dddd  
  
$ for line in $(<file); do echo $line; done  
aaaa  
bbbb  
cccc  
dddd
```