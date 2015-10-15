#**Module:** API - Overview#

WIP (bgosselin)

To Discuss:
- review the commands in the first module
- overview
	- MIT info
	- DN vs RN
	- API structure
		- class vs mo
		- query target
		- queries
		- filters
		- examples for each thru-out
- need to cover Tenant, Ctx, BD, Filter, contract, subject, EPG as they are discussed in the next module
- also cover info on the pod itself
	- example pulling info from the pod
	
##Learning Objectives##

After Completing this module you will be able to:

##API Review from Module 1##
In the first Module, we issues four main API commands:
- POST: Create Tenant
- POST: Create Private-Network for the Tenant
- GET: All Tenants
- GET: All Private-Networks for a specific Tenant

No two RESTful API's are identical and so weather you’re a novice scripter or an experienced programmer, it will helpful to delve into the specifics of the API calls to get a better understanding of the  for these commands.  First we’ll cover some background information on the APIC and its API and then run through some concrete examples.



####API Instruction Formate####
For this module, we’ll continue to use the Chrome plugin, Postman, to execute our API calls.  For the remainder of this module you can assume that no API header or authentication will be needed and API payload or body will only be included if necessary.  As such, we will streamline our API instructions.

**For example,** rather than displaying API Login Instruction in this formate:
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

We will display it is as:
```
POST https://<IP-of-APIC>/api/aaLogin.json
	{
		“aaaUser” : {
			“attributes” : {
				“name” : “username-for-APIC”,
				“pwd” : “password-for-APIC”
			}
		}
	}
```

In the case of no payload, the API call will simple be display as:
```
GET https://<IP-of-APIC>/<API-URL>
```