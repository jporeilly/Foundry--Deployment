## <font color='red'>Hitachi Vantara Foundry 2.4.0 Platform</font>  

Ansible playbooks install and configures Hitachi Vantara Foundry Platform.

Prerequisites for the AlamLinux machines:
* A public key generated on your Ansible Controller
* SSH passwordless access on Nodes with root permissions
* Completed 01 Infrastructure section
* Completed 02 Pre-flight section

The following playbooks are run:  

#### preflight-foundry.yml
* Update packages
* Install Helm - all Nodes
* Prepare kubeconfig
* Install jq
* Install kubectl
* Configure kubectl for 'installer' access

#### docker-registry.yml
* Install Docker
* Configure a Docker insecure Registry - Ansible Controller
* Copy over certs to 'installer'

#### storage-class.yml
* Install OpenEBS storage class

#### deploy-foundry.yml
* Creates a Log directory
* Creates a Foundry directory
* Unarchives Foundry Control Plane 2.4.0
* Creates a Metrics directory
* Unarchives Metrics 1.0.0
* Installs Cluster Services
* Run Hitachi CRDs
* Install Foundry Control Plane
* Upload Foundry charts & images
* Install Metrics Addon
* Upload Metrics image

---

<em>Run the playbook - preflight-foundry.yml</em>      
This will update, install and configure the various required packages for the Foundry Platform.
 

``run the playbook - preflight-foundry.yml:`` 
```
cd
cd /etc/ansible/playbooks
ansible-playbook -i hosts-skytap.yml --extra-vars="@extra-vars.yml" -b -v preflight-foundry.yml
```

---

<em>Run the playbook - docker-registry.yml</em>      
This will update, install and configure the various required packages for the Foundry Platform.
 

``run the playbook - preflight-foundry.yml:`` 
```
cd
cd /etc/ansible/playbooks
ansible-playbook -i hosts-skytap.yml --extra-vars="@extra-vars.yml" -b -v preflight-foundry.yml
```

<em>Configure Registry</em>  
Notice that the last few playbooks haven't run.  To complete the playbook tasks:

``restart Docker:``
```
systemctl status docker
systemctl restart docker
```
Note: This is really just a check of the docker service.

``to 'log' the 'installer' user out and in:`` 
```
sudo su - installer 
```
``re-run the playbook - pre-flight_foundry.yml:`` 
```
cd /etc/ansible/playbooks
ansible-playbook -i hosts-skytap.yml --extra-vars="@extra-vars.yml" -b -v preflight-foundry.yml -t continue
```
Note:  This will pick up the playbook from the continue tag onwards.

---

<em>Run the playbook - install_foundry.yml</em> 

``run the playbook - install_foundry.yml:`` 
```
cd /etc/ansible/playbooks
ansible-playbook -i hosts-skytap.yml --extra-vars="@extra-vars.yml" -b -v deploy-foundry.yml
```
Note: It will take about 10mins to unachive the Foundry Platform package.  

you should have some logs appearing.  
``tail install-cluster-services.log: (new terminal)``
```
cd /installers/logs
ls
tail -f install-cluster-services.log
```
``check namespaces:``
```
kubectl get ns
```
Note: wait until all the cluster services have been installed, otherwise not all the namespaces will appear.  
``check the pods:``
```
kubectl get pods -A
```
``to access the Foundry Solutions Control Plane:``
```
username: foundry
echo $(kubectl get keycloakusers -n hitachi-solutions keycloak-user -o jsonpath="{.spec.user.credentials[0].value}")
```
or if you have configured .kubectl_aliases, just type ``foundry`` at command prompt.

---

<em>.kubectl_aliases</em>  
To save typing out the kubectl commands, in the resources folder there's a kubectl_aliases file which you copy over to your $HOME directory.

<font color='green'>The .kubectl_alias has been configured.</font>

``add the following to your .bashrc/.zshrc file:``
```
[ -f ~/.kubectl_aliases ] && source ~/.kubectl_aliases
```

If you want to print the full command before running it.   

``add this to your .bashrc or .zshrc file:``
```
function kubectl() { echo "+ kubectl $@">&2; command kubectl $@; }
```

For further information:

> browse to: https://github.com/ahmetb/kubectl-aliases

---