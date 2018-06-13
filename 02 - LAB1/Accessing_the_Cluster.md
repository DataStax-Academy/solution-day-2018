  **[Back to Agenda](./../README.md)**

# Lab 1 - Accessing the Cluster

Open a web browser to your OpsCenter node.
For this cluster, it's running at **http://opscenter_id_address:8888**.   
Find your IP address here: [Cluster IP address](./res/cluster_ip.md).    

![](./img/lab1-1opsc.png)

Mouse over the nodes in your ring. There should be three, with the names node0, node1 and node2. Click on node0.


![](./img/lab1-2opsc.png)

We're now going to SSH into each node and modify a configuration file. You will have to repeat these steps for nodes: node0, node1 and node3. Please refer to the  **[IP list here](./res/cluster_ip.md).**

If you are on a Mac, you already have SSH installed in your terminal. If you are on Windows, you may need to install an SSH client. A popular SSH client is Putty. Putty can be downloaded from [http://www.putty.org](http://www.putty.org).

For this cluster, the username is ds_user. In the terminal I can ssh to the node.

You need to download the key file to your local computer.   
For Linux/Mac user : [Key File](./res/rs.pem)   
For Windows user : [Key File](./res/rs.ppk)   

For this cluster, the username is **ds_user**. So, in the terminal I can ssh to the node by running the command:

```
On Linux/Mac
ssh ds_user@<my_node_ip> -i /path/to/keyfile/rs.pem

On Windows
ssh ds_user@<my_node_ip> -i /path/to/keyfile/rs.ppk

```

You may be prompted to accept the node's key. If so, type "yes" and hit enter.

![](./img/lab1-3sshlogin.png)

Great!

```

dsetool ring

```

![](./img/lab1-4dsetoolstatus.png)

Each node should say the words "Search" and "Analytics" and the Graph's column has the value "yes". If any of them don't, you may have to SSH back into that node and ensure the new configuration is set.

Note that one of the nodes says "(JT)" This is your Spark job track. You can view a webpage with information about Spark jobs by opening a web browser to port 7080 on that node. For this cluster that is at http://my_node_ip:7080. Note your URL will be different.

![](./img/lab1-5sparkjt.png)

We also enabled Solr on our nodes. You can actually view the Solr UI on any node. However, for our exercises we're going to use node0.  Open a web browser to port 8983 /solr/ on node0. For this cluster that is at http://my_node0_ip:8983/solr. The URL will be different for your cluster.

![](./img/lab1-6solrui.png)

Great! You've now logged into the administrative tool, OpsCenter, on your cluster. You've also used SSH to connect to each database node in your cluster. Finally you've logged into the administrative interfaces for both Spark and Solr. Next up we're going to start putting data in the database!



  **[Back to Agenda](./../README.md)**
