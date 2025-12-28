example serviec file 

[Unit]
Description=cpetools_api autodeploy uwsgi
After=network.target

[Service]
User=nginx
Group=nginx

KillSignal=SIGQUIT
Type=notify
NotifyAccess=all
TimeoutStartSec=0
RestartSec=10
Restart=always

WorkingDirectory=/opt/cpetools_api_autodeploy/active_deployment/
ExecStart=/opt/cpetools_api_autodeploy/active_deployment/bin/run_venv_cmd uwsgi --ini /etc/cpetools_api_autodeploy/uwsgi.ini --env="DJANGO_CONFIG=/etc/cpetools_api_autodeploy/django.env"

[Install]
WantedBy=multi-user.target

===============
1. Who runs the service?
The [Service] section defines:
INIUser=nginxGroup=nginxShow more lines

The process will run as user nginx and group nginx.
This controls file permissions and access rights for the service.
For example:

It can only read/write files that nginx user/group has permission for.
It cannot access root-only files unless explicitly allowed.




2. How is the service started?

ExecStart=... specifies the command to run:

INIExecStart=/opt/cpetools_api_autodeploy/active_deployment/bin/run_venv_cmd uwsgi --ini /etc/cpetools_api_autodeploy/uwsgi.ini --env="DJANGO_CONFIG=/etc/cpetools_api_autodeploy/django.env"``Show more lines
This means:

It runs uwsgi inside your virtual environment using run_venv_cmd.
It loads configuration from /etc/cpetools_api_autodeploy/uwsgi.ini.
It sets an environment variable DJANGO_CONFIG.


3. How does systemd manage it?

Type=notify → The service uses systemd notifications (uwsgi sends READY signal).
Restart=always → If the process crashes, systemd restarts it.
RestartSec=10 → Wait 10 seconds before restarting.
KillSignal=SIGQUIT → Graceful shutdown signal.
TimeoutStartSec=0 → No timeout for startup.


4. When is it started?
The [Install] section:
INIWantedBy=multi-user.targetShow more lines

This means the service will start when the system reaches multi-user mode (normal boot level for servers).
When you run:

Shellsudo systemctl enable cpetools_api_autodeploy_uwsgi.serviceShow more lines
systemd creates a symlink under:
/etc/systemd/system/multi-user.target.wants/

So it starts automatically at boot.

Summary

Who runs it? → nginx user and group.
When does it start? → At boot, after network is up (After=network.target).
Permissions? → Limited to what nginx user can access.
How is it managed? → Auto-restart, graceful shutdown, systemd notifications.



Diagram: How the Service Starts and Runs
 ┌─────────────────────────────┐
 │        System Boot          │
 └───────────────┬─────────────┘
                 │
                 ▼
 ┌─────────────────────────────┐
 │   multi-user.target (runlevel) │
 │   (Normal server mode)       │
 └───────────────┬─────────────┘
                 │
                 ▼
 ┌─────────────────────────────┐
 │ systemd loads unit file     │
 │ /etc/systemd/system/        │
 │ cpetools_api_autodeploy_uwsgi.service │
 └───────────────┬─────────────┘
                 │
                 ▼
 ┌─────────────────────────────┐
 │ ExecStart command runs:     │
 │ /opt/.../run_venv_cmd uwsgi │
 │ --ini /etc/.../uwsgi.ini    │
 │ --env DJANGO_CONFIG=...     │
 └───────────────┬─────────────┘
                 │
                 ▼
 ┌─────────────────────────────┐
 │ Process runs as:            │
 │ User = nginx                │
 │ Group = nginx               │
 │ Permissions = nginx user    │
 └─────────────────────────────┘


✅ Checklist to Verify Permissions & Startup Behavior


Confirm service file location
Shellsystemctl show -p FragmentPath cpetools_api_autodeploy_uwsgi.serviceShow more lines
Should show /etc/systemd/system/....


Check user and group
Shellgrep -E 'User|Group' /etc/systemd/system/cpetools_api_autodeploy_uwsgi.serviceShow more lines
Should show User=nginx and Group=nginx.


Verify file permissions for nginx
Shellsudo -u nginx ls -l /opt/cpetools_api_autodeploy/active_deployment/sudo -u nginx cat /etc/cpetools_api_autodeploy/uwsgi.iniShow more lines
Ensure nginx can read/write what it needs.


Check startup target
Shellsystemctl list-dependencies multi-user.target | grep cpetools_api_autodeploy_uwsgiShow more lines
Confirms it’s linked to multi-user.target.


Verify auto-start at boot
Shellsystemctl is-enabled cpetools_api_autodeploy_uwsgi.serviceShow more lines
Should return enabled.


Check runtime status
Shellsystemctl status cpetools_api_autodeploy_uwsgi.serviceShow more lines
Look for Active: active (running).


View logs
Shelljournalctl -u cpetools_api_autodeploy_uwsgi.service -eShow more lines