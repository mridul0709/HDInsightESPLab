## Create HBase Table and add data to the  table

1. SSH into the HDInsight cluster using 
 ````
 ssh hdinsightclusteradmin@<clustername>-ssh.azurehdinsight.com
````
>**Note:**
> 
>Note that instead of using *sshuser* we have instead used the *Cluster Admin* user with domain credentials and the corresponding password that was set for this user in section 1.5.
## Create and test Apache Ranger policies on ESP enabled HDInsight clusters

1. After the HDInsight cluster has been successfully created log into the Ranger portal. The ranger portal can be accessed at the below URL. 

````
   https://<clustername>.azurehdinsight.net/ranger/
````

![Ranger1](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture36.png)

 2. Log into Ranger with the below username and password 
 ````    
 - Username: hdinsightadmin@xxxxxx.onmicrosoft.com
 - Password: <Password you set in section 1.5>
````
 
  3. Click on the Settings icon on the header to see the list of users and groups. Explore if the users and  groups selected step 2 of section 2.1 have made it to Ranger.

![Ranger2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture38.png) 


![Ranger2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture39.png)



![Ranger2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture40.png)

4. Click on the 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU0ODk4Mjg5MSwtMTA3NzQ1MDY1OCwxNT
I2OTE4OTM3LDEwOTU5MDMwMTAsLTIwODg3NDY2MTJdfQ==
-->