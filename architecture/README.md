# OpenRMF Architecture
This has the current architecture information for the OpenRMF application as of version 0.11 and beyond (current version).

![Image](./openRMF-Tool-Architecture.png?raw=true)

## The Genesis
The January 2019 Phase 1 Vision / Concept as drawn on my whiteboard in my basement:
![Image](./phase1-architecture-whiteboard.jpg?raw=true)

## Current Architecture

The architecture was setup to do a few things for this tool and for myself actually:
* https://github.com/Cingulara/openrmf-web is the web UI pointing to all these APIs below to render checklists listings, data, vulnerabilities, reports, and allowing saving 
of chart data and XLSX downloads.
* https://github.com/Cingulara/openrmf-api-read is for listing, getting, and downloading a checklist and its metadata of title, description, type, and future user info. It also has an export to Excel function that is color coded for status thanks to a request by a good IA/CS friend of mine that needed that.
* https://github.com/Cingulara/openrmf-api-save is for saving checklist data by posting it ALL in a form, including the raw checklist data (not a file). This publishes an "openrmf.save.xxxx" type of event to NATS.
* https://github.com/Cingulara/openrmf-api-template is for uploading, listing, and getting checklist file templates to start from.
* https://github.com/Cingulara/openrmf-msg-score is a NATS messaging subscriber listening to "openrmf.save.*" events from Save and Upload to score the checklist and putting that score into the Mongo DB for the scoring API
* https://github.com/Cingulara/openrmf-api-scoring is for reading a score of a checklist as well as scoring a checklist based on a file posted (at runtime).
* https://github.com/Cingulara/openrmf-api-upload is for uploading a .CKL checklist file with metadata and saving the result. This publishes an "openrmf.save.xxxx" type of event.
* https://github.com/Cingulara/openrmf-api-controls is a read-only lookup of NIST controls to match to CCI for the compliance API and other pieces that need to pull the NIST control descriptions for 800-53.
* https://github.com/Cingulara/openrmf-api-compliance is for generating the compliance listing, matching NIST controls via CCI to 1 or more checklists in a System. This generates a table of controls and the checklists corresponding to the control from the system's group of checklists. The checklist is linked to the Checklist service and color coded by status.
* https://github.com/Cingulara/openrmf-api-audit is a read-only lookup of Audit information for OpenRMF that only Administrators can access.
* https://github.com/Cingulara/openrmf-msg-controls is a NATS client for responding to request/reply on a list of all RMF controls or get the information on a specific control (i.e. AC-1).
* https://github.com/Cingulara/openrmf-msg-compliance is a NATS client for responding to request/reply on a list of all compliance listings mapping STIG vulnerability IDs to controls. Use this for a full listing based on a low/moderate/high level as well as if you are using personally identifiable information (PII) or similar data.
* https://github.com/Cingulara/openrmf-msg-template is a NATS client for responding to request/reply on a request for a System template based on the title passed in.
* https://github.com/Cingulara/openrmf-msg-checklist is a NATS client for responding to request/reply on a request for a checklist based on the Mongo DB record Id passed in.
* https://github.com/Cingulara/openrmf-msg-system is a NATS client for responding to published messages for updating a System based on title, number of checklists, or running a compliance check.
* https://github.com/Cingulara/openrmf-msg-audit is a NATS client for responding to published messages for recording auditable events through OpenRMF.

I started this project with separate microservices all over including messaging for API-to-API communication. Future enhancements are to organically add publish / subscribe pieces such as compliance, auditing, logging, etc. to make this more user and enterprise ready. Along with all the error trapping, checking for NATS connection, etc. that a production 1.0 application would have. 

## Current Messaging Architecture

OpenRMF uses NATS messaging to work eventual consistency as well as API-to-API communication. The items below talk on the types of messaging, who initiates the communication, the receiving NATS client, and a description of what it does.

| Subject | Msg Type | Calling API |     Receiving Client  | Description |
|---------|----------|-------------|-----------------------|-------------|
| openrmf.checklist.read | Request/Reply | Score (Msg Client), Compliance  | openrmf-msg-checklist | Ask for a full checklist/artifact record based on the ID passed in |
| openrmf.system.checklists.read | Request/Reply | Compliance          | openrmf-msg-checklist | Ask for all checklist records for a given system title passed in |
| openrmf.checklist.save.new | Subscribe | Upload | openrmf-msg-score | Grab the new uploaded checklist ID sent and generate the score of open, not applicable, not a finding, and not reviewed items across categories |
| openrmf.checklist.save.update | Subscribe | Upload | openrmf-msg-score | Grab the updated checklist ID sent and generate the score of open, not applicable, not a finding, and not reviewed items across categories |
| openrmf.checklist.delete | Subscribe | Save | openrmf-msg-score | Delete the score record for the passed in checklist ID  |
| openrmf.score.read | Subscribe | Read | openrmf-msg-score | Read API calling for the score when generating an XLSX checklist download listing the score. |
| openrmf.compliance.cci | Request/Reply | Compliance | openrmf-msg-compliance | Send back all CCI to NIST Major Controls listing. |
| openrmf.compliance.cci.control | Request/Reply | Compliance, Read | openrmf-msg-compliance | Send back a full listing of CCI items based on the NIST/RMF control passed in.  |
| openrmf.controls | Request/Reply | Compliance |  openrmf-msg-controls| Send back the list of all controls. |
| openrmf.controls.search | Request/Reply | Controls | openrmf-msg-controls | Send back a single record for the passed in control (i.e. AC-2). |
| openrmf.template.read | Request/Reply | Upload | openrmf-msg-template | Send back a single template checklist record for the passed in title. Used when you upload an XCCDF SCAP scan result to create a checklist. |
| openrmf.checklist.read | Request/Reply | Score | openrmf-msg-checklist | Send back a single checklist record for the passed in Mongo DB InternalId title. Used when you score a checklist in eventual consistency to pull the checklist and create the structure so we can do a count on status. |
| openrmf.system.checklists.read | Request/Reply | Read | openrmf-msg-checklist | Send back the list of checklists so we can export them into XLSX from the System page. |
| openrmf.system.update.{Id} | Subscribe | Save | openrmf-msg-system | When a system title is updated, make sure all references throughout the checklists are updated. We save the system group Id and the title with the checklists for easier usage throughout OpenRMF. The source-of-truth is the systemgroups collection in MongoDB. |
| openrmf.system.count.> | Subscribe | Upload (add) and Save (delete) | openrmf-msg-system | Increments with a ".add" at the end of the subject or decrements if there is a ".delete" at the end of the subject. The payload is the system group Id. |
| openrmf.system.compliance | Subscribe | Compliance | openrmf-msg-system | Stores the date of the last compliance check run into the system group record for display later. |

| openrmf.compliance.cci.references | Request/Reply | Compliance | openrmf-msg-compliance | Passing in the CCI it returns the CCI title and NIST list of references for the CCI passed in to the Compliance API. |
