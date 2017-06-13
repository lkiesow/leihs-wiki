While deploying leihs one may encounter following problems:
### Error when creating / extracting archive with tar 

A possible reason for this may be:
```
Your 'tar' does not support the '--concatenate' option, which we need
to produce a single tarfile. Either install a compatible tar (such as
gnutar), or invoke git-archive-all with the '--separate' option.
``` 
This can be solved by installing and using [gnu-tar](https://www.gnu.org/software/tar/) on the control machine.