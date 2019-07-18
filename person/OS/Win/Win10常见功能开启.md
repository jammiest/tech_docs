#

## Hyper-V

> PowerShell(管理员权限)

```powershell
# 启用，然后重启电脑
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

> cmd(管理员权限)

```cmd
# 启用，然后重启电脑
bcdedit /set hypervisorlaunchtype auto

# 禁用，然后重启电脑
bcdedit /set hypervisorlaunchtype off
```