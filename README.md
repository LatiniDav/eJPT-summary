---
icon: door-closed
---

# Port

## <mark style="color:$primary;">TCP port</mark>

<table><thead><tr><th width="106.20001220703125">Port</th><th width="156">Service</th><th width="252.199951171875">WINDOWS VULN</th><th width="227.80010986328125">LINUX VULN</th></tr></thead><tbody><tr><td>21</td><td>FTP</td><td></td><td><ul><li>FTP vsftpd v2.3.4 [HP]</li></ul></td></tr><tr><td>22</td><td>SSH</td><td></td><td><ul><li>SSH v2 - libssh v0.6.0-0.8.0[HP]</li></ul></td></tr><tr><td>25</td><td>SMTP</td><td></td><td><ul><li>SMTP - hakara smtp v&#x3C;=2.8.9 [HP]</li></ul></td></tr><tr><td>80</td><td>HTTP</td><td><ul><li>WebDAV [HP]</li><li>HTTPFileServer httpd v2.3</li><li>Xoda</li></ul></td><td><ul><li>Apache</li></ul></td></tr><tr><td>139</td><td>NetBIOS (SMB)</td><td><ul><li>SMB v1 [HP]</li><li>SMB MITM</li></ul></td><td><ul><li>SAMBA v3.5.0, v4.4.14, v4.5.10 e v4.6.4</li></ul></td></tr><tr><td>443</td><td>HTTPS</td><td><ul><li>WebDAV [HP]</li></ul></td><td></td></tr><tr><td>445</td><td>SMB/SAMBA</td><td><ul><li>SMB v1 [HP]</li><li>SMB MITM</li></ul></td><td><ul><li>SAMBA v3.5.0, v4.4.14, v4.5.10 e v4.6.4</li></ul></td></tr><tr><td>465</td><td>SMTP + SSL</td><td></td><td><ul><li>SMTP - hakara smtp v&#x3C;=2.8.9 [HP]</li></ul></td></tr><tr><td>587</td><td>SMTP + SSL</td><td></td><td><ul><li>SMTP - hakara smtp v&#x3C;=2.8.9 [HP]</li></ul></td></tr><tr><td>1433</td><td>mssql (Microsoft SQL Server)</td><td></td><td></td></tr><tr><td>3306</td><td>MySQL</td><td></td><td></td></tr><tr><td>3389</td><td>RDP</td><td><ul><li>RDP - XP/Vista/7/2008</li></ul></td><td></td></tr><tr><td>5985</td><td>WinRM</td><td></td><td></td></tr><tr><td>5986</td><td>WinRM (HTTPS)</td><td></td><td></td></tr><tr><td>8080</td><td>HTTP</td><td><ul><li>Apache Tomcat</li></ul></td><td></td></tr></tbody></table>

***

## <mark style="color:$primary;">UDP port</mark>

<table><thead><tr><th width="112">Port</th><th></th><th></th></tr></thead><tbody><tr><td>161</td><td>SNMP (queries)</td><td></td></tr><tr><td>162</td><td>SNMP (traps = notifications)</td><td></td></tr></tbody></table>
