# Git

许多操作都在提示当前是在游离分支而不能继续，需要` git status`查看当前状态，我遇到过的是之前` git rebase`失败了，需要` git rebase --abort`



> kex_exchange_identification: Connection closed by remote host
> fatal: Could not read from remote repository.
>
> Please make sure you have the correct access rights
> and the repository exists.

尝试`ssh -v git@github.com`，发现没什么问题

最后问题在于开了全局代理，如果想要ssh进行代理

编辑`~/.ssh/config`（如果不存在则需要手动创建

# CI

不同的matrix之间环境是不会共享的，比如说你在第一个matrix中build生成的文件，如果不install或者做其他处理，第二个matrix就会找不到

CI的env会覆盖掉系统的环境变量