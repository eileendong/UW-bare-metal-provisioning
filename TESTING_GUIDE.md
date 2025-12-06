# Quick Test Guide - Single Windows Computer

## Prerequisites
-  Windows computer with Windows already installed
-  Administrator access to Windows computer
-  Linux/WSL machine for Ansible control
-  Both computers on same network

## Step-by-Step Test

### 1️ Prepare Windows Computer (5 minutes)

Open PowerShell as Administrator on Windows and run:

```powershell
# Enable WinRM
Enable-PSRemoting -Force
winrm quickconfig -q -force

# Configure authentication
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'

# Set execution policy
Set-ExecutionPolicy RemoteSigned -Force

# Allow through firewall
netsh advfirewall firewall add rule name="WinRM-HTTP" dir=in localport=5985 protocol=TCP action=allow

# Set network to Private (IMPORTANT!)
Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private

# Test WinRM
Test-WSMan -ComputerName localhost
```

If the last command shows XML output, WinRM is working

### 2️ Set Up Ansible Control Machine (5 minutes)

On your Linux/WSL machine:

```bash
# Install Ansible and WinRM support
pip install ansible pywinrm

# Install Windows collections
ansible-galaxy collection install ansible.windows
ansible-galaxy collection install community.windows

# Verify installation
ansible --version
```

### 3️ Configure Test Files (2 minutes)

Edit `test-inventory.ini`:

```ini
[test_computer]
# Replace this with your Windows computer's actual IP
test-ws ansible_host=192.168.1.100

[test_computer:vars]
# Replace with your actual Windows credentials
ansible_user=Administrator
ansible_password=YourActualPassword
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
ansible_port=5985
```

**IMPORTANT:** Update these values:
- `192.168.1.100` → Your Windows computer's IP address
- `Administrator` → Your Windows username
- `YourActualPassword` → Your Windows password

### 4 Test Connection (1 minute)

```bash
# Simple ping test
ansible test_computer -i test-inventory.ini -m win_ping

# Expected output:
# test-ws | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

If you see "SUCCESS" and "pong", you're connected! 🎉

### 5️ Run Test Playbook (5-10 minutes)

```bash
# Run the test playbook
ansible-playbook -i test-inventory.ini test-playbook.yml

# Or with verbose output to see what's happening
ansible-playbook -i test-inventory.ini test-playbook.yml -vv
```

This will:
-  Test connection
-  Install Google Chrome
-  Install VLC
-  Install Python 3.13
-  Create a test "ladmin" user
-  Verify everything installed

### 6️⃣ Verify on Windows

On your Windows computer, check:
- Google Chrome should be installed
- VLC should be installed
- Python 3.13 should be installed
- "ladmin" user should exist (check Computer Management → Local Users and Groups)

## Troubleshooting

### "Connection timeout" or "Connection refused"
**Fix:**
1. Check firewall on Windows (port 5985)
2. Verify WinRM service is running: `Get-Service WinRM`
3. Test WinRM locally on Windows: `Test-WSMan localhost`
4. Ping the Windows computer from Linux: `ping 192.168.1.100`

### "Bad HTTP authentication header"
**Fix:**
1. Check username/password in inventory.ini
2. Make sure Basic auth is enabled in WinRM (see step 1)

### "Network path not found"
**Fix:**
1. Ensure network profile is Private: `Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private`
2. Restart WinRM service: `Restart-Service WinRM`

### "Winget not found" during software installation
**Fix:**
1. Windows 11: Winget should be pre-installed
2. Windows 10: Install App Installer from Microsoft Store
3. Or install manually: https://github.com/microsoft/winget-cli/releases

### Software installation fails
**Fix:**
1. Verify internet connection on Windows computer
2. Try installing manually first: `winget install Google.Chrome`
3. Check Windows Update is working

## Test Commands Reference

```bash
# Test connection
ansible test_computer -i test-inventory.ini -m win_ping

# Get Windows info
ansible test_computer -i test-inventory.ini -m setup

# Run a command
ansible test_computer -i test-inventory.ini -m win_shell -a "hostname"

# Check installed software
ansible test_computer -i test-inventory.ini -m win_shell -a "Get-Package | Select-Object Name"

# Run test playbook
ansible-playbook -i test-inventory.ini test-playbook.yml

# Run with verbose output
ansible-playbook -i test-inventory.ini test-playbook.yml -vv

# Dry run (check mode - doesn't make changes)
ansible-playbook -i test-inventory.ini test-playbook.yml --check
```

## What to Test

### Minimal Test (5 minutes)
Just test connection:
```bash
ansible test_computer -i test-inventory.ini -m win_ping
```

### Software Installation Test (10 minutes)
Run the test playbook (installs Chrome, VLC, Python)

### Full System Test (60 minutes)
Once the above works, try the full playbook:
```bash
ansible-playbook -i test-inventory.ini ../cmu302-ansible-deployment/cmu302_workstation_setup.yml
```

## Files You Need

1. **test-inventory.ini** - Defines your Windows computer
2. **test-ansible.cfg** - Ansible configuration
3. **test-playbook.yml** - Simple test that installs a few apps

All three files are in the outputs folder.

## Next Steps

Once this test works:
1. ✅ You've proven Ansible can manage Windows
2. ✅ You've proven software installation works
3. ✅ You're ready to test the full CMU 302 playbook
4. ✅ Then integrate with PXE/MDT for new deployments

## Quick Checklist

Before running test:
- [ ] WinRM enabled on Windows
- [ ] Firewall allows port 5985
- [ ] Network profile set to Private
- [ ] Ansible installed on Linux/WSL
- [ ] test-inventory.ini updated with correct IP/password
- [ ] Both computers on same network
- [ ] Internet access on Windows (for Winget)

After test completes:
- [ ] Connection successful (win_ping works)
- [ ] Chrome installed
- [ ] VLC installed
- [ ] Python installed
- [ ] ladmin user created
- [ ] 
update chris: christopher@wherestarsend.com


---

**Time estimate:** 20-30 minutes total for complete test

**If it works:** You're ready for full deployments! 🚀
