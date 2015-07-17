silinternational/vagrant-boot2docker
====================================

Vagrantfile provisions a boot2docker box, configures shared folders with
translated permissions, installs `docker-compose`, and runs `docker-compose`
to create a simple webserver.

As a demonstration, just run

    vagrant up

You can then connect to the containerized webserver at
[http://192.168.70.249](http://192.168.70.249)
