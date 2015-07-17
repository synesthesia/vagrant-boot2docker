Provisions a new Ubuntu machine, sets up shared folders and permissions,
installs `docker` and `docker-compose`, and runs `docker-compose` to create
a simple webserver.

To use, just run:

    vagrant up

You can then connect to the containerized webserver at
[http://192.168.70.249](http://192.168.70.249)

Details
-------

The `data` directory is shared into `/data` in VirtualBox, with its permissions,
regardless of what they are in the local folder, translated by VirtualBox. This
is accomplished by the following line in the Vagrantfile:

    config.vm.synced_folder "./data", "/data", owner: "www-data", group: "www-data"

The provided `compose/docker-compose.yml` configures a volume pointed to `/data` in VirtualBox
(mounting it in the container at `/data` also) so that the container's data files will
by synced through the volume, through VirtualBox (with translated permissions) to the
local folder. You can edit `data/greet.html` and see the changes reflected after a couple
reloads of the page.

The `compose` directory is likewise shared into `/home/vagrant/compose`.

The `docker` daemon running in VirtualBox is set up to listen on port 2376; you can run
`docker` and `docker-compose` against it by exporting an environmental variable:

    export DOCKER_HOST=tcp://192.168.70.249:2376

The Vagrantfile uses (via a shell script) `docker-compose` to provision the box. The
installation shell provision scripts are separated from the `docker-compose` provision
script, which is set to run always. Consequently, it is possible to have the box re-run
the `compose/docker-compose.yml` by simply executing:

    vagrant reload
