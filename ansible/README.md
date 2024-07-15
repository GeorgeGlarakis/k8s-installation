## Create a Kubernetes Cluster by following the next steps:

1. Install Ansible on the machine that will run the playbook (management device)
2. Set the appriopriate IPs under the [inventory](inventory) file
   1. We need only one Master node, like: 
    ```
    [master_node]
    master ansible_port=22 ansible_host=10.0.100.156
    ```

   2. and multiple Worker Nodes, like:
    ```
    [worker_node]
    10.0.100.157
    10.0.100.158
    ```
3. Change the ssh user under [ansible.cfg](ansible.cfg), by setting the `remote_user` on line 5.
    ```
    ...
    remote_user = glarakis
    ...
    ```
4. Set the user on which Kubernetes will be installed under [kube-main.yml](kube-main.yml), by setting the `ansible_user` on line 11.
    ```
    ...
        - ansible_user = "glarakis"
    ...
    ```
5. Initialize Kubernetes, by running the following command from the management device:
`$ ansible-playbook -i inventory kube-main.yml -K`
The console will wait for root password of the user that connects to the remote server (see Step. 3).