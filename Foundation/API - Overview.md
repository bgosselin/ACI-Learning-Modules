#**Module:** API - Overview#

	
##Learning Objectives##

After Completing this module you will be able to:
- Make API calls with JSON or XML payload 
- Understand the API Object Model and its tree structure
- Differentiate ACI objects by Name or by Class


##API Review from Module 1##
In the first Module, we issued three main API calls:
- POST: Create Tenant & Private-Network
- GET: All Tenants
- GET: All Private-Networks for a specific Tenant

No two RESTful API's are identical and so weather you’re a novice scripter or an experienced programmer, it will helpful to delve into the specifics of the APIC API to get a better understanding of how to structure your calls.  First we’ll cover some background information on the APIC and its API and then run through some concrete examples.

##Payload Format##
As was discussed in the last module, ACI supports two formats in the Body of its REST calls:
- JSON
- XML

For the purposes of creating a script or a simple program you should consider these options as having equal merit but a significantly different approach for representing the data we are reading and writing to the APIC controller.  

To compare these options recall the JSON payload we used to create a new tenant and private network:
```
{
	"polUni": {
      	"children":[{
          	"fvTenant" : {
              	"attributes" : {
                  	"name" : "ExampleApiTenant"
              	}
              	"children": [{
              		"fvCtx": {
                        "attributes": {
                            "name": "myVRF"
                        }
                    }
              	}]
          	}
      	}]
	}
}
```

The XML payload for this same REST call would be:
```
<polUni>
	<fvTenant name="ExampleApiTenant">
		<fvCtx name="myVRF"/>
	</fvTenant>
</polUni>
```
In order to execute the REST call with this XML payload, we would need also change the URL to end with ‘.xml’ rather than ‘.json’.
```
https://<IP-of-APIC>/api/node/mo.xml
```

As you can see, using XML yields a similar hierarchy with the private network being contained within the tenant and so on.  However, in this case the XML format is *MUCH* easier to write and to understand.  There may be API calls in which the opposite is true – reading and writing in JSON may be preferable. You should feel free to use the option that works best for you.  (Please note that most of the only ACI programming examples leverage XML.  In an effort to provide more JSON examples, we’ll use JSON for the remainder of this module.) 

	
##The ACI Management Information Tree (MIT)##
So far we have seen a few examples of the hierarchal nature of components within ACI.  In order to understand how to build the proper API calls, we’ll spend a bit of time learning about this structure.

All the physical and logical components that comprise the ACI fabric are represented in a hierarchical management information model (MIM), also known as a Management Information Tree (MIT). Each node in the tree represents a managed object (MO) or group of objects that contains its administrative state and its operational state.

The hierarchical structure starts with the Policy Universe at the top (Root) and each node contains parent and child nodes. Each node in the tree is a MO which represents a physical object, such as a switch or adapter, or a logical object, such as a policy or fault. 

Below is upper-most view of the tree with the Policy Universe and the various types of MO’s directly connected to it. Each MO below the Policy Universe also has many child MO’s of its own, further extending the tree.

