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
