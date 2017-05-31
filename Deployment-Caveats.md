While deploying leihs one may encounter following problems:
### Error when creating archive with tar 
```
Your 'tar' does not support the '--concatenate' option, which we need
to produce a single tarfile. Either install a compatible tar (such as
gnutar), or invoke git-archive-all with the '--separate' option.
``` 
This can be solved by installing [gnu-tar](https://www.gnu.org/software/tar/) on the control machine.