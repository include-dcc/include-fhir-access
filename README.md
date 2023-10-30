# INCLUDE FHIR Data Access
There are currently 6 FHIR servers that contain INCLUDE data (not including the data stored in the Kids First FHIR Servers). These servers are partitioned into 3 "Legacy" servers, and 3 "New" servers. These two groups each include "Dev", "QA" and "PROD" servers. For end users that are not directly involved in the Loading, ETL or QA, only the PROD Servers of interest. For information about accessing data stored in the Kids First servers, please see the [KF API FHIR Service](https://github.com/kids-first/kf-api-fhir-service), though, this document is really written for developers working on ETL to load data into the servers. 

The two sets of INCLUDE servers, "Legacy" and "New" differ by authorization mechanism. The old servers use a cookie based authorization scheme. This works and users can pass the cookie along with their FHIR REST calls to provide the necessary credentials. However, the cookies expire after a few days and must refreshed, which can only be done through the browser. 

The new servers use OAuth2 for authorization. Eventually, one should be able to simply follow the standard end user OAth flow to access the server for queries. However, for now, the only access available is through the service account style access. Please see then instructions [here](https://github.com/kids-first/kf-api-fhir-service#-quickstart---api-users) about how to set up an account and acquire a secret that can be used for authorizing FHIR queries. 

## Legacy Servers
As mentioned above, the following servers employ a cookie based authorization that requires periodic refreshing through the browser. While this works, it can be klunky when one tries to access the server after the cookie's expiration. 

* DEV - https://include-api-fhir-service-dev.includedcc.org/ (This server is no longer accessible)
* QA - https://include-api-fhir-service-qa.includedcc.org/
* PROD - https://include-api-fhir-service.includedcc.org/

Typically, the only server those who are not involved in the ETL or QA process will only be interested in the PROD server. For details about executing queries using these servers, please find the [legacy README](https://github.com/kids-first/kf-api-fhir-service/blob/master/docs/legacy/README.md#authenticate-to-access-server-environment). 

It should be noted that QA is whitelisted and, thus, only accessible to those who have requested access from the admins. Typically, there is no reason for those not involved in the ETL process nor QA to access this server. 

## Updated Servers
As mentioned above, the new servers have been configured to use OAuth for authorization. At some point in the near future, this will mean that anyone who has accessed the INCLUDE portal and accepted the agreement posted there can use the standard OAth flow to perform queries. However, at this time, only service accounts style access is permitted. Please see then instructions [here](https://github.com/kids-first/kf-api-fhir-service#-quickstart---api-users) about how to acquire a secret that can be used for accessing the data.

* DEV - https://include-api-fhir-service-upgrade-dev.includedcc.org
* QA - https://include-api-fhir-service-upgrade-qa.includedcc.org
* PROD - https://include-api-fhir-service-upgrade.includedcc.org

Typically, the only server those who are not involved in the ETL or QA process will only be interested in the PROD server. 

# Ways to Query FHIR Servers
FHIR provides a rich, [REST api](https://hl7.org/fhir/R4B/http.html) to query the data contained within the server. The data within these INCLUDE FHIR servers conforms to the [NCPI FHIR IG](https://nih-ncpi.github.io/ncpi-fhir-ig/). If you aren't familiar with the IG, that is the best place to start when trying to understand how to build your queries. 

Because the API is a true web api, users can use any modern HTTP client to perform the queries. This means, that your favorite tool, be it R or Python or most other modern programming environments, provide some sort of library that can be used to perform the queries. Due to the authorization component, though, you will need to understand about how to build headers and append the necessary authorization pieces to your requests. 

# Helpful Tools
## NCPI Fhir Client
The [NCPI Fhir Client](https://github.com/NIH-NCPI/ncpi-fhir-client) is a python library that is designed to help streamline FHIR Queries by handling the authorization components (for a number of different authorization schemes) and traversing pagination when performing queries with a large number of results. It also exposes a simple command line tool called __fhirq__ which can be used to perform queries. 

For more information about the __fhirq__ application, please see the [README file](https://github.com/NIH-NCPI/ncpi-fhir-client#fhirq---cli-fhir-query). 

# Query Tips
In general, users interested in understanding how to pull data from the INCLUDE FHIR Servers should have a reasonably solid understanding of FHIR ResourceTypes and their relationships with one another as well as a moderate understanding of the current version of the [NCPI FHIR IG](https://nih-ncpi.github.io/ncpi-fhir-ig/). 

For the sake of simplicity, the queries below will use the v1 PRODUCTION server, which can be accessed via the browser (i.e. you can easily paste these into your browser and see the responses without requsting and configuring any authentication credentials). These can be converted to V2 servers by replacing the words, __include-api-fhir-service__, for __include-api-fhir-service-upgrade__ if you are using the 

To help get someone started on accessing the data, however, the following tips may prove helpful: 

## Restricting queries to a particular study
Currently, there are 7 INCLUDE studies in the INCLUDE FHIR Servers: ABC-DS, BRI-DSR (Khor), DSC (DS Connect), DS-Sleep, HTP, X01-deSmith and X01-Hakonarson. The Study IDs shown in the previous sentence are used to annotate all resources loaded for the particular study they are associated with inside the meta.tag property. As such, one can restrict the results of a query by providing the _tag paremeter as part of the query. 

All queries can be modified to restrict responses to only include data from one of these studies by including it as a parameter filter for the _tag property. For instance, to query for all Patients from ABC-DS, your query would look something like this: 
> Patient?_tag=ABC-DS

## For row-level data, know the patient's ID
The first step in finding data for a patient is to know the patient's resource ID. For INCLUDE Patients, this is going to be the patient's __Global ID__. 

If you are familiar with the __External ID__ associated with a patient, but not the __Global ID__, you can use a query such as the one below to find the resource and pull the __Global ID__ itself from the resulting response. 

For instance, to find the subject with the ID, 10069, as would be found in some of the ABC-DS dataset files, you could run the following query:
> https://include-api-fhir-service.includedcc.org/Patient?identifier=10069

From there, you can find the global ID as the actual "id" property of the resource itself, or down as the "official" use identifier. For this example, the __Global ID__ for this particular subject is pt-dg7zyh89kj.

Now that you know the global ID, you can pull data associated with that subject using queries such as:

Returns all Observations associated with the patient from above
> https://include-api-fhir-service.includedcc.org/Observation?subject=Patient/pt-dg7zyh89kj
Looking over the three observations here, we can see that the patient does have Down Syndrome and is the Proband for the family. 

Return all conditions associated with the Patient:
> https://include-api-fhir-service.includedcc.org/Condition?subject=Patient/pt-dg7zyh89kj
This shows 9 different conditions associated with this particular patient. 

For some other types of resources, we need to switch over to a study that has more information, such as HTP. For these examples, we'll use the following subject: 
> https://include-api-fhir-service.includedcc.org/Patient/pt-vvecsz13

To find out what types of specimen have been collected for this patient, we could use: 
> https://include-api-fhir-service.includedcc.org/Specimen?subject=Patient/pt-vvecsz13
This query provides 21 entries. 

To identify all HTP specimen from Saliva samples, you might use a query like the following: 
> https://include-api-fhir-service.includedcc.org/Specimen?_tag=HTP&type=Saliva
or Blood
> https://include-api-fhir-service.includedcc.org/Specimen?_tag=HTP&type=Peripheral-Whole-Blood

To query for all files associated with the HTP Patient from a few queries back, you might use: 
> https://include-api-fhir-service.includedcc.org/DocumentReference?subject=Patient/pt-vvecsz13

To query for all files associated with HTP that are registered as "Controlled" Access: 
> https://include-api-fhir-service.includedcc.org/DocumentReference?_tag=HTP&security-label=controlled

