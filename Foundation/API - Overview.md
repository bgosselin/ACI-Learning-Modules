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
In the first Module, we issued three main API calls:
- POST: Create Tenant & Private-Network
- GET: All Tenants
- GET: All Private-Networks for a specific Tenant

No two RESTful API's are identical and so weather you’re a novice scripter or an experienced programmer, it will helpful to delve into the specifics of the APIC API to get a better understanding of how to structure your calls.  First we’ll cover some background information on the APIC and its API and then run through some concrete examples.
