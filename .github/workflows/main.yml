name: RDP

on:
  workflow_dispatch:

jobs:
  secure-rdp:
    runs-on: [self-hosted, windows]
    timeout-minutes: 0   # unlimited on self-hosted

    steps:
      - name: Enable RDP and Firewall
        shell: pwsh
        run: |
          Write-Host "Enabling Remote Desktop..."

          Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" `
            -Name "fDenyTSConnections" -Value 0

          Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

          Restart-Service TermService -Force

          Write-Host "RDP Enabled."

      - name: Create RDP User
        shell: pwsh
        run: |
          $password = -join ((33..126) | Get-Random -Count 16 | ForEach-Object {[char]$_})
          $secure = ConvertTo-SecureString $password -AsPlainText -Force

          if (-not (Get-LocalUser -Name "RDP" -ErrorAction SilentlyContinue)) {
            New-LocalUser -Name "RDP" -Password $secure
            Add-LocalGroupMember -Group "Remote Desktop Users" -Member "RDP"
            Add-LocalGroupMember -Group "Administrators" -Member "RDP"
          }

          echo "RDP_USER=RDP" >> $env:GITHUB_ENV
          echo "RDP_PASS=$password" >> $env:GITHUB_ENV

      - name: Install Tailscale
        shell: pwsh
        run: |
          $url="https://pkgs.tailscale.com/stable/tailscale-setup-latest-amd64.msi"
          $out="$env:TEMP\tailscale.msi"
          Invoke-WebRequest $url -OutFile $out
          Start-Process msiexec.exe -ArgumentList "/i `"$out`" /quiet /norestart" -Wait

      - name: Connect Tailscale
        shell: pwsh
        run: |
          & "$env:ProgramFiles\Tailscale\tailscale.exe" up `
            --authkey=${{ secrets.TAILSCALE_AUTH_KEY }} `
            --hostname=rdp-$env:COMPUTERNAME

          Start-Sleep 5
          $ip = & "$env:ProgramFiles\Tailscale\tailscale.exe" ip -4
          echo "TS_IP=$ip" >> $env:GITHUB_ENV

      - name: Show Connection Info
        shell: pwsh
        run: |
          Write-Host ""
          Write-Host "=============================="
          Write-Host " RDP READY"
          Write-Host " Address : $env:TS_IP"
          Write-Host " Username: $env:RDP_USER"
          Write-Host " Password: $env:RDP_PASS"
          Write-Host "=============================="
