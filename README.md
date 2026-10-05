AdGuard is the best use of that time, since it's independent of the pool. Here's how I'd do it:

1. Create the container in the Proxmox UI (Create CT). Use the Debian 12 template, unprivileged, 1 core, 512 MB RAM, a 4 GB disk on the NVMe, and DHCP networking. Then set a DHCP reservation for it in your router so its IP never changes, since every device on your network will depend on that address.

2. Install AdGuard Home from inside the container (pct enter <ID>):

apt update && apt install -y curl
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v

3. Finish setup in a browser at http://<container-ip>:3000. The wizard has you pick the admin port (80 is fine) and set an admin login. Choose upstream DNS servers you trust, such as Quad9 or Cloudflare, and leave the default blocklists on to start.

4. Test before switching anything. From your own computer, run nslookup example.com <adguard-ip>. It should return an answer, and a known ad domain should come back blocked.

5. Only then change your router's DHCP DNS setting to point at AdGuard. Devices pick it up as their leases renew, so the change isn't instant.




To remove the “You do not have a valid subscription for this server” popup message while logging in and when refreshing packages to do updates.

You’ll need to SSH to your Proxmox server or use the node console through the PVE web interface. One note ctl-w to search in nano closes the tab in firefox so using an SSH client like putty works better.

Login to your proxmox server via ssh.

you can change directories,

cd /usr/share/javascript/proxmox-widget-toolkit

Make a backup

cp proxmoxlib.js proxmoxlib.js.bak

or run,

cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak

Then open the file in nano

nano /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js

or nano proxmoxlib.js

While in nano use ctrl-w to search for "No valid subscription"

" .data.status.toLowerCase() !== 'active') {

Ext.Msg.show({

title: gettext('No valid subscription'), "

now go to the ! before the == and delete it

it should now look like ".data.status.toLowerCase() == 'active') {"

ctrl-o to save the file ctrl-x and exit nano.

Restart the ProxMox service.

systemctl restart pveproxy.service

Reload your browser tab and log back in.

This just changes the logic of the code from not active to is active.

if you pay for a subscription then you would need to change it back.
