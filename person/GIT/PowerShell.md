# PowerShell Git配置

## PowerShell整合Post-git

代码仓库: [https://github.com/dahlbyk/posh-git](https://github.com/dahlbyk/posh-git)

> PowerShell版本要求

Windows PowerShell 5.x 或者 PowerShell Core 6.0

1、查看版本信息

```powershell
$PSVersionTable.PSVersion
```

2、PowerShell脚本执行策略要求：RemoteSigned或者Unrestricted

```powershell
# 查看powershell执行策略
Get-ExecutionPolicy

# 如果powershell的执行策略不是RemoteSigned或者Unrestricted，则执行以下命令
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Confirm
```

3、安装post-git

```powershell
PowerShellGet\Install-Module posh-git -Scope CurrentUser -AllowPrerelease -Force
```

4、为PowellShell导入post-git，该操作是当前会话有效

```powershell
Import-Module posh-git
```

5、持久化PowerShell生效

```powershell
Add-PoshGitToProfile
```