![Root Tree](https://github.com/bgosselin/ACI-Learning-Modules/blob/master/Foundation/API%20-%20Overview%20-%20Pictures/policyUniverse.png)


The entire MIT is quite extensive.  Fortunately we can begin making useful scripts with only rudimentary understanding of the tree. For example, below is an overview of the Tenant portion of the MIT. 

![Tenant Tree](https://github.com/bgosselin/ACI-Learning-Modules/blob/master/Foundation/API%20-%20Overview%20-%20Pictures/tenant.png)


When we use the API, we take this MIT structure into account.  We already examined the payload for our API call to create a new tenant and private network. In that JSON format, see the tree – the private network is a child of the tenant which is a child of the policy universe (polUni).

#**May want to discuss moving thru the tree to create a specific object**#


##Class vs Managed Object##
The concept of object oriented programming in used in the ACI tree. Each MO’s in the MIT is off a certain type (or Class).  The MO’s Class defines its charactistics – its attributeds and where it sits releative to other objects in the tree.  Again, going back to our API call to create a tenant and private network, to create our new tenant, we actually create an object of the type ‘fvTenant’ and gave it the name.  Similar, we create an object of type fvCtx when we wanted to create a new private network or VRF.  fvTenant and fvCtx are the names of the Tenant and Private-Network Classes in the MIT, respectively.  This JSON payload is reproduced below as a reference:
```
{
	"polUni": {
      	"children":[{
          	"fvTenant" : {
              	"attributes" : {
                  	"name" : "ExampleApiTenant"
              	}
              	"children": [{
              		"fvCtx": {
                        "attributes": {
                            "name": "myVRF"
                        }
                    }
              	}]
          	}
      	}]
	}
}
```


*As an interesting side note, the name fvTenant makes as a Class name since it is a virtual fabric Tenant. However fvCtx is not an intuitive name.  Originally, Private-Networks were referred to as Context in ACI, The name was changed but the original MIT Class name for the context or Ctx is still a part of the underlying codebase and API.*

We can refer to a specific MO in our API calls or we can refer to all the MO of the same type or class. When we create the Tenant and Private-Network, our API call contained MO:
```
https://<IP-of-APIC>/api/node/mo.json
```
This is to specify that we were working with specific objects.

However, when we wanted to see **all** the tenants in the APIC, we used a GET API call, referring to the whole class, fvTenant:
```
https://<IP-of-APIC>/api/node/class/fvTenant.json?
```  

####Calling a MO####
In the above example we called a specific class.  We could do the same thing with a specific MO. For example, instead of using the /class/fvTenant.json URL to return all the tenants, use the follow API call to return just the tenant we created:
```
GET https://<IP-of-APIC>/api/node/mo/uni/tn-ExampleApiTenant.json
no payload…
```

Notice that we see the same info as when we retrieved **all** the tenants in the last module but now we only see this info for our single tenant as in the output below:

![Single Tenant Return](https://github.com/bgosselin/ACI-Learning-Modules/blob/master/Foundation/API%20-%20Overview%20-%20Pictures/tenantAPIReturn.png)


##Distinguished Name & Forgiving Model##
In the previous example you may notice to two things in the API and response which seem a little odd:
1. We referred to our tenant as ‘uni/tn-tenant’ in the API URL instead of just its name.
2. In the response, see a number of attributes that we did NOT define we created the tenant.

The first oddity is due to the concept of a Distinguished Name (DN).  There can be many MO’s of the same class as we now know. For instance, there are many Tenants in the ACI fabric which are all of the fvTenant Class.  Furthermore, many MO’s can have varying types of child MO’s. For example, the Policy Universe can have Tenants as child MO’s but it can also have VM Domains, Network Services and serveral other child MO’s. In order to logically separate each MO beyond any ambiguity, each MO had a DN.  In order to know where the MO fits in the tree, the DN is the summation of all the parent elements above the MO.  But the DN also needs to identify each MO’s type or Class, so a prefix is added the MO’s name.  In the case of a tenant, the prefix ‘tn-‘ for tenant name. 

The DN is used in the URL for API call to a specific MO, the DN for our tenant is:
```
uni/tn-ExampleApiTenant
```

The Other oddity you may notice is that we only gave our Tenant a name but in the response above, there are quite a few attitubes that have been defined.  This is because, the API operates in forgiving mode, which means that missing attributes are substituted with default values (if applicable) that are maintained in the internal data management engine (DME).

We can include these default parameters when we initially create the our new tenant MO.  Or, since we have already created our tenant, we can update the attributes in the MO. Let’s a descriptions to our tenant.  We’ll say, “My first tenant :)”.  To do so execute the following API call:

```
POST https://198.18.133.200/api/node/mo.json
{
	"polUni": {
      "children":[{
          "fvTenant" : {
              "attributes" : {
                  	"name" : "ExampleApiTenant",
                    "descr" : "My first tenant :)"
              }
              
          }
      }]
	}
} 
```
Now if you re-issue the GET request with the tenants DN, you should see the updated description.

##APIC HTTP Verbs##
Notice in the previous example, that we updated the Tenant using a POST API call.  The APIC API uses the following verbs:
- POST
- GET
- DELETE

##Add a Bridge Domain##
Now lets take the information we’ve just learned and add to our net tenant and private network
**More to come…**
