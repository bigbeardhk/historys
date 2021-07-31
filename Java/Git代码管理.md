##### 1.取消本次commit(commit未提交)
> 选中这次commit记录,然后右键选中"Undo Commit",那么这次commit会被取消,修改代码或重新勾选文件后再次commit
##### 2.git reset
> git reset [--soft | --mixed [-N] | --hard] HEAD~X  <br>
> Reset Type: <br>
> Soft：选择这个模式意思是仅仅撤销commit而已，不影响本地的任何文件，也不影响（index）缓存区的任何文件。<br>
> Hard：不仅撤销commit的内容，还将本地的文件指向commit前的版本，同时index也会指向commit前的版本。<br>
> Mixed：回滚index，其余的保持不变。<br>
> 如果把HEAD后面加个“~1”，这里的数字代表次数，比如commit了三次，  1，就是回滚最后一次提交的，2，就是后两次提交的一起回滚了。
