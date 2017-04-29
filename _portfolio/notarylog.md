---
layout: post
title: NotaryLog
thumbnail-path: "img/blocitoff.png"
short-description: NotaryLog is a tool for notaries to simplify invoicing, accounting, and tracking of their jobs as independent contractors.

---

{:.center}
![]({{ site.baseurl }}/img/blocitoff.png)

## Summary
###### A Notary's Function
A notary's role is to authenticate the execution of legal documents (docs) by witnessing the signature process and verifying identities of the signatories (signers). In a perfect world, the client will present docs bearing accurate "form" language under the signature lines which a notary amends with info identifying the signers and notary, before marking the form with a state-issued stamp that uniquely identifies the notary and the validity of their commission.

> Authentication is prescribed on a state-by-state basis as is the fee a notary may charge for each signature ($10 per signature in CA). In light of this fee limitation, the bulk of their income comes from related services like travel-fees (most jobs are remote) or by specializing in complicated transactions like loan packages.

###### Attachments
There are two types of forms -- an Acknowledgment and Jurat. Forms are usually printed directly on the doc to be notarized but are often incorrect. Form language changes every few years and varies from state-to-state. Any inconsistency in language invalidates the legal power of the underlying notarized document so notaries must carry accurate forms called "attachments" since they're attached to the notarized document.

>Aside from valid statutory language, attachments require that a signer's name must exactly match their name as represented on the document. This means that "Dan" and "Daniel" are not equivalents. Same goes for middle names, "John F. Kennedy" is not the same as "John Fitzgerald Kennedy". The point is, mistakes are easy to make and increase with number of times a notary has to repeat entry of redundant information.

###### Two Types of Notary Jobs
Some jobs involve a simple doc with a handful of signatures from one signer who is a one-off client. Here, the signer is client and the total due is based on the sum of all services rendered. Other jobs come from repeat company-clients (e.g., banks), where the signing involves a loan package consisting of hundreds of pages of "legaleze", and require dozens of signatures from multiple parties. The total due is based on an agreed upon flat-fee.

###### Application Objective
This app was primarily developed as a tool for notaries to simplify invoicing, accounting, and tracking of their jobs as independent contractors. A secondary task of NotaryLog is to generate accurate and up-to-date "form-attachments" auto-populated using data in which a notary only need enter once. I performed the role of lead developer, with the help of mentor, Adam Louis.
## Explanation

General public clients are often one-off customers who don't ask for an invoice, so the utility of NotaryLog mostly pertains to repeat company-clients with "package" jobs. The job sequence in these cases is as follows: a company-agent contacts a notary for a job; the notary accepts and contacts the parties to the transaction, and arranges a time and place for executing the package. Once completed, the notary invoices the company.

Notaries are only paid after the underlying transaction is successfully executed. If a deal falls through or an error arrises in the docs, the job goes unpaid. But if everything goes right, payment is rendered according to a company's billing cycle which occurs weeks after a job. To complicate matters, payments are aggregated for all jobs fulfilled during a billing cycle and are often itemized in ways that make it difficult to match to set of jobs.

For general public jobs, even if the client doesn't require an invoice, a notary will want to track these signings for accounting purposes pertaining to taxes, expenses, and revenue tracking. Also, regardless of invoicing, a notary wants the power to generate accurate and valid form-attachments. Given the precision of language for form-attachments, the volume of jobs, and latency in payment, NotaryLog is a powerful tool for any notary.

## Problem

NotaryLog's invoicing behavior has to account for the various billing scenarios. This behavior differs on _who_ gets billed and _how_ they're charged. The client charged always turns on where the job is sourced; either the company or the signer for jobs from the general public. All flat-fee jobs come from company clients, but not all companies are billed a flat-fee. All general public jobs are billed piecemeal according to the sum of line-item services rendered.

After generating an invoice, a user should be able to track payment status along with broader accounting information like: total jobs performed, total billed and received, as well as where revenue is sourced. Aside from job data, NotaryLog should also store the contact information for both companies, and the company-agents who act as a point person for any given job.

Finally, for cases when a notary needs to generate a "form", NotaryLog should auto-populate up-to-date statutory language along with the other information a notary enters to complete the attached form. This job-specific language includes: the date of notarization and name of underlying document; the signer's names exactly as they appear on the corresponding document; and the notary's name, commission number and expiration date.

## Solution

#### Shared Resources
###### Contact-Info
We quickly noticed that the structure of a user's contact-info overlapped with that of a signer, company, and agent. We also noted that not all of the entities referencing contact-info would require the same attributes; a notary has phone numbers, emails, and a billing address, but a agent doesn't need an address. So we divided contact-info into smaller polymorphic resources (`Location`, `Phone`, & `Email`) to be shared as needed.

