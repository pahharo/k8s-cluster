# Kubernetes in a box - Three nodes deployment + Scaling

Deploy kubernetes on the fly, the project creates a ``master node and two minions nodes``,
it uses **Vagrant** with **KVM** as infrastecture provider (IaaS) and **Ansible** as configuration manager
to automatically have a ready and functional **kubernetes cluster** in less than 15 minutes.

As an extra the entire **kubernetes cluster** can be scalable if wished, the project has a **k8s-scale** project
to **add a new minion** to the existing cluster.

## 1. Pre-requisites

* Localhost machine with Linux distribution
* KVM
* Vagrant 1.8.1 or higher
* Vagrant libvirt plugin for KVM
* Ansible 2.2.1 or higher

## 2. Prepare your localhost environment

The first thing is check that you localhost support virtualization, just type 
``egrep -c '(vmx|svm)' /proc/cpuinfo`` if the result is ``0``, your localhost does not support it, 
in other case ``> 0``, means you locahost support virtualization, but also must be ensured it is enable 
in the BIOS. Once it is assured procced to install ``kvm`` (https://www.linux-kvm.org/page/Downloads).

Next to complete the environment and reproduce the ``kubernetes cluster``, 
with the use of ``Vagrant`` just install it (https://www.vagrantup.com/) on your localhost and must be 
installed ``Ansible`` also (http://docs.ansible.com/ansible/latest/intro_installation.html).

In the other hand to use ``kvm`` to setup kubernetes cluster nodes, must be installed the ``vagrant libvirt provider``,
it is a ``plugin`` for ``vagrant`` to use with ``libvirt`` API to manage ``kvm`` as infraestructure provider.
(https://github.com/vagrant-libvirt/vagrant-libvirt#installation) 

Finally we need a ``Public RSA Key`` to inject in the ``Kubernetes Cluster`` nodes, therefore if you have already 
one fine, it is going to be used later, otherwise proceed to ``Generate SSH Keys`` in your localhost
(https://www.cyberciti.biz/faq/linux-unix-generating-ssh-keys/)

## 3. Setup your kubernetes cluster

* In the localhost just clone the repository   
   ``git clone https://github.com/pahharo/k8s-cluster.git``

* Go inside the folder k8s-cluster  
   ``cd k8s-cluster``

* Set your ``Publis RSA Key`` in the script ``scripts/prepare_cluster.py``, generally it is located in ~/.ssh/id_rsa.pub

* Start up the ``Kubernetes Cluster``  
   ``vagrant up --provider libvirt``

That's all ...

## 4. Check your Kubernetes Cluster

Now check the entire cluster with the next tips

* Go to minion-1 node and check nodes  
  ``vagrant ssh minion-1``  
  ``kubectl -s http://10.10.10.51:8080 get nodes`` it must show the two minions nodes ready and working

* Go to minion-2 node and check nodes  
  ``vagrant ssh minion-2``  
  ``kubectl config set-cluster test-cluster --server=http://10.10.10.51:8080``  
  ``kubectl config set-context test-cluster --cluster=test-cluster``  
  ``kubectl config use-context test-cluster``  
  ``kubectl get nodes`` it must show the two minions nodes ready and working

* Go to master node and check nodes  
  ``vagrant ssh master``  
  ``kubectl cluster-info``  

The above command must be show someting similar to:  
   
>Kubernetes master is running at http://localhost:8080   
>KubeDNS is running at http://localhost:8080/api/v1/proxy/namespaces/kube-system/services/kube-dns

## 5. Scale the existing Kubernetes Cluster

Follow the next steps to scale up your entire cluster

* Go inside the folder k8s-scale   
  ``cd k8s-cluster/k8s-scale/``   

* Start up the new Minion node to add in the ``Kubernetes Cluster``  
   ``vagrant up --provider libvirt``  

* Go to master node and check minion nodes (Must be appears three nodes)  
  ``vagrant ssh master``  
  ``kubectl get nodes``  

* Deploy a new pod inside the new minion node  
  ``Replace file in /tmp/avg-api-rc.yml in line 6 **replicas: 2** key by **replicas: 3**``  
  ``kubectl replace rc --filename=/tmp/avg-api-rc.yml``  

* Check the new pod deployed in the new minion node (Must be appears three avg-api-controller-*, one of these deployed
  in the new minion node)  
  ``kubectl get pods``  

That's all, cluster has been scaled up !!!

## 6.Credits

Thanks also to my partners Noel Ruiz Lopez and Jorge Valderrama for the contributions
