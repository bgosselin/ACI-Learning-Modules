#Communicating with the APIC#

##Learning Objectives##

After Completing this unit you will be able to:
- Understand how logical and physical devices
- Identify the main interfaces to the APIC Controller


##ACI Interfaces##

Application Centric Infrastructure (ACI) provides a centralized approach to network fabric operation.  All components of the fabric – spine and leaf switches – as well as network services and certain VM management features can be provisioned and managed solely from the Application Policy Infrastructure Controller (APIC).  While operators can connect to individual fabric nodes, one of the key features of ACI is to provide a single access point for status, provisioning, configuration and troubleshooting to greatly simplify the network engineer’s ability to manage the network. 

There are three primary ways to access the APIC
- Command Line Interface (CLI)
- Application Program Interface (API)
- Graphic User Interface (GUI)

Note that while there are three ways to access the APIC, there is actually only two interfaces, the CLI and API.  The GUI is actually a separate Web-based application hosted on the APIC which leverage the API to interface with the internal data model housed in the controller. We’ll discuss both the CLI and API this module but focus on the API going forward.
 
![APIC-Interfaces](\Pix\APIC-Interfaces.png)


##Command Line Interface##

Network Engineers have a great deal of experience in using a CLI to interact with network equipment.  While the commands and features of ACI differ from traditional switch operating systems, such as IOS or NX-OS, the method of interaction is the same.

To highlight this interface, we will log into the APIC and create a new VRF instance in ACI. In ACI, VRF’s are known as ‘Private Networks’ and they contained within a Tenant. Tenants in ACI define a logically separate space for both networks and operation.  Each tenant contains its own networks on a separate Virtual Routing Forwarding instance. At this time we’ll just create an empty VRF inside a tenant to show case the use of the CLI.

- Open an SSH client, such as SecureCRT or Putty, and establish an SSH session with one of the IP addresses in the APIC cluster. 
- Provide the Username and password for your APIC

Once logged in, you will see the command prompt for the APIC:
 
![CLI-Login](\Pix\CLI-Login.png)

First, we’ll create a new tenant called ‘ExampleCliTenant’.  To do so we’ll navigate to the tenants directory on the APIC. Once in the folder we’ll create this tenant as a new managed object and commit this configuration. We’ll discuss managed objects in more detail in the next module, for now we’ll just focus on the interfaces.
 
![CLI-CreateTenant](\Pix\CLI-CreateTenant.png)

Now that the tenant had been created in the APIC, we will change directories in the APIC to the private-network folder where will create a new private network (or VRF). Just as with the tenant, we will create it as a managed object and commit this configuration to the controller.
 
![CLI-CreateVRF](\Pix\CLI-CreatVRF.png)

We can confirm the tenant has been created with the following:

```
	show tenant
```

Issuing this command in the SSH client, we see that we are actually just using the Linux command to view the contents of the Tenants directory in the APIC
    
![CLI-ShowTenant](\Pix\CLI-ShowTenant.png)

So, we can also very easily confirm that our ‘myVRF’ private-network was created with the following:

‘’’
	ls /aci/tenants/ExampleCliTenant/networking/private-networks/
‘’’


##Application Program Interface##

