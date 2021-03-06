## Create an ESP enabled HDInsight cluster

### 1: Set up Azure Active Directory Domain Services(AAD-DS) 

#### [1.1 Configure Basic Settings](#11-configure-basic-settings)
#### [1.2 Configure network settings](#12-configure-network-settings-1)
#### [1.3 Configure group membership](#13-configure-group-membership-1)
#### [1.4 Enable AD Domain Services](#14-enable-ad-domain-services-1)
#### [1.5 Create HDInsight users and groups in Azure Active directory](#15-create-hdinsight-users-and-groups-in-azure-active-directory-1) 
#### [1.6 Join a Windows VM to manage the Domain (Optional)](#16-configure-secure-ldap-for-aad-ds-managed-domain) 
#### [1.7 Create and authorize a managed identity](#17-create-and-authorize-a-managed-identity-1) 
#### [1.9 Networking](#19-networking-1) 

### 2: Create HDInsight cluster with Enterprise Security Package enabled 
#### [2.1 Create an ESP enabled HDInsight cluster](#21-create-an-esp-enabled-hdinsight-cluster)  


### 1.1 Configure Basic Settings
1. Log into your Azure Subscription and create a resource group(example: HDIESPDemo)to hold the assets

![Create Azure Resource Group](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture0.png)





![Azure Resource Group](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture2.png)



2. In the search window search for “Azure AD Domain Services” and click Create.  

![Create Azure ADDS](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture3.png)




 3. In the basics blade enter the below details ( or as per your choice )
 - *DNS Domain Name*: contoso.com
 - *Subscription*: Choose the subscription that has   owner access to Azure Active directory
 - *Resource Group*: Choose the resource group you created (HDIESPDemo)
 - *Location*: Choose the Azure region where you would deploy the assets (East US 2)  
   
     
     ![ADDS details](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture4.png)

### 1.2 Configure network settings
 - Create the VNet and the Subnet in which the Azure Active Directory Domain Services will be created.  
 - *Name*: Contoso_VNet 

 - *Address Space*: 10.0.0.0/16
 - *Subnet name*: domainservices
 - *Subnet address range*: 10.0.0.0/24

  
  ![Network Settings](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture5.png)
 
### 1.3 Configure group membership
Create the Administrator group and add users to the Administrator group called ‘AAD DC Administrators’. Users in this group have administration permissions on machines which are joined to this domain.  Users available to add are the users in the AAD.

![Group Membership](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture6.png)
  
  ![Group Membership](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture7.png)
 
### 1.4 Enable AD Domain Services

Click OK to begin the creation of the Azure Active Directory Domain Services. This activity can take up to an hour to complete.
  
  ![Group Membership](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture8.png)
  
Click configure to update the DNS settings for the Virtual Network. This action updates the DNS server settings for the VNet  point to the IP address of the VMs where Azure AD domain services is available.

![DNS Settings](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture9.png)

![DNS Settings](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture10.png)   

### 1.5 Create HDInsight users and groups in Azure Active directory
Log into the Azure Active directory and create the following users and groups. These  groups and assigned members groups to depict how this would potentially work at customer organizations and will help us understand Ranger policies later.


| Group Name             | Members Name            | User ID                                |
|------------------------|-------------------------|----------------------------------------|
| HDInsightAdministrator | HDInsight Administrator | hdinsightadmin@xxxxxx.onmicrosoft.com  |
| HiveControlledAccess   | Hive Restricted User    | hiverestricted@xxxxxx.onmicrosoft.com  |
| KafkaControlledAccess  | Kafka Restricted User   | kafkarestricted@xxxxxx.onmicrosoft.com |
| SparkControlledAccess  | Spark Restricted User   | sparkrestricted@xxxxxx.onmicrosoft.com |
| HbaseControlledAccess  | Hbase Restricted User   | hbaserestricted@xxxxxx.onmicrosoft.com |
    
For bulk user creation you could use the example code below with Azure CLI.   

    
    for i in {1..n}; 
    do 
    az ad user create --display-name "ESP OCP Lab User"$i --password Esppass$i@ESP --user-principal-name espuser$i@xxxxx.onmicrosoft.com;
    az role assignment create --role "Contributor" --assignee espuser$i@hdiesp.onmicrosoft.com --resource-group <resource_group_name>;
    done
     
HDInsightAdministrator group would have Admin access to all cluster resources while the other groups will have controlled access to the cluster resources which would be managed by Ranger. User’s assigned to these groups will inherit the access provided.   


  ![Users](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture34.png) 
      
   ![Groups](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture35.png)
  
Log into the portal http://myapps.microsoft.com with the userid and password of an AD user. After 20 minutes or so after the password change one can login to computers which are domain joined to this domain using this username and password.

