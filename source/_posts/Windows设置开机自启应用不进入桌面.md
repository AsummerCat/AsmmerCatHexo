---
title: Windows设置开机自启应用不进入桌面
date: 2024-05-22 23:28:23
tags: [windows]
---
# 基础流程概念
希望在系统启动的时候不进去桌面,并且自启动应用:
考虑在启动时直接启动一个全屏的应用程序，而不显示桌面。这可以通过配置启动项来实现。

具体步骤可能因操作系统而异，以下是针对 Windows 操作系统的一般指南：

1.配置自动登录：在控制面板或设置中，找到用户账户设置，并启用自动登录。这样系统在启动时会自动登录到指定的用户账户。

2.创建全屏应用程序：你可以创建一个简单的程序或脚本，该程序会启动JAR包和H5页面，并确保H5页面全屏显示。可以使用编程语言如Python、Java等编写一个小型应用程序来实现这一点。确保程序在启动时最大化窗口或将其设置为全屏模式。

3.将程序添加到启动项：在开始菜单中搜索"运行"，打开"运行"对话框，输入shell:startup并按Enter键。这将打开启动文件夹。将你的全屏应用程序的快捷方式或可执行文件复制到这个文件夹中。这样，系统在启动时会自动运行该程序。

4.这样配置后，系统将在启动时自动登录到用户账户，并且你的全屏应用程序会启动，不会显示桌面，而是直接打开JAR包和H5页面，并确保H5页面全屏显示。

<!--more-->
## 注意事项:
```

这取决于你想要启动的程序是要在系统启动时运行，还是在用户登录后运行。

1.管理员账户的窗口界面操作：

如果你想要在系统启动时就运行程序，你需要使用管理员账户来将程序添加到启动项。这样，无论哪个用户登录到系统，该程序都会在系统启动时自动运行。
登录到自定义的用户界面里操作：

2.
如果你想要在特定用户登录后运行程序，你可以登录到该用户的账户中，然后按照上述步骤将程序添加到启动项中。这样，只有该用户登录到系统后，程序才会自动运行。
因此，根据你的需求，你可以选择在管理员账户的窗口界面操作还是登录到特定用户账户中进行操作。
```

## 第一步全局设置
```
打开 PowerShell：在开始菜单中搜索“PowerShell”，然后以管理员身份运行 PowerShell。右键单击 PowerShell 快捷方式，选择“以管理员身份运行”。

启用执行策略：如果你从未在系统上运行过 PowerShell 脚本，可能需要先启用 PowerShell 执行策略以允许运行脚本。为此，可以使用以下命令：
Set-ExecutionPolicy RemoteSigned
```

## 第二步 建立自动登录脚本
脚本名称: configure_auto_login.ps1
`ps: 这个脚本是用来自动登录的,设置自动登录功能，以便系统在启动时自动登录到指定的用户账户,如果我们运行的是管理员账户 那么就不用这个脚本了`

```
# 设置自动登录的用户名和密码    Windows 通常要求密码具有一定的复杂性，包括包含至少一个大写字母、一个小写字母、一个数字和一个特殊字符等。
$Username = "自定义用户名"
$Password = ConvertTo-SecureString "自定义用户名密码" -AsPlainText -Force

# 创建自动登录凭据
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username, $Password

# 设置自动登录
Set-LocalUser -Name $Username -PasswordNeverExpires $true
Set-LocalUser -Name $Username -Password $Credential

# 设置自动登录注册表项
$RegPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
Set-ItemProperty -Path $RegPath -Name "AutoAdminLogon" -Value "1"
Set-ItemProperty -Path $RegPath -Name "DefaultUsername" -Value $Username
Set-ItemProperty -Path $RegPath -Name "DefaultPassword" -Value $Password
Set-ItemProperty -Path $RegPath -Name "ForceAutoLogon" -Value "1"

# 添加输出以指示脚本已经执行完毕
Write-Output "自动登录设置已完成。"

# 显式退出脚本
exit
```

## 第三步 创建应用自启动脚本
脚本名称: start_fullscreen_app.ps1

### 第一种脚本 -chrome浏览器
```
# 启动 JAR 包 比如我的jar包地址是:C:\Users\Administrator\Desktop\1.jar
Start-Process -FilePath "java" -ArgumentList "-jar", "C:\Users\Administrator\Desktop\1.jar" -NoNewWindow

# 启动谷歌浏览器并打开 H5 页面  比如我的H5地址是C:/Users/Administrator/Desktop/1.html
Start-Process -FilePath "chrome" -ArgumentList "--kiosk file:///C:/Users/Administrator/Desktop/1.html" -NoNewWindow

# 如果带上 "--start-fullscreen"参数则表示隐藏浏览器的任何信息 直接显示全屏
Start-Process -FilePath "chrome" -ArgumentList "--app=file:///C:/Users/Administrator/Desktop/1.html", "--start-fullscreen" -NoNewWindow

```
### 第一种脚本 -Edge浏览器
```
# 启动 JAR 包
Start-Process -FilePath "java" -ArgumentList "-jar", "C:\Users\Administrator\Desktop\1.jar" -NoNewWindow

# 启动Edge浏览器并打开 H5 页面
Start-Process -FilePath "msedge" -ArgumentList "--kiosk file:///C:/Users/Administrator/Desktop/1.html" -NoNewWindow

```
### 第二种脚本 自启动jar包并且运行成功后->执行打开浏览器
```
# 启动 JAR 包并保持在后台运行
$JarProcess = Start-Process -FilePath "java" -ArgumentList "-jar", "C:\Users\Administrator\Desktop\1.jar" -PassThru

# 等待一段时间（例如10秒）以确保JAR包已经启动
Start-Sleep -Seconds 10

# 检查JAR包进程是否在运行
if ($JarProcess -ne $null -and !$JarProcess.HasExited) {
    Write-Output "JAR包已成功启动并在后台运行。"

    # 启动谷歌浏览器并打开 H5 页面
    Start-Process -FilePath "msedge" -ArgumentList "--kiosk file:///C:/Users/Administrator/Desktop/1.html" -NoNewWindow
} else {
    Write-Output "无法启动JAR包。"
}

```

