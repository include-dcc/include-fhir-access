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