![Changepassword](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture11_1.png)
  
  ![Changepassword](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture12.png)   
      
      
![Changepassword](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture13.png)    
  
  ![Changepassword](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture14.png)


### 1.6 Join a Windows VM to manage the Domain (Optional)

Creating Domain Joined machines is not a necessity to create ESP enabled HDInsight clusters and hence is not demonstrated here. You can get further information on the process of joining a virtual machine to an Azure Active Directory Domain Services(AAD DS) managed domain [here](https://docs.microsoft.com/en-us/azure/active-directory-domain-services/active-directory-ds-admin-guide-join-windows-vm-portal).
 
### 1.6 Configure secure LDAP for AAD-DS managed domain



One would need to obtain a certificate for secure LDAP Access to the managed domain and one can get this through the below means

 - Obtain a certificate from a Public CA or enterprise CA
 - Create a self-signed certificate

Enterprise customers would use their enterprise Certificate Authority to generate these certificates but for the purposes of this demonstration we would create a self-signed certificate.

1. On a windows machine, launch Powershell and paste the below code . Note that we have used the domain name used earlier in three separate places. This certificate is valid for 365 days after the day of creation. Post creation the certificate would need to exported and signed, steps we will accomplish next. 


   
````

$lifetime=Get-Date

New-SelfSignedCertificate -Subject contoso.com `

-NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `

-Type SSLServerAuthentication -DnsName *.contoso.com, contoso.com
   
````
  
  ![Createcertificate](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture15.png)
   
   2. To export the self-signed certificate a series of steps needs to be performed which are  documented at [here](https://docs.microsoft.com/en-us/azure/active-directory-domain-services/active-directory-ds-admin-guide-configure-secure-ldap-export-pfx). At the end of the exercise save the password protected certificate file(*.pfx) to your local desktop or C drive.
   
   3. Login to the domain Contoso.com that you created earlier , click on *Secure LDAP* blade and then click *Enable* on the right. 
   
 
   ![SecureLDAP1](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture16.png)   
  
  4.  To enable secure LDAP Access over internet. Point to the LDAP certificate saved in the earlier step and enter the password used to decrypt the certificate. Click *Enable* on both *Secure LDAP* and *Allow secure access over the internet*. Once done click *Save*. This process takes 10 - 15 minutes to complete.  

![SecureLDAP2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture17.png)
     
![SecureLDAP2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture18.png)
      
  ![SecureLDAP2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture19.png)      
    
 
### 1.7 Create and authorize a managed identity
1. Go to the central search bar and search for managed identities. For more information on managed identities click [here](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).   

![ManagedIdentity1](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture20.png)
  
 2. Populated the fields and click *Create* to create a *User Assigned Managed Identity* MGI1  as shown below
    
    ![ManagedIdentity2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture21.png)
  
  3. Assign an owner to the HDInsight Managed Services identity and subsequently assign the *HDInsight Domain Services Contributor* role to the newly created managed services identity MGI1. 
  
  ![ManagedIdentity2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture22.png)
  
  ![ManagedIdentity2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture23.png)



### 1.9 Networking 
  
After the AD-DS service is created a DNS server is created on the AD VM. the AD-DS virtual network must be changed to point to these DNS servers. 

![Networking1](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture24.png)
  
 ![Networking2](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture25.png) 
  

![Networking3](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture26.png)   

If the HDInsight Cluster is created in a different VNet, these virtual networks would need to be peered.  Optionally you could choose to create the HDInsight Cluster in a  different subnet in the same VNet which this lab follows.

Create a new subnet in which the HDInsight cluster will be deployed in the subsequent steps.  
  - *Name*: Contoso_VNet
 - *Subnet name*: HDInsight
 - *Subnet address range*: 10.0.0.0/24
    
    ![Networking4](https://github.com/arnabganguly/HDInsightESPLab/blob/master/images/Picture27.png)



 

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjEzNjA0NDgwNywtMjAzNzMyODU1MywxMD
Q5OTk1ODY3LDIwMjMyMzI1NSwxNzUwODc2Njk4LDE3NTgwOTgw
MTEsMTY0MzE5NzEyNSw2ODk4OTI4NDAsLTI2OTQ3MjU5NiwtMT
AzMDA0MTE2OSwtNDkyNDM5OTkxLC0xOTIzODIyNjIzLDExMzU4
MDU4MTUsMzUxNTIyMDM5LC0xMzgwNzk1NDkwLDEzNTkxNjk4ND
gsMTMwMDU4OTE4MiwxOTg2NDI2NDE2LDExMzY2ODI3MTYsLTE3
MDcwMTg5Nl19
-->