################ Upgrade  ####################

    kubeadmin version
    apt-get update && apt upgrade kubeadmin
    kubeadmin upgrade plan

    kubeadmin upgrade apply v1.10.1

############## update nodes ##################

    kubectl get pods -o wide
    kubectl drain node-1 --ignore-daemonsets

    apt-get upgrade kubectl
    systemctl status kubcetl

    kubectl uncordon node-1


############## upgrade the OS ###################

    kubectl get nodes
    kubectl drain node-1 --ignore-daemonsets

Do the upgrade the OS or you can remove the node

    kubectl delete node node-1

check the new tokens and generte new ones


    kubeadm token list
    kubeadm token generate
    kubeadm token create sdads.dsddsd --ttl -3h --print-join-command 