解释:
```
全屏显示浏览器页面的方式在不同的浏览器中可能会有所不同，因此启动不同浏览器的脚本参数也会不同。在 PowerShell 脚本中，你需要使用浏览器支持的特定参数来实现全屏显示。

对于 Chrome 和 Edge 浏览器：

在 Chrome 中，使用 --start-fullscreen 参数可以将浏览器窗口设置为全屏模式。

在 Edge 中，使用 --kiosk 参数可以以类似全屏的方式启动浏览器，但是它还会隐藏一些浏览器界面元素，例如地址栏、工具栏等。
```

## 第四步  添加到系统启动项中 让他开机自启动

#### 脚本方式:
请注意，此脚本需要以管理员权限运行，因为要将脚本复制到启动文件夹中。你可以将此脚本保存为一个文件，例如 `configure_auto_startup.ps1`，然后以管理员身份运行它。 ->本地运行一次即可,会新增到启动项中循环触发

######  脚本一: 自启判断存在这个脚本,则不新增处理
```
# 启动 JAR 包并保持在后台运行
$JarProcess = Start-Process -FilePath "java" -ArgumentList "-jar", "C:\Users\Administrator\Desktop\1.jar" -PassThru

# 等待一段时间（例如10秒）以确保JAR包已经启动
Start-Sleep -Seconds 10

# 检查JAR包进程是否在运行
if ($JarProcess -ne $null -and !$JarProcess.HasExited) {
    Write-Output "JAR包已成功启动并在后台运行。"

    # 启动 Microsoft Edge 并打开 H5 页面
    Start-Process -FilePath "msedge" -ArgumentList "--kiosk file:///C:/Users/Administrator/Desktop/1.html" -NoNewWindow

    # 检查启动项中是否已存在脚本的快捷方式
    $StartupFolder = [Environment]::GetFolderPath("Startup")
    $ShortcutPath = Join-Path -Path $StartupFolder -ChildPath "start_fullscreen_app.ps1"
    if (-not (Test-Path $ShortcutPath)) {
        # 如果不存在快捷方式，则将脚本添加到启动项中
        Copy-Item -Path $MyInvocation.MyCommand.Path -Destination $ShortcutPath -Force
        Write-Output "脚本已添加到启动项。"
    } else {
        Write-Output "启动项中已存在脚本的快捷方式，无需再次添加。"
    }
} else {
    Write-Output "无法启动JAR包。"
}
```

##### 脚本2: 自启判断存在这个脚本,则删除后新增
```
# 启动 JAR 包并保持在后台运行
$JarProcess = Start-Process -FilePath "java" -ArgumentList "-jar", "C:\Users\Administrator\Desktop\1.jar" -PassThru

# 等待一段时间（例如10秒）以确保JAR包已经启动
Start-Sleep -Seconds 10

# 检查JAR包进程是否在运行
if ($JarProcess -ne $null -and !$JarProcess.HasExited) {
    Write-Output "JAR包已成功启动并在后台运行。"

    # 启动 Microsoft Edge 并打开 H5 页面
    Start-Process -FilePath "msedge" -ArgumentList "--kiosk file:///C:/Users/Administrator/Desktop/1.html" -NoNewWindow

    # 获取启动项中脚本的快捷方式路径
    $StartupFolder = [Environment]::GetFolderPath("Startup")
    $ShortcutPath = Join-Path -Path $StartupFolder -ChildPath "start_fullscreen_app.ps1"

    # 如果启动项中已存在快捷方式，则删除现有的快捷方式
    if (Test-Path $ShortcutPath) {
        Remove-Item -Path $ShortcutPath -Force
        Write-Output "已删除现有的快捷方式。"
    }

    # 将脚本添加到启动项中
    Copy-Item -Path $MyInvocation.MyCommand.Path -Destination $ShortcutPath -Force
    Write-Output "脚本已添加到启动项。"
} else {
    Write-Output "无法启动JAR包。"
}

```

#### 手动运行方式:
```
打开启动文件夹：

1.在开始菜单中搜索"运行"，然后打开"运行"对话框。
2.在运行对话框中输入 shell:startup，然后按 Enter 键。这将打开启动文件夹。
3.将程序添加到启动文件夹：

在打开的启动文件夹中，将你的程序的快捷方式或可执行文件复制到这个文件夹中。
这样，系统在启动时会自动运行该程序。
通过这种方式，你可以手动将程序添加到启动项，以便在系统启动时自动运行。
```


## 异常问题处理:
1. 运行脚本提示:`Start-Process : 由于出现以下错误，无法运行此命令: 系统找不到指定的文件。`
```
指定浏览器路径:

Chrome的默认路径: C:\Program Files\Google\Chrome\Application\chrome.exe 
Edge的默认路径: C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe


比如这样:
Start-Process -FilePath "C:\Program Files\Google\Chrome\Application\chrome.exe" -ArgumentList "--kiosk file:///C:/Users/Administrator/Desktop/1.html" -NoNewWindow

```