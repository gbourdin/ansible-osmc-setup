# ansible-osmc-setup

## What is this?
This is a set of scripts I've been using to set up a raspberry pi running
osmc to automate TV and Movie downloads using ansible.

These can be run on a freshly installed system.

## What will be installed and configured
* nginx and php-fqm (used to serve rutorrent)
* network mounts, assuming you have any
* rtorrent + rutorrent
* sickchill - To automate TV Show downloads
* radarr - To Automate Movie downloads
* chouchpotato (replaced by radarr, script still available if you want it)

## How to use
### Clone this repo
In your teminal just type:
```bash
git clone https://github.com/gbourdin/ansible-osmc-setup.git
cd ansible-osmc-setup
```
### Set up your variables
#### Mounts
I use some nfs mounts to store downloads, tv shows and movies. I assume you'll
have a similar setup. Downloads at least is required for rtorrent.
To set this up:
* Create a ```main.yml``` file inside ```roles/configure-mounts/vars/``` and 
populate it based on the example in ```group_vars/all```
```yaml
mounts:
  - { name: "Downloads",
      device: "10.0.0.100:/nfs/downloads",
      mount_point: "{{ downloads_mount_point }}",
      fstype: "nfs",
      options: "noatime,noauto,x-systemd.automount,async,nfsvers=3,rsize=65536,wsize=65536,nolock,nofail,local_lock=all,soft,retrans=2,tcp",
      dump: 0,
      passno: 0
    }
  - { name: "Music",
      device: "10.0.0.100:/nfs/music",
      mount_point: "/mnt/Music",
      fstype: "nfs",
      options: "noatime,noauto,x-systemd.automount,async,nfsvers=2",
      dump: 0,
      passno: 0
    }
```
#### sickchill
You'll need to provide your desired user and password (and if you figure how
to restore from a backup, set ```sickchill_restore_backup: true```). In order to
do this:
* Create a ```main.yml``` file inside ```roles/install-sickchill/vars/``` and 
add the following variables with your desired user and password:
```
sickchill_web_user: YOUR_SICKCHILL_USER
sickchill_web_pass: YOUR_SICKCHILL_PASSWORD
```

### Set up what you want and don't want
If you want everything listed above then you're done. If not, edit the sites.yml
file and comment out what you don't want or don't need

### Set up your hosts
Copy the hosts.template file to hosts and replace 127.0.0.1 with 
the ip address for your osmc instance.

### Running the script
* Set up a python3 venv and install the requirements
```bash
python3 -m venv
source /tmp/ansible/bin/activate
pip install -r requirements.txt
```

* Now you're ready to run the playbook!
```bash
ansible-playbook -i hosts site.yml 
```

## Caveats
* The regex that updates kodi's http port is not working correctly as of
OSMC VERSION HERE. This has something to do with the structure of kodi's
settings changing at some point between my first iteration of this and
the current version. You will have to manually update that.

* Scripts to collect backup files are missing and probably will for a while
you don't need these if you're setting up a box from scratch. But I use
them as I've been running this on a fresh installation and then restoring
settings from backups.

* Newer versions of ansible are not working and you will run into this error
which I haven't been able to fix. Nor did I spend much time on it. Just use
the version in requirements.txt
```bash
dpkg: warning: 'ldconfig' not found in PATH or not executable
dpkg: warning: 'start-stop-daemon' not found in PATH or not executable
dpkg: error: 2 expected programs not found in PATH or not executable
Note: root's PATH should usually contain /usr/local/sbin, /usr/sbin and /sbin
```

* rtorrent 0.9.7 is out, but it's not compatible with sickchill so config
has been downgraded to 0.9.6 for now. (There's an issue on the sickchill side
that needs to be addressed)