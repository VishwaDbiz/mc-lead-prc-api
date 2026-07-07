### Business Requirement: Lead Process API
**Requirement ID:** BR-001  
**Endpoint:** POST https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/${dropbox.path}

**Business Objective**  
The Lead Process API is designed to automate the lead generation process by integrating Dropbox with Salesforce. When a new or updated file is detected in a specified Dropbox folder, the API will extract lead information from the file and upsert it into Salesforce, ensuring that the sales team has the most current lead data available.

**API Contract**  
| Field         | Type   | Required | Description                          |
|---------------|--------|----------|--------------------------------------|
| No request schema available. |        |          |                                      |

**Response Schema**  
| HTTP Status | Field | Type   | Description                          |
|-------------|-------|--------|--------------------------------------|
| No response schema available. |       |        |                                      |

**Flow Implementation**  
1. The system shall execute the dropbox:on-new-or-updated-file-in-folder-listener when a file is created or updated in the specified Dropbox folder.
2. The system shall log a message at INFO level indicating that a file has been detected.
3. The system shall execute the dropbox:files-download to download the file contents from Dropbox.
4. The system shall transform the downloaded payload using DataWeave to extract lead information.
5. The system shall execute the salesforce:upsert to insert or update the lead information in Salesforce.
6. The system shall log a message at INFO level indicating that the upsert operation has been completed.

**Field Mapping / Data Transformation**  
| Source Field       | Target Field              | Transformation Logic                     |
|--------------------|--------------------------|------------------------------------------|
| payload            | csvData                  | Read the payload as application/csv      |
| row.FirstName      | FirstName                | Direct mapping                           |
| row.LastName       | LastName                 | Direct mapping                           |
| row.Email          | Email                    | Direct mapping                           |
| row.Email          | Lead_Email_Id__c        | Direct mapping                           |
| row.Company        | Company                  | Direct mapping                           |
| "Dropbox"          | LeadSource               | Static value assignment                   |

**Connected Systems**  
| System     | Protocol            | URL / Destination         | Direction      | Purpose                          |
|------------|---------------------|---------------------------|----------------|----------------------------------|
| Salesforce | salesforce:upsert   | Salesforce_Config         | Bidirectional   | To upsert lead data into Salesforce |

---

### Business Requirement: Lead Process API
**Requirement ID:** BR-001  
**Endpoint:** GET https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/leads(leadId)

**Business Objective**  
The Lead Process API is designed to retrieve lead information based on a unique lead identifier. This functionality is essential for enabling sales and marketing teams to access lead data efficiently, thereby improving customer engagement and conversion rates.

**API Contract**

| Field       | Type   | Required | Description                          |
|-------------|--------|----------|--------------------------------------|
| No request schema available. |        |          |                                      |

**Response Schema**

| HTTP Status | Field       | Type   | Description                          |
|-------------|-------------|--------|--------------------------------------|
| 200         | payload     | object | The lead information retrieved.      |
| 400         | error       | string | Error message for bad requests.     |
| 404         | error       | string | Error message when lead not found.  |
| 500         | error       | string | Error message for server errors.    |

**Flow Implementation**
1. The system shall log a message at INFO level.
2. The system shall transform the payload using DataWeave, utilizing the following configurations:
   - `p('prc.url')` is set to "https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/lead".
   - `p('prc.get.method')` is set to "GET".
   - `p('prc.contentType')` is set to "application/json".
   - `p('prc.client_secret')` is set to "bb1da774897f4271842F4379B7Eb44E2".
   - `p('prc.client_id')` is set to "99d58299e759411aac621988a467c7b9".
3. The system shall invoke the sub-flow "commonRequestSub_Flow".
4. The system shall log a message at INFO level within the sub-flow.
5. The system shall call an external REST API using the method and URL defined in the variables.
6. The system shall log a message at INFO level after the API call.
7. The system shall end the execution of the "commonRequestSub_Flow".
8. The system shall transform the payload using DataWeave.
9. The system shall log a message at INFO level.

**Field Mapping / Data Transformation**

| Source Field                     | Target Field         | Transformation Logic                                      |
|----------------------------------|----------------------|----------------------------------------------------------|
| p('prc.url')                    | URL                  | Concatenation with `attributes.uriParams.'leadId'`     |
| attributes.uriParams.'leadId'   | URL                  | Concatenation with `p('prc.url')`                        |
| p('prc.get.method')             | Method               | Direct assignment                                         |
| p('prc.contentType')            | Content-Type         | Direct assignment                                         |
| p('prc.client_secret')          | client_secret        | Direct assignment                                         |
| p('prc.client_id')              | client_id            | Direct assignment                                         |
| payload                          | payload              | Direct assignment                                         |

**Connected Systems**

| System               | Protocol      | URL / Destination                                                                 | Direction | Purpose                          |
|---------------------|---------------|-----------------------------------------------------------------------------------|-----------|----------------------------------|
| External REST API   | http:request  | https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/lead              | Outbound  | To retrieve lead information.    |

---

### Business Requirement: Lead Process API
**Requirement ID:** BR-001  
**Endpoint:** POST https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/leads  

**Business Objective**  
The Lead Process API is designed to facilitate the creation of new leads within the CRM system. By integrating with Salesforce and other external systems, this API streamlines the lead management process, ensuring that all relevant information is captured and processed efficiently. This enhances the sales team's ability to follow up on leads and ultimately drive revenue growth.

**API Contract**

| Field      | Type   | Required | Description                       |
|------------|--------|----------|-----------------------------------|
| firstName  | string | Yes      | The first name of the lead.      |
| lastName   | string | Yes      | The last name of the lead.       |
| email      | string | Yes      | The email address of the lead.   |
| phone      | string | Yes      | The phone number of the lead.    |
| company    | string | Yes      | The company name of the lead.    |

