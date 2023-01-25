# Install Confluent Platform with cp-ansible 

You can read further more in the guide about how to install [Confluent Platform](https://docs.confluent.io/ansible/current/ansible-download.html), this example is the most basic one you can find. 

## Get cp-ansible collection   

```
ansible-galaxy collection install confluent.platform 
```

## Install CP platform 

I've provided an example [inventory file](inventory.yml) which you just need to edit the hosts names.  
After that, you can run the following commands: 

```
cd ~/.ansible/collections/ansible_collections/confluent/platform
ansible-playbook -i inventory.yml confluent.platform.all
```

The above should install the zookeeper and Confluent server components with the default ports for JMX Exporter and jolokia. 

