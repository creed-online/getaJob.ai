# GetaJob.ai — Conceptual Product Intent

> [!WARNING]
> **Superseded by V2 Architecture & Implementation Plan**  
> Retained for original product intent and vision only. Discovery, referrals, and stale-filter sections no longer reflect the technical build. Refer to [`BUILD_GUIDE.md`](./BUILD_GUIDE.md), [`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md), and [`techStack.md`](./techStack.md) for the active technical specification.






# Multi Tenant System



* Onboarding Routes - 
  1\. Login (Mail, Password)
  2\. Register (Mail, Name, Number, create passowrd,   confrim password)
  3\. Delete user
  4\. Update user Credentials






# User Knowledge Base



* Upload Resume
* Github Repos
* Research Paper or other docs (Optional)
* Transcript (Optional)
* QA bank (Optional)
* Other information (Optional)

# Learn about the User



* Parse The user's Resume and learn about there information and store it for future operations.
* Research the User's Github Repos and other docs and also store that information for future operations
* Understand all the other given information by the user and store that aswell for future operations

# Discovery Engine



* Discovers Opportunities from various platforms such as linkedin, indeed, unstop, research fellowship, X and over the internet according to the information provided by the user and matches the job description with the user's data to create a confidence level.
* Confidence of opportunities will be calculated by the similarity of job description with user's data and stale opportunities (opportunities having more then 1000 applicants and posted 2 weeks ago are stale).

# Listings

List all the opportunities with there confidence levels

# Opportunity Validation



* When User clicks on a particular opportunity to apply the Ai must run a ATS score test on the user's resume for that job description.
* If the ATS score is less then 8 then required keywords and changes must be done in the resume.
* If ATS score is really less then all the efforts must be done to make that resume slection worthy.
* Create a Cover letter if required for that particular opportunity.

Give option to download the updated resume for that opportunity.

Give option to download the created CV if a CV was created. 

# Auto - apply with one click (on the company protal, form)



* Auto fill for all the fields for that opportunity according to the user's stored data.
* While applying if something comes which is a custom question and the Ai have no answer about it then that particular question must be left unanswared and the Ai must tell user to fill that himself.

If there are multiple pages of form for the application of the particular opportunity an error must not be created and it should be handled gracefully.

# Find referral for that opportunity



* Find atleast 5 referrals (HR's , similar department people) for that opportunity for the user to reach out to with there linkedin, contact info such as mail, mobile number.
* Craft a really good, short and simple message for every referral so the user can directly copy paste that message to the referral.

# Tracker Page 



* Tracks the amount of application user have applied to in a day, in a week, in a month and in a year.
* Must add all the applicatiions in real time.
* should have functionality to sort the table from applied today, appliead a week ago, applied a month ago and applied a year ago aswell.
* Creates a tabular from of data format for the applied applications how many were rejections, selections, no response.
* user can fill the data table Themselves and keep a track of applications they have applied for.
* An export functionality must also be there to export the file in excel or json format.

&#x20;  
