```
netsh interface portproxy reset
```

```
netsh interface portproxy show all
```

以下是powershell上的命令：

```powershell
# 1) 取当前 WSL 的 IPv4（NAT 下每次可能变化）
$wslIP = (wsl.exe hostname -I).Split()[0]

# 2) 确保 IP Helper 运行（portproxy 依赖它）
sc query iphlpsvc | findstr RUNNING >$null 2>&1; if ($LASTEXITCODE -ne 0) { sc start iphlpsvc | Out-Null }

# 3) 建议确保网卡没禁用 IPv6（禁用 IPv6 会导致只在回环监听）
# （若你之前关过 IPv6，请到网卡属性把“Internet 协议版本 6 (TCP/IPv6)”勾上）

# 4) 清旧规则，显式绑定到你的网卡地址（不要用 0.0.0.0 的玄学默认）
netsh interface portproxy reset
netsh interface portproxy add v4tov4 listenaddress=10.61.149.95 listenport=10088 connectaddress=$wslIP connectport=10088

# 5) 放行 Windows 防火墙的本机入站（仅此端口）
netsh advfirewall firewall add rule name="WSL-SSH-10088" dir=in action=allow protocol=TCP localport=10088

# 6) 两个自检：确认真的在网卡地址监听，并能本机连通
Get-NetTCPConnection -LocalPort 10088 -State Listen | Format-Table -Auto LocalAddress,LocalPort,OwningProcess
Test-NetConnection 10.61.149.95 -Port 10088
```

以下是vbs命令，只需运行一次后，每次开机都会执行：

```vbscript
Set sh=CreateObject("WScript.Shell")
Set fso=CreateObject("Scripting.FileSystemObject")
If Not fso.FolderExists("C:\Scripts") Then fso.CreateFolder("C:\Scripts")
Set f=fso.CreateTextFile("C:\Scripts\wsl_portproxy_refresh.ps1",True)
f.WriteLine "$ErrorActionPreference=""Stop"""
f.WriteLine "$log=""C:\Scripts\wsl_portproxy.log"""
f.WriteLine "function L($m){Add-Content -Path $log -Value ""$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') $m""}"
f.WriteLine "$wslIP="""""
f.WriteLine "for($i=0;$i -lt 30 -and [string]::IsNullOrWhiteSpace($wslIP);$i++){try{$wslIP=(wsl.exe -e sh -lc ""hostname -I | awk '{print $1}'"").Trim()}catch{};Start-Sleep 2}"
f.WriteLine "if([string]::IsNullOrWhiteSpace($wslIP)){L ""no wsl ip"";exit 2}"
f.WriteLine "$hostIP=(Get-NetIPConfiguration|Where-Object{$_.IPv4DefaultGateway -ne $null -and $_.NetAdapter.Status -eq ""Up""}|Select-Object -First 1).IPv4Address.IPAddress"
f.WriteLine "if(-not $hostIP){$hostIP=(Get-NetIPAddress -AddressFamily IPv4|Where-Object{$_.IPAddress -notlike '127.*'}|Select-Object -First 1).IPAddress}"
f.WriteLine "Start-Service iphlpsvc -ErrorAction SilentlyContinue"
f.WriteLine "$port=10088"
f.WriteLine "try{netsh interface portproxy reset|Out-Null}catch{}"
f.WriteLine "netsh interface portproxy add v4tov4 listenaddress=$hostIP listenport=$port connectaddress=$wslIP connectport=$port|Out-Null"
f.WriteLine "netsh advfirewall firewall add rule name=""WSL-$port"" dir=in action=allow protocol=TCP localport=$port|Out-Null"
f.WriteLine "L ""listen $hostIP:$port -> $wslIP:$port"""
f.Close
sh.Run "schtasks /Create /TN WSL-PortProxy-10088 /TR ""powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Scripts\wsl_portproxy_refresh.ps1"" /SC ONSTART /RU SYSTEM /RL HIGHEST /F",0,True
sh.Run "schtasks /Create /TN WSL-PortProxy-10088-Logon /TR ""powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Scripts\wsl_portproxy_refresh.ps1"" /SC ONLOGON /RU SYSTEM /RL HIGHEST /F",0,True
sh.Run "schtasks /Run /TN WSL-PortProxy-10088",0,True
sh.Run "powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -Command ""Get-NetTCPConnection -LocalPort 10088 -State Listen | ft -Auto LocalAddress,LocalPort; Test-NetConnection 127.0.0.1 -Port 10088 | fl TcpTestSucceeded; Test-NetConnection -ComputerName ((Get-NetIPConfiguration|?{$_.IPv4DefaultGateway -ne $null -and $_.NetAdapter.Status -eq 'Up'}|select -First 1).IPv4Address.IPAddress) -Port 10088 | fl TcpTestSucceeded""",0,True
WScript.Echo "installed"

```

### 用法与验证

- 第一次：右键以管理员运行 `install_wsl_portproxy.vbs`，弹出 `installed` 即完成。
- 验证：重启后，执行
     `Get-NetTCPConnection -LocalPort 10088 -State Listen`
     应该看到监听在你的网卡 IPv4（不是 127.0.0.1）。同机或同网段测：
     `Test-NetConnection <你的网卡IP> -Port 10088` 为 True。

### 维护

- 改端口：把 VBS 里 `"$port=10088"` 改成目标端口，重新以管理员运行一次。

- 卸载任务：管理员 PowerShell 执行
     `schtasks /Delete /TN WSL-PortProxy-10088 /F`
     `schtasks /Delete /TN WSL-PortProxy-10088-Logon /F`

​    
