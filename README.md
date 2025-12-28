# telegram-signal-copier
Windows-based Telegram Signal Copier for Trading

# Telegram Signal Copier for Windows

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![Platform](https://img.shields.io/badge/platform-Windows-success)
![License](https://img.shields.io/badge/license-MIT-green)

A complete Windows application for copying trading signals from Telegram channels to MT4/MT5 brokers.

## 📥 Download & Install

### Quick Install
1. Download the latest installer: [TelegramSignalCopier-Setup.exe](https://github.com/YOUR_USERNAME/telegram-signal-copier/releases/latest/download/TelegramSignalCopier-Setup.exe)
2. Run the installer as Administrator
3. Follow the setup wizard
4. Launch from Start Menu or Desktop shortcut

### Manual Installation
If you prefer manual installation:
```bash
# Download the portable version
curl -LO https://github.com/YOUR_USERNAME/telegram-signal-copier/releases/latest/download/TelegramSignalCopier-Portable.zip

# Extract to your preferred location
Expand-Archive TelegramSignalCopier-Portable.zip -DestinationPath C:\TelegramSignalCopier

# Run the application
C:\TelegramSignalCopier\TelegramSignalCopier.exe


powershell
# Telegram Signal Copier Installer
# Version: 1.0.0
# Author: Your Name

param(
    [switch]$Silent,
    [string]$InstallPath = "C:\Program Files\Telegram Signal Copier"
)

$ErrorActionPreference = "Stop"

# Check if running as Administrator
$isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")
if (-not $isAdmin) {
    Write-Host "This installer requires Administrator privileges." -ForegroundColor Red
    Write-Host "Please run PowerShell as Administrator and try again." -ForegroundColor Yellow
    pause
    exit 1
}

function Write-Log {
    param([string]$Message)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    Write-Host "[$timestamp] $Message"
}

function Show-Header {
    Clear-Host
    Write-Host "=========================================" -ForegroundColor Cyan
    Write-Host "  TELEGRAM SIGNAL COPIER INSTALLER      " -ForegroundColor Cyan
    Write-Host "=========================================" -ForegroundColor Cyan
    Write-Host ""
}

# Main installation function
function Install-Application {
    Show-Header
    Write-Log "Starting installation..."
    
    # Check prerequisites
    Write-Log "Checking prerequisites..."
    
    # Check .NET 6.0
    $dotnetPath = Get-Command dotnet -ErrorAction SilentlyContinue
    if (-not $dotnetPath) {
        Write-Log ".NET 6.0 not found. Installing..."
        Install-DotNet
    } else {
        Write-Log ".NET 6.0 already installed."
    }
    
    # Create installation directory
    Write-Log "Creating installation directory: $InstallPath"
    New-Item -ItemType Directory -Force -Path $InstallPath | Out-Null
    
    # Copy application files
    Write-Log "Copying application files..."
    $sourceDir = $PSScriptRoot
    
    # Define files to copy
    $files = @(
        "TelegramSignalCopier.exe",
        "TelegramSignalCopier.dll",
        "appsettings.json",
        "README.md",
        "LICENSE"
    )
    
    foreach ($file in $files) {
        if (Test-Path "$sourceDir\$file") {
            Copy-Item "$sourceDir\$file" "$InstallPath\" -Force
            Write-Log "  Copied: $file"
        }
    }
    
    # Create configuration file if doesn't exist
    $configPath = "$InstallPath\appsettings.json"
    if (-not (Test-Path $configPath)) {
        Write-Log "Creating default configuration..."
        $defaultConfig = @"
{
  "Telegram": {
    "ApiId": "",
    "ApiHash": "",
    "PhoneNumber": "",
    "SessionPath": "session.dat"
  },
  "Application": {
    "AutoStart": true,
    "MinimizeToTray": true,
    "CheckForUpdates": true
  },
  "Brokers": []
}
"@
        $defaultConfig | Out-File -FilePath $configPath -Encoding UTF8
    }
    
    # Create desktop shortcut
    Write-Log "Creating desktop shortcut..."
    $WshShell = New-Object -ComObject WScript.Shell
    $Shortcut = $WshShell.CreateShortcut("$env:USERPROFILE\Desktop\Telegram Signal Copier.lnk")
    $Shortcut.TargetPath = "$InstallPath\TelegramSignalCopier.exe"
    $Shortcut.WorkingDirectory = $InstallPath
    $Shortcut.Save()
    
    # Create start menu shortcut
    Write-Log "Creating Start Menu shortcut..."
    $startMenuPath = "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Telegram Signal Copier"
    New-Item -ItemType Directory -Force -Path $startMenuPath | Out-Null
    $Shortcut2 = $WshShell.CreateShortcut("$startMenuPath\Telegram Signal Copier.lnk")
    $Shortcut2.TargetPath = "$InstallPath\TelegramSignalCopier.exe"
    $Shortcut2.WorkingDirectory = $InstallPath
    $Shortcut2.Save()
    
    # Add to startup (if enabled)
    if (-not $Silent) {
        $addToStartup = Read-Host "Add to Windows startup? (Y/N)"
        if ($addToStartup -eq 'Y' -or $addToStartup -eq 'y') {
            Write-Log "Adding to Windows startup..."
            $regPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
            $regName = "TelegramSignalCopier"
            $regValue = "`"$InstallPath\TelegramSignalCopier.exe`" --minimized"
            New-ItemProperty -Path $regPath -Name $regName -Value $regValue -Force | Out-Null
        }
    }
    
    # Create uninstaller
    Write-Log "Creating uninstaller..."
    $uninstallScript = @"
# Uninstaller for Telegram Signal Copier
`$installPath = "$InstallPath"

Write-Host "Uninstalling Telegram Signal Copier..." -ForegroundColor Yellow

# Remove shortcuts
Remove-Item "$env:USERPROFILE\Desktop\Telegram Signal Copier.lnk" -ErrorAction SilentlyContinue
Remove-Item "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Telegram Signal Copier" -Recurse -Force -ErrorAction SilentlyContinue

# Remove from startup
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "TelegramSignalCopier" -ErrorAction SilentlyContinue

# Remove installation directory
if (Test-Path `$installPath) {
    Remove-Item `$installPath -Recurse -Force
}

Write-Host "Uninstallation complete!" -ForegroundColor Green
pause
"@
    
    $uninstallScript | Out-File "$InstallPath\Uninstall.ps1" -Encoding UTF8
    
    Show-Header
    Write-Log "Installation completed successfully!" -ForegroundColor Green
    Write-Log ""
    Write-Log "Installation Summary:" -ForegroundColor Cyan
    Write-Log "  - Installed to: $InstallPath"
    Write-Log "  - Desktop shortcut created"
    Write-Log "  - Start Menu shortcut created"
    Write-Log ""
    Write-Log "Next Steps:" -ForegroundColor Yellow
    Write-Log "  1. Launch 'Telegram Signal Copier' from Desktop or Start Menu"
    Write-Log "  2. Get your Telegram API credentials from https://my.telegram.org"
    Write-Log "  3. Add channels to monitor"
    Write-Log "  4. Configure your broker connections"
    Write-Log ""
    
    if (-not $Silent) {
        $launchNow = Read-Host "Launch Telegram Signal Copier now? (Y/N)"
        if ($launchNow -eq 'Y' -or $launchNow -eq 'y') {
            Start-Process "$InstallPath\TelegramSignalCopier.exe"
        }
    }
}

function Install-DotNet {
    Write-Log "Downloading .NET 6.0 Runtime..."
    
    $downloadUrl = "https://dotnet.microsoft.com/download/dotnet/thank-you/runtime-desktop-6.0.25-windows-x64-installer"
    $installerPath = "$env:TEMP\dotnet-runtime.exe"
    
    # Download .NET installer
    Invoke-WebRequest -Uri $downloadUrl -OutFile $installerPath
    
    Write-Log "Installing .NET 6.0 Runtime (this may take a few minutes)..."
    
    # Silent install
    $process = Start-Process -FilePath $installerPath -ArgumentList "/install /quiet /norestart" -Wait -PassThru
    
    if ($process.ExitCode -eq 0) {
        Write-Log ".NET 6.0 Runtime installed successfully." -ForegroundColor Green
    } else {
        Write-Log "Failed to install .NET 6.0 Runtime. Exit code: $($process.ExitCode)" -ForegroundColor Red
        throw "Failed to install .NET 6.0"
    }
    
    # Cleanup
    Remove-Item $installerPath -Force
}

# Create Windows Service installer (optional)
function Install-Service {
    param([string]$ServiceName = "TelegramSignalCopierService")
    
    $serviceExe = "$InstallPath\TelegramSignalCopier.Service.exe"
    
    if (Test-Path $serviceExe) {
        Write-Log "Installing Windows Service..."
        
        # Check if service exists
        $service = Get-Service -Name $ServiceName -ErrorAction SilentlyContinue
        
        if ($service) {
            Write-Log "Service already exists. Stopping and removing..."
            Stop-Service $ServiceName -Force -ErrorAction SilentlyContinue
            sc.exe delete $ServiceName
        }
        
        # Create new service
        New-Service -Name $ServiceName `
            -BinaryPathName "`"$serviceExe`"" `
            -DisplayName "Telegram Signal Copier Service" `
            -StartupType Automatic `
            -Description "Monitors Telegram channels for trading signals and executes trades automatically." | Out-Null
        
        Write-Log "Service installed successfully." -ForegroundColor Green
        
        # Start the service
        Start-Service $ServiceName
        Write-Log "Service started." -ForegroundColor Green
    }
}

# Main execution
try {
    if ($Silent) {
        Install-Application
    } else {
        Show-Header
        Write-Host "This will install Telegram Signal Copier on your computer." -ForegroundColor White
        Write-Host ""
        Write-Host "Installation path: $InstallPath" -ForegroundColor Yellow
        Write-Host ""
        
        $confirm = Read-Host "Do you want to continue? (Y/N)"
        if ($confirm -eq 'Y' -or $confirm -eq 'y') {
            Install-Application
            
            # Ask about service installation
            $installService = Read-Host "Install as Windows Service (for 24/7 operation)? (Y/N)"
            if ($installService -eq 'Y' -or $installService -eq 'y') {
                Install-Service
            }
        } else {
            Write-Host "Installation cancelled." -ForegroundColor Yellow
        }
    }
} catch {
    Write-Host "Installation failed: $_" -ForegroundColor Red
    Write-Host "Error details:" -ForegroundColor Red
    Write-Host $_.Exception -ForegroundColor Red
    pause
    exit 1
}

# Keep window open if not silent
if (-not $Silent) {
    Write-Host ""
    Write-Host "Press any key to exit..." -ForegroundColor Gray
    $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
}
