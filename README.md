### Deployment demo

1.  Setup VMs on play-with-docker.com

    Go to [labs.play-with-docker.com](https://labs.play-with-docker.com/) and create 3 instances

2.  Setup Swarm

        docker info # --> see "Swarm" status, default is inactive
        docker swarm init --advertise-addr <public IP> # enable swarm

    Join swarm as manager

        # at swarm init host
        docker swarm join-token manager

        # at new node which want to be manager
        docker swarm join --token SWMTKN-1-3pbuiawudg5zrgiiiccw38jrlmn6gkkuhc2npo7kt4441jdpy9-07zifdu7v1xwokxz0lw08ch06 192.168.0.8:2377

    Join swarm as worker

        docker swarm join --token SWMTKN-1-0uybfosx80hbq4cr5gcdk31e9st09hlkg9jc7iwv41f15hz9bn-20r11eplw3vo61q16lh9fp0lk 192.168.0.23:2377

    Check

        docker swarm --help

        docker node --help
        docker node ls # list all nodes joined the swarm
        docker node ps # list running tasks on current node
        docker node ps <name of node> # list running tasks on a targeted node

        docker service --help
        docker service ps <name of service> # list all tasks on every including nodes

3.  Create service

        docker service create alpine ping 8.8.8.8 # create first service
        docker service ls # list all services
        docker service ps <name_of_service> # Check details of a service
        docker container ls # list all containers, now they have additional information from docker service

4.  Update service

        docker service update <service ID> --replicas 3 # update attribute of service: replicas
        docker service ls # check REPLICAS column for updated value
        docker service ps <service name> # list three tasks which contain three containers, two more have just started
        docker service logs -f <service name>

    `docker service update` vs `docker update`

5.  Demo how service re-create a new container to replace one which has been removed

        docker container rm -f <ID of a container belongs to service>
        docker service ls # check number of replicas
        docker service ps <name of service> # check details of service including its histories about
                                            # removing task and creaete new task

6.  Remove a service

        docker service rm <name of service>
        docker service ls # check
        docker container ls # check actual running containers, can has some delay before all containers are stopped and removed

7.  Deploy a stack to swarm by docker-compose.yml file

    See what is inside docker-compose.yml

    Copy sample app files from local to swarm node

        ssh ip172-18-0-41-bqd4v42osm4g00e4ei2g@direct.labs.play-with-docker.com
        scp -r stackdemo ip172-18-0-41-bqd4v42osm4g00e4ei2g@direct.labs.play-with-docker.com:/root/

    Test setup with docker-compose

        # At swarm manager node
        cd stackdemo
        docker-compose up -d

        # Test app
        $ curl http://localhost:8000
        Hello World! I have been seen 1 times.

        $ curl http://localhost:8000
        Hello World! I have been seen 2 times.

        $ curl http://localhost:8000
        Hello World! I have been seen 3 times.

        # Stop app
        docker-compose down --volumes # include remove volume

    Deploy to swarm

        docker stack --help

        # Deploy stack name stackdemo with information from docker-compose.yml
        docker stack deploy --compose-file docker-compose.yml stackdemo

        # Check
        docker stack services stackdemo # list services of this stack

        $ curl http://localhost:8000
        Hello World! I have been seen 1 times.

        $ curl http://localhost:8000
        Hello World! I have been seen 2 times.

        $ curl http://localhost:8000
        Hello World! I have been seen 3 times.

        $ curl http://address-of-other-node:8000
        Hello World! I have been seen 4 times.

        # Open public domain address of one node like to see response
        http://ip172-18-0-63-bqd4v42osm4g00e4ei2g-8000.direct.labs.play-with-docker.com/

    Update replicas for web app

        # At manager node
        docker stack services stackdemo # get service names
        docker service update stackdemo_web --replicas 2

    Remove the stack

        docker stack rm stackdemo
        docker service rm registry

    Leave Swarm

        docker swarm leave # at worker first

        docker swarm leave --force # at manager node
