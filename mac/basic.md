### 注意事项:

mac .zsh 中后台运行程序(&)不能直接exit,会报错，并导致程序退出。

```sh
zsh: you have running jobs.
```

需要执行以下命令保持后台运行

```	shell
nohup java -jar scheduler_job.jar&!
```

