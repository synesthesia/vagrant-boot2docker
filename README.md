silinternational/vagrant-boot2docker
====================================

A Vagrantfile to provisions a boot2docker box, configure shared folders with
translated permissions, install `docker-compose`, and run `docker-compose` to
start containers.

The intent is to provide a Vagrantfile which can be dropped into existing
projects, minimally edited to configure networking and synced folders.

The Vagrantfile is the only notable object in this project; the other files are
just a sort of sample application as a demonstration.

Please note that vagrant 1.7.3 is needed to use VirtualBox 5.0.

Demonstration Use
-----------------
Just clone this repository and run:

    vagrant up

You can then connect to the containerized webserver at
[http://192.168.70.249](http://192.168.70.249) You can edit data/greet.html and
see the changes reflected after a couple reloads of the page.

The Vagrantfile syncs `data/` in the working directory to `/data` in the
vagrant box, bidirectionally converting the ownership and group permissions of
files in the local directory to `33.33` in the vagrant box.

The working directory with untranslated permissions is also synced to 
`/vagrant` in the box.

Then, docker-compose is run within the Vagrantfile against the sample
docker-compose.yml to spin up two containers: the a volume container mapping
`/data` in the vagrant box to `/data` in the container; the second, the
"webserver" container, grabbing its volumes from the first.

Since the files ownership is translated in the vagrant box, when the containers
map their volumes to the vagrant box, they see those translated ownerships.
In the Ubuntu container, 33 is the UID and GID of `www-data`, so the net effect
is that data files can be edited no matter what the platform or filesystem
Vagrant is running on, and the container will still run correctly.

Rationale
---------

Docker provides excellent encapsulation to unify development and operations;
however, especially for web development, it is still sometimes useful to be able
edit files "live", and see the changes take effect immediately. This can be done
easily with Docker using volumes mapped to local folders.

However, two issues arise: first, docker only runs natively on Linux, and
second, the file permissions (ownership and mode) of the local folder may need
to be quite different from those mapped in a volume into the container.

To allow diverse development environments, one can use a Linux virtual machine
to run docker, and for this there is the
[boot2docker](http://boot2docker.io/)
image running in VirtualBox.

The boot2docker project includes both the ISO for the virtual machine, and a
number of installers for different platforms, which hook into VirtualBox to
configure the virtual machine. Unfortunately, (at least at the time 
this Vagrantfile was written), boot2docker does install easily on Windows, is 
deemed unnecessary on Linux, does not not have a consistent method to fix the 
permissions problem, and does not have docker-compose installed.

Now, as VirtualBox does have the capability to share folders and translate their
permissions, it just becomes a matter of configuring it to to do so, which
[Vagrant](https://www.vagrantup.com/)
does admirable. Vagrant can also configure the virtual machines it creates, so
features that boot2docker lacks may be added.

The result is a solution which will work correctly and consistently on all
platforms supported by both VirtualBox and Vagrant, and is also relatively
lightweight.

Features
--------

Docker-compose is installed and run on every `vagrant reload`. If you don't want
to use docker-compose, just change the Vagrantfile so that it doesn't run.

If you have the docker command-line client installed locally, it may be easier to
just connect directly to the docker daemon running in the virtual machine, than
to use `vagrant ssh`. After the first `vagrant up`, a some suggested export
lines will be printed which you can just paste into your shell to set up the ENV
variables necessary to connect. Try them until one works.

The docker daemon running in the virtual machine will save images you build or
pull to persistent storage, but if you consistently delete and rebuild the
virtual machine with `vagrant destroy` and `vagrant up`, it may speed things to
define `DOCKER_IMAGEDIR_HOME` in your shell's environment. If defined as a path
to a folder on the box, the Vagrantfile will use that folder as an image cache,
loading tar archives from that folder to the daemon's image store on boot (with
`docker load`), and saving running images as tar archives to the folder after
docker-compose is run (with `docker save`). This may speed up initial boot times
by avoiding the need to keep pulling an image from the network.

Notes
-----

Windows line endings may be an issue; VirtualBox can synchronize folders and
translate permissions, but the data in the files still needs to be correct.

Implementation Details
----------------------

Boot2docker is designed as an essentially read-only image (albeit with some
persistent storage for docker and ssh keys), so to allow Vagrant to add a key to
allow `vagrant ssh` to work correctly, the Vagrantfile does a number of bind
mounts in on start up.

Python and pip are installed to get docker-compose. Boot2docker is based on
[Tinycore Linux](http://distro.ibiblio.org/tinycorelinux/welcome.html), so the
tce packages are downloaded and installed on the first boot, and reinstalled
from persistent storage thereafter. Pip is updated on every boot to get the
latest version of docker-compose.
