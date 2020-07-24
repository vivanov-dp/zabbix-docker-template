# zabbix-docker-template
Zabbix 5.x docker template for Zabbix Agent ver.1, with containers and images LLD
_This will probably run on older version of Zabbix too, but is only tested on 5.0._

This doesn't use any external scripts or modules to collect data, the only dependencies are curl and read access to docker's API.

*Intended setup:* Running Zabbix Agent alongside Docker on the same host, access API by UNIX socket.

*Possible:* Run Zabbix Agent separately and point the template to docker host, access API by HTTP.

## Setup
- `cp docker_template.conf /etc/zabbix/zabbix_agentd.d/`
- `sudo usermod -a -G docker zabbix`
- restart Zabbix Agent
- import `docker_template_agent1.xml` into Zabbix templates
    
    This will create a template named **Template App Docker - Agent 1**

- attach the new template to hosts to start monitoring
- configure macros as needed

    `{$DOCKER.SOCKET}` has to point to docker's unix socket (default is `/var/run/docker.sock`)

## Details
The `Docker overview` host screen will be added to monitored hosts - it contains all data, including discovered containers and their stats.

This template uses Docker's API to fetch data and accesses it locally via docker's UNIX socket. 

By changing the items in the `docker_template.conf` file it is possible to set it up to access the API via http - example items are provided as a comment in the file - if you use them, the `{DOCKER.SOCKET}` macro should contain the name of the host to contact.

Unlike the built-in Docker template for Agent v2, this template doesn't extract every possible item from the stats that Docker API returns, but (with few exceptions) only the ones that can be reasonably put into graphs. If you need more - it is trivial to extend by cloning new items.
 
## Performance
The template relies on local JS preprocessing in the Zabbix Server for the low-level discovery.

This is certainly not the most effective way to fetch and prepare data and if you have a really big setup you might need to look at other solutions. 

But to put **really big** into perspective: I have about 20 docker hosts with 4-5 containers each and one with 20 containers and I monitor them with this template running on a pretty modest vhost with 2 vcpu's and 4G RAM.

This amounts to about 80 values per second on the server, CPU load and network traffic is negligible - about 4% and 100Kbit/s respectively. The Zabbix front-end generates more load than that when it renders the graphs.

## Contributions

If you find anything fishy or have a feature request, add a new issue and I'll take a look.

Pull requests appreciated.
