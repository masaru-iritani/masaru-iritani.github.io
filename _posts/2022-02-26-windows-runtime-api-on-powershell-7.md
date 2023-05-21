---
layout: post
title: PowerShell 7 で Windows Runtime (WinRT) API を使う
categories: powershell
---

## 結論

Microsoft.Windows.SDK.NET.Ref NuGet パッケージから入手可能な DLL で C#/WinRT が使えます。

## 解説

何らかの理由で PowerShell から Windows Runtime (WinRT) API を使いたいとき、PowerShell 5.1 では次の方法で型を[読み込むことができます](https://stackoverflow.com/a/28077679)。

```powershell
[Windows.Devices.Power.Battery,Windows.Devices.Power,ContentType=WindowsRuntime] | Out-Null
[Windows.Devices.Power.Battery]::AggregateBattery.GetReport()
```

```
ChargeRateInMilliwatts             : 0
DesignCapacityInMilliwattHours     : 43200
FullChargeCapacityInMilliwattHours : 38800
RemainingCapacityInMilliwattHours  : 38800
Status                             : Idle
```

しかし、この方法は PowerShell 7 では使えません。[.NET 5 で WinRT のビルトイン サポートが削除された](https://github.com/dotnet/docs/issues/18875)のが[原因](https://github.com/PowerShell/PowerShell/issues/13042#issuecomment-650567539)のようです。

```powershell
[Windows.Devices.Power.Battery,Windows.Devices.Power,ContentType=WindowsRuntime] | Out-Null
```

```
InvalidOperation: Unable to find type [Windows.Devices.Power.Battery,Windows.Devices.Power, ContentType=WindowsRuntime].
```

代替手段として、[C#/WinRT](https://docs.microsoft.com/ja-jp/windows/apps/develop/platform/csharp-winrt/) の利用が挙げられています。具体的には、[Microsoft.Windows.SDK.NET.Ref NuGet パッケージ](https://www.nuget.org/packages/Microsoft.Windows.SDK.NET.Ref/)に含まれている Microsoft.Windows.SDK.NET.dll と WinRT.Runtime.dll を[読み込めば良い](https://github.com/PowerShell/PowerShell/issues/13042#issuecomment-799077441)ようです。NuGet パッケージの中身は ZIP として展開できるので、例えば次のような手順で自動的に DLL を取得できます。

```powershell
# Set the directory to downoad C#/WinRT DLLs.
# Use $PWD or any other folder instead of $PSScriptRoot outside of a script file.
[string] $pkgPath = Join-Path -Path $PSScriptRoot -ChildPath 'Microsoft.Windows.SDK.NET.Ref'

# Download C#/WinRT DLLs from Microsoft.Windows.SDK.NET.Ref NuGet package if not yet.
if (!(Test-Path -LiteralPath $pkgPath -PathType Container))
{
    [string] $pkgFileName = [IO.Path]::GetTempFileName()
    Invoke-WebRequest -Uri 'https://www.nuget.org/api/v2/package/Microsoft.Windows.SDK.NET.Ref/10.0.22000.23' -OutFile $pkgFileName | Out-Null
    Expand-Archive -LiteralPath $pkgFileName -DestinationPath $pkgPath
    Remove-Item -LiteralPath $pkgFileName -Force -ErrorAction Ignore
}

# Load C#/WinRT DLLs.
Add-Type -AssemblyName (Join-Path -Path $pkgPath -ChildPath 'lib\Microsoft.Windows.SDK.NET.dll') | Out-Null
Add-Type -AssemblyName (Join-Path -Path $pkgPath -ChildPath 'lib\WinRT.Runtime.dll') | Out-Null

# Call a WinRT API.
[Windows.Devices.Power.Battery]::AggregateBattery.GetReport()
```