**Response Schema**

| HTTP Status | Field | Type   | Description                       |
|-------------|-------|--------|-----------------------------------|
| 201         |       |        | Lead created successfully.        |

**Flow Implementation**
1. The system shall log a message at INFO level.
2. The system shall transform the payload using DataWeave, utilizing configuration parameters for URL, content type, client secret, client ID, and create method.
3. The system shall query Salesforce for existing leads.
4. The system shall transform the payload using DataWeave to filter based on the email of the lead.
5. The system shall route based on conditions, executing the appropriate branch.
6. If the condition is met, the system shall invoke the sub-flow "commonRequestSub_Flow".
7. The system shall log a message at INFO level within the sub-flow.
8. The system shall call an external REST API using the method and URL defined in variables.
9. The system shall log a message at INFO level after the API call.
10. The system shall end the execution of the sub-flow "commonRequestSub_Flow".
11. If the condition is not met, the system shall publish a message to a queue/topic.
12. The system shall raise an error with the code CUSTOM:RECORDEXISTS.
13. The system shall transform the payload using DataWeave for error handling.
14. The system shall log a message at INFO level.
15. The system shall handle errors raised within the flow.
16. On error, the system shall propagate the failure to the caller.

**Field Mapping / Data Transformation**

| Source Field                     | Target Field         | Transformation Logic                                      |
|----------------------------------|----------------------|----------------------------------------------------------|
| p('prc.url')                    | URL                   | Direct mapping to the URL for the external API call.    |
| p('prc.contentType')            | Content-Type          | Direct mapping to set the content type for the request.  |
| p('prc.client_secret')          | client_secret         | Direct mapping for authentication.                        |
| p('prc.client_id')              | client_id             | Direct mapping for authentication.                        |
| p('prc.create.method')          | Method                | Direct mapping to specify the HTTP method.               |
| payload                          | payload               | Direct mapping of the incoming payload.                  |
| payload                          | Filtered Payload      | Filter based on email matching vars.requestPayload[0].email. |
| vars.resp                        | Response Payload      | Direct mapping of the response variable.                 |

**Connected Systems**

| System                | Protocol            | URL / Destination                                                                 | Direction     | Purpose                                      |
|-----------------------|---------------------|----------------------------------------------------------------------------------|---------------|----------------------------------------------|
| Salesforce            | salesforce:query    | Salesforce_Config1                                                               | Bidirectional  | To query existing leads in Salesforce.      |
| External REST API     | http:request        | https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/lead            | Outbound      | To create a new lead in the external system.|
| Anypoint MQ           | anypoint-mq:publish | Anypoint_MQ_Config                                                              | Outbound      | To publish messages to a queue/topic.       |

---

### Business Requirement: Lead Process API
**Requirement ID:** BR-001  
**Endpoint:** POST https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/Subscriber  

**Business Objective**  
The Lead Process API is designed to facilitate the processing of lead data by consuming messages from a queue. It interacts with an external REST API to create leads based on the incoming messages, ensuring that lead information is processed efficiently and accurately.

**API Contract**  
| Field            | Type   | Required | Description                                      |
|------------------|--------|----------|--------------------------------------------------|
| No request schema available | -      | -        | No request schema is defined for this API.      |

**Response Schema**  
| HTTP Status | Field | Type | Description |
|-------------|-------|------|-------------|
| No response schema available | -     | -    | No response schema is defined for this API. |

**Flow Implementation**  
1. The system shall be triggered by Anypoint MQ messages from the queue "${mq.queue.name}".
2. The system shall log a message at INFO level.
3. The system shall transform the payload using DataWeave, utilizing the following configurations: 
   - `p('prc.url')` = "https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/lead"
   - `p('prc.contentType')` = "application/json"
   - `p('prc.client_secret')` = "bb1da774897f4271842F4379B7Eb44E2"
   - `p('prc.client_id')` = "99d58299e759411aac621988a467c7b9"
   - `p('prc.create.method')` = "POST"
4. The system shall invoke the sub-flow "commonRequestSub_Flow".
5. The system shall log a message at INFO level within the sub-flow.
6. The system shall call the external REST API using the method and URL defined in the variables.
7. The system shall log a message at INFO level after the API call.
8. The system shall execute an acknowledgment (Ack) for the message in the Anypoint MQ.
9. The system shall log a message at INFO level after acknowledging the message.
10. The system shall handle any errors raised within the flow.
11. On error, the system shall propagate the failure to the caller.

**Field Mapping / Data Transformation**  
| Source Field                | Target Field       | Transformation Logic                          |
|-----------------------------|--------------------|-----------------------------------------------|
| p('prc.url')                | URL                 | Direct mapping to the URL for the external API call. |
| p('prc.contentType')        | Content-Type        | Direct mapping to the Content-Type header.   |
| p('prc.client_secret')      | client_secret       | Direct mapping to the client_secret field.    |
| p('prc.client_id')          | client_id           | Direct mapping to the client_id field.        |
| p('prc.create.method')      | Method              | Direct mapping to the HTTP method for the API call. |
| payload                      | Payload             | Direct mapping of the incoming payload.       |
| attributes.ackToken         | ackToken            | Direct mapping of the acknowledgment token.   |

**Connected Systems**  
| System               | Protocol      | URL / Destination                                                                 | Direction | Purpose                          |
|----------------------|---------------|----------------------------------------------------------------------------------|-----------|----------------------------------|
| External REST API    | http:request  | https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/lead            | Outbound  | To create leads based on incoming messages. |