As we continued to map a notary's workflow to the user-flow, we noticed the difference between a signer and company-agent was only contextual; both entities share the same attributes but they reference different parent objects, `Job` and `Company` respectively. As such, both of these entities could be represented by a single `People` model.  

###### Document Names
We identified two areas of the app where a notary references  a `Document` by name. First, during the process of generating form-attachments; a notary references the name of the specific document in order to link it with the attachment. Second, when a notary generates a piecemeal invoice, each notarized document is listed as a service rendered. As such we defined the `Document` resource as polymorphic.

#### Building Jobs & Invoices
###### Two Types of Jobs & Three Types of Line Items
The `Job` resources is a nexus for associating a notary with job-specific data like a job's date, location and signers. An invoice includes this data plus a list of services rendered, the total due, and a bill attributed to a client. Given the overlap between a job and invoice, we had to decide if they were separate resources or just different views. We also had to develop a billing solution capable of handling both flat-fee and piece-meal invoicing.

We developed our solution according to a notary's view of the problem; as a matter of profession, notaries see two types of jobs rather than two billing scenarios. For invoicing, both types of jobs list services rendered, but the services are not identical and the total is calculated differently. We used Single Table Inheritance to govern this behavior by defining two types of jobs and three types of line-items based on the type of service rendered.

Validations require a user to declare a job's `:type`; either `PackageJob` for flat-fee cases or `PieceMealJob` for piecemeal billing. Depending on a job's type, two HTML forms are conditionally displayed on the view where invoices are generated. One form is unique to a `PackageJob` and another is unique to `PiecemealJob`, while the third appears an invoice for both jobs. The line-item's `:'type` is set using hidden fields on their respective forms.

Aside from `:type`, a line-item consists of a `:name` attribute which describes the service, a `:fee` representing the cost of a single unit of a service, a `:quantity`, and `:amount` calculated from the number of units of the service multiplied by its cost. The names of these categorical services are not entirely unique so a notary may either select its name from a dropdown or create a new categorically specific name if one doesn't already exist.

###### Package-Jobs: Package-Line-Items & Service-Line-Items
The first categorical line-item service is unique to `PackageJob` and is represented by `PackageLineItems`. A class method automatically sets `:quantity` to "1", while `:fee` is manually set. The `:amount` equals the value of `:fee`. The second type of line-item is represented by `ServiceLineItems` and describes things like "travel" or "printing". Again, `:quantity` is set to "1" but `:fee` is set to "0" since it doesn't factor into the total due.

###### Piecemeal-Job: Document-Line-Items & Service-Line-Items
The third line-item is unique to `PiecemealJob` and represented by `DocumentLineItems`. A user manually sets `:quantity`, while `:fee` is automatically set to whatever the statutorily allowed amount to charge for a notarized signature. The amount is calculated by multiplying the number of signatures by the fee. The second line-item,`ServiceLineItems`, is shared between job-types but `:amount` is manually entered by the user.

###### Attributing an Invoice to a Client
Both companies and signers are candidates for billing so we declared this relationship polymorphic (`:billable`). Flat-fee jobs are always billed to companies, so once this job-type is initialized, `:billable_type` is set to "company". Jobs invoiced piecemeal can be billed to a company or person, so `:billable_type` is set manually for this jobs-type.

Once `billable_type` is set, a user manually sets `billable_id` to associate an invoice to a client. From the invoice view, the user selects from a conditional value list populated by either all company names or the names of signers for a specific job. In either case, the dropdown displays the `:name` attribute but sets the `billable_id`.

## Results & Conclusion
I had a general idea of the problems NotaryLog would attempt to solve, but I grounded our development approach in the professional realities of the job by conducting ongoing interviews with a notary. After the initial conversations I was left with an overly ambitious vision of NotaryLog's scope. This was motivating at first but quickly stymied the process.

I was moving in several directions at once before my mentor encouraged me to focus on a single feature with a minimal amount of functionality before incrementally scaling up. That was perhaps the first lesson building this app. The next lesson involved bridging the gap between a developer's perspective and a notary-user's perspective of their workflow.

This disconnect tested my ability to represent elements of a notary's job sequence as interrelated resources and app features. For instance, she viewed an invoice as a single "thing" rather than a collection of associated entities. At the same time, she viewed an invoice and job as two separate "things" rather than a single entity.

It was also tricky because our notary had been in the business for over a decade and so she had internalized all the minutia of the job. We had to gradually unpack these details before mapping them to app features. This wasn't just difficult for myself as a developer, the notary was also put in a position where she had to stretch and scrutinize her workflow.

I didn't get anywhere near finishing all the features I initially envisioned, but I learned some valuable lessons. For one, build solutions only after extensively researching the dimensions of the problem. False starts are motivation killers. Next, build the smallest valuable feature before scaling up. Being overly ambitious leads to false starts. Finally, don't be paralyzed by the idea that there is a single best way to do something. Every time I revisit this app's code, I see alternative approaches to a given problem. 