Just as with the CLI, the API exposes full access to the APIC for configuration, management, operation, troubleshooting and more.  The APIC API is REST-based which offers several advantages over the CLI when creating scripts and programs to interact with ACI.  For the purposes of this module, we’ll assume basic knowledge of RESTful API’s. However, a detailed discussion of REST API’s is available in the Coding 101 DevNet Lab, [here]( https://learninglabs.cisco.com/lab/aci/step/1). 

We can showcase APIC interaction with the API by creating another tenant and VRF in the fabric.  Just as there are several tools which can be used to access the Command Line Interface of a device, we have a variety of options for API access as well.  We can access the API directly from a script or program. However, for testing and learning a new API, it is often helpful to use a REST client to explicitly see the API call and response. For this purpose we’ll use Postman, a Google Chrome browser plugin. For Instructions on how to install and use Postman are available in the Coding 101 DevNet Lab, [here]( https://learninglabs.cisco.com/lab/aci/step/1). These instructions demonstrate Postman using the APIC-EM API, not ACI. Since REST is not a protocol but a framework, no two RESTful API’s are identical.  Nevertheless, there are many similarities between these two API’s and minor differences will be addressed as we as we use Postman, below.

Note, before using Postman with the APIC, it may be necessary to accept the APIC’s self-signed SSL certificate in your Chrome browser. Do so by opening your chrome browser and navigate here:

```
	https://<IP-of-APIC>
```

Click on ‘Proceed to <IP-of-APIC> (unsafe)’
 
![Postman-AcceptCert](\Pix\Postman-AcceptCert.png)

We’ll get into the specifics of the ACI API in the next module, for now let’s focus the specifics of creating and confirming a new Tenant and VRF.  Open the Postman browser add-on and execute the following by clicking the send button:

- **URL:** 
```
	https://<IP-of-APIC>/api/aaLogin.json
```
- **Method:** POST
- **Header:** None
- **Authentication:** None
- **Body:**
```Python
	{
		“aaaUser” : {
			“attributes” : {
				“name” : “username-for-APIC”,
				“pwd” : “password-for-APIC”
			}
		}
	}
```
The REST call in Postman should match the following:
 
![Postman-Login](\Pix\Postman-Login.png)

You can confirm the REST call was successful with the response Code.  In the picture below, the response code is the well-known value, 200 – indicating the REST call was successfully received and a response returned. 

When a login message is accepted, the API returns a data structure that includes a session timeout period in seconds and a token that represents the session. The token is also returned as a cookie in the HTTP response header. 
 
![Postman-LoginResponse](\Pix\Postman-LoginResponse.png)

To maintain your session, you must send login refresh messages to the API if no other messages are sent for a period longer than the session timeout period. To send a refresh message, execute the following in Postman:

- **URL:** 
```
	https://<IP-of-APIC>/api/aaaRefresh.json
```
- **Method:** GET
- **Header:** None
- **Authentication:** None
- **Body:** None

The response should include a status 200.

Now that we have logged into the APIC, we’ll create a new Tenant named ‘ExampleApiTenant’ by executing the following call in Postman:

- **URL:** 
```
	https://<IP-of-APIC>/api/node/mo/uni.json
```
- **Method:** POST
- **Header:** None
- **Authentication:** None
- **Body:** 
```Python
	{
		"fvTenant" : {
			"attributes: {
				"name" : "ExampleApiTenant"
			}
		}
	}
```

Check for a status 200 response from this previous command. If the REST call was successful, we can add a private-network instance, named ‘myVRF’ by executing the following command:

- **URL:** 
```
	https://<IP-of-APIC>/api/node/mo/uni/tn-ExampleApiTenant.json
```
- **Method:** POST
- **Header:** None
- **Authentication:** None
- **Body:** 
```Python
	{
		"fvCtx" : {
			"attributes: {
				"name" : "myVRF"
			}
		}
	}
```

Similar to the CLI implementation, lets confirm that the Tenant and private-network were created.  First we can confirm the tenant by executing the following:
- **URL:** 
```
	https://<IP-of-APIC> /api/node/class/fvTenant.json?
```
- **Method:** GET
- **Header:** None
- **Authentication:** None
- **Body:** None

This API call should return a JSON model for each of the tenants in the APIC.  You should be able to note one of the results which contain a Distinguished Name (DN) with the value:

```
	uni/tn-<name-of-tenant>
```

In this case it will be ‘uni/tn-ExampleApiTenant’.

![API-TenantDistinguishedName](\Pix\API-TenantDistinguishedName.png)

To confirm the private-network was created, execute the following:

- **URL:** 
```
	https://<IP-of-APIC> /api/node/mo/uni/tn-ExampleApiTenant/.json?query target=subtree&target-subtree-class=fvCtx
```
- **Method:** GET
- **Header:** None
- **Authentication:** None
- **Body:** None

Again, we should see a JSON model in which the Distinguished Name containts:

```
 	uni/tn-<name-of-tenant>/ctx-<name-of-private-network>
```

![API-VRFDistinguishedName](\Pix\API-VRFDistinguishedName.png)

We have now successfully created a new Tenant and private-network using the APIC API.  In the next module explore these API calls in detail.

##Graphic User Interface##

The last interface for the APIC is the Graphic User Interface (GUI).  As mentioned earlier, the GUI actually leverages the API.  Since this lab is focused on API interaction with the APIC, we won’t go through all of the steps to create a new tenant and private-network in the GUI.  For more information on GUI configurations please see the ‘Cisco APIC Getting Started Guide, available [here]( http://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/getting-started/b_APIC_Getting_Started_Guide.pdf). 
