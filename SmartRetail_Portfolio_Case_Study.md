# SmartRetail – Lessons Learned and Development Process

## 1. Introduction

SmartRetail was developed as a hands-on project to demonstrate how data analytics, business application development, AI integration, and workflow automation can be combined to solve practical business problems.

The project evolved from an inventory management automation concept into a connected business application integrating AppSheet, Google Sheets, Make.com, Google Gemini AI, Gmail, and Looker Studio.

Throughout development, I gained practical experience in designing database structures, building business applications, creating automated workflows, integrating AI, validating business rules, and troubleshooting system integration issues.

This document captures the major lessons I learned, the challenges I encountered, how I addressed them, and how I intend to approach future projects.

---

## 2. What I Built

I developed SmartRetail as a demonstration inventory and sales management solution.

The system includes:

- A Google Sheets database for storing business and inventory records.
- An AppSheet application for inventory transaction entry and management.
- Make.com scenarios for processing inventory transactions and automating business operations.
- Google Gemini AI integration for inventory forecasting and analysis.
- Gmail notifications for communicating important inventory events.
- A Looker Studio dashboard for monitoring business performance.

### Main Capabilities

1. Inventory restocking.
2. Low-stock monitoring and email alerts.
3. Inventory adjustment processing.
4. Adjustment response notifications.
5. AppSheet inventory transaction processing.
6. AI-powered inventory forecasting.
7. Interactive management reporting.

The project demonstrates how separate technologies can work together as one connected business solution.

---

## 3. What Went Well

### 3.1 Building a Centralized Database

I established Google Sheets as the central data source for the application.

I created separate tables for products, sales, inventory transactions, employees, application users, AI forecasts, and other business records.

This helped me understand the importance of organizing business information before building automation workflows.

**What I learned:** A well-structured database makes application development, automation, reporting, and future integrations easier to manage.

### 3.2 Separating Business Processes into Automation Scenarios

I developed separate Make.com scenarios for different business responsibilities.

For example:

- Restocking is handled by the restock workflow.
- Low-stock monitoring is handled by the low-stock alert workflow.
- Adjustment requests are handled by the adjustment workflows.
- AppSheet inventory transactions are handled by the transaction-processing workflow.

This made the overall solution easier to understand, test, and maintain.

**What I learned:** Each automation should have a clearly defined responsibility. Combining unrelated processes into one large scenario can make troubleshooting and maintenance more difficult.

### 3.3 Integrating AppSheet with Make.com

One of the most important developments was introducing AppSheet as the user-facing inventory management application.

I configured the application to allow inventory users to submit:

- Restock transactions.
- Sales transactions.
- Inventory adjustments.
- Product returns.

I also implemented current stock visibility, quantity validation, conditional low-stock warnings, and processing status tracking.

**What I learned:** A business application needs more than a working interface. It must also enforce business rules, protect data, and communicate processing results to users.

### 3.4 Integrating Google Gemini AI

I integrated Google Gemini AI into the inventory forecasting process.

The workflow analyzes inventory and sales information to generate forecasts, risk classifications, and inventory recommendations.

**What I learned:** AI becomes more useful when it is integrated into an actual business process rather than treated as a standalone chatbot.

### 3.5 Developing a Business Intelligence Dashboard

I used Looker Studio to present sales performance, inventory information, and AI-generated forecasts.

This helped me understand how operational data can be transformed into information that supports management decisions.

**What I learned:** Data collection and automation are important, but businesses also need clear reporting and visibility into their operations.

---

## 4. Challenges I Encountered and How I Solved Them

### 4.1 Choosing the Correct Make.com Modules

During development, I sometimes selected the wrong Google Sheets module.

For example, I initially used an Add a Row module in situations where I needed to update an existing product record.

This created duplicate product records instead of modifying the original records.

**How I solved it:**

I reviewed the workflow and replaced the incorrect Add a Row modules with Update a Row modules. I also used the product search result to identify the correct row to update.

**Lesson learned:** Before selecting a module, I must understand whether the business process requires creating a new record or modifying an existing one.

### 4.2 Understanding Router Filters

I encountered situations where transactions followed the wrong route or failed to reach the expected module.

One example involved transaction types that needed to be distinguished using router filters.

Another issue occurred when a sale-processing route did not have the correct transaction-type condition, allowing an unrelated transaction to reach the sale logic.

**How I solved it:**

I reviewed the filter conditions, corrected the transaction-type rules, and tested each route independently.

I also learned to account for differences in capitalization when comparing transaction values.

**Lesson learned:** Every router route must have clear and specific conditions. I should test each route with both valid and invalid examples.

### 4.3 Preventing Incorrect Inventory Calculations

Inventory processing requires accurate stock calculations.

I had to ensure that:

- Restocking increases stock.
- Sales decrease stock.
- Returns increase stock.
- Positive adjustments increase stock.
- Negative adjustments decrease stock.

I also needed to prevent sales and negative adjustments from reducing stock below zero.

**How I solved it:**

I implemented transaction-specific validation in AppSheet and additional stock availability checks in Make.com.

**Lesson learned:** Important business rules should not depend exclusively on the user interface. Critical validations should also exist in the backend automation.

### 4.4 Handling Duplicate Transactions

I recognized that a transaction should never update inventory more than once.

A repeated processing attempt could otherwise cause stock to be added or deducted multiple times.

**How I solved it:**

I introduced a Make.com Data Store named `SmartRetail_Processed_Transactions`, using the transaction ID as the unique key.

This provides a mechanism for checking whether a transaction has already been processed.

**Lesson learned:** Automation systems need duplicate protection, especially when processing financial or inventory-related records.

### 4.5 Managing Multiple Connected Systems

The project required several technologies to communicate correctly.

A transaction entered in AppSheet needed to appear in Google Sheets, trigger Make.com, update the correct product, and return its processing result to the original transaction record.

**How I solved it:**

I tested the integration in stages, checking the data at each point rather than assuming that a successful submission meant the entire process had completed.

**Lesson learned:** When building connected applications, I must understand where data originates, where it is stored, which system processes it, and where the final result is recorded.

### 4.6 Debugging Make.com Expressions

I encountered formula and mapping issues involving numerical values, text values, stock quantities, and data returned from different modules.

**How I solved it:**

I inspected module outputs, reviewed mapped fields, checked filter conditions, and corrected expressions where necessary.

I also learned to use explicit number parsing and absolute values where the transaction logic required them.

**Lesson learned:** I should verify the actual data returned by a module before assuming that a formula or mapping is correct.

### 4.7 Testing the AppSheet User Interface

Some virtual columns and form fields did not initially appear where I expected them.

For example, a field could be visible in a main view but missing from the transaction form.

**How I solved it:**

I reviewed the form view configuration, column order, visibility settings, and field properties.

**Lesson learned:** A correctly configured database column does not automatically guarantee that the field will appear correctly in the application interface. I must test the actual user experience.

---

## 5. Important Technical Lessons

The SmartRetail project helped me understand several important principles.

### 5.1 Database Design Comes First

I should identify the required tables, columns, relationships, unique identifiers, and data types before building automation workflows.

### 5.2 Understand Data Ownership

Each system should have a clearly defined responsibility.

For SmartRetail:

- AppSheet collects user input.
- Google Sheets stores operational records.
- Make.com processes business logic and updates records.
- Gemini generates AI-based analysis.
- Looker Studio presents business information.

### 5.3 Separate Frontend and Backend Validation

AppSheet provides immediate validation to users.

Make.com provides additional processing checks before changing inventory records.

This creates multiple layers of protection.

### 5.4 Use Update Operations Carefully

I must distinguish between:

- Adding a new record.
- Updating an existing record.
- Searching for a record.
- Checking whether a record already exists.

Using the wrong operation can create duplicate or inconsistent data.

### 5.5 Design for Failure

A reliable application should account for invalid inputs, missing records, insufficient stock, duplicate transactions, and unsuccessful automation runs.

Failure handling should explain what happened rather than silently stopping.

### 5.6 Test Incrementally

I learned that testing an entire automation only after building it makes debugging harder.

I should test individual modules, filters, routes, and data updates as I build.

---

## 6. How I Will Approach Future Projects

The SmartRetail experience has helped me establish a more structured development approach.

For future business application projects, I intend to follow these stages.

### Phase 1 – Business Requirements

- Identify the business problem.
- Understand the users and their responsibilities.
- Identify the business processes that need improvement.
- Define the expected outcomes.
- Document the project scope.

### Phase 2 – Database Design

- Identify the required tables.
- Define the columns and data types.
- Establish primary keys and relationships.
- Identify required and optional fields.
- Define data validation requirements.

### Phase 3 – Application Design

- Identify the required screens and forms.
- Define user roles and permissions.
- Design the user experience.
- Determine which fields should be automatically populated.
- Define the application workflows.

### Phase 4 – Automation Design

- Identify processes that should be automated.
- Define triggers and conditions.
- Map the required data between systems.
- Design routers and filters.
- Define success and failure paths.
- Include duplicate protection where necessary.

### Phase 5 – AI Integration

- Identify where AI adds meaningful business value.
- Prepare structured input data.
- Define the expected AI output.
- Validate AI-generated information.
- Integrate the output into the appropriate business process.

### Phase 6 – Testing and Quality Assurance

- Test individual modules.
- Test valid and invalid inputs.
- Test every automation route.
- Test duplicate submissions.
- Test failure handling.
- Verify database updates.
- Test the application from the user's perspective.

### Phase 7 – Documentation and Deployment

- Document the database structure.
- Document the application.
- Document automation workflows.
- Record testing results.
- Prepare user instructions.
- Review permissions and access.
- Prepare the solution for demonstration or deployment.

---

## 7. My Development Principles Going Forward

Based on this project, I will follow these principles:

1. I will understand the business problem before choosing technologies.
2. I will design the database before building automation.
3. I will define the workflow before configuring Make.com modules.
4. I will give every automation scenario a clear responsibility.
5. I will validate important business rules in both the application and backend.
6. I will test every route and important condition independently.
7. I will design failure handling and duplicate protection from the beginning.
8. I will inspect actual module outputs instead of guessing how data is structured.
9. I will document important technical decisions as I develop.
10. I will prioritize maintainability, reliability, and user experience.

---

## 8. Future Improvements to My Development Skills

Although SmartRetail demonstrates several practical capabilities, I recognize that I still have areas to strengthen.

My future learning priorities include:

- Advanced Make.com scenario design.
- REST APIs, HTTP modules, and webhooks.
- More advanced AppSheet expressions and security configuration.
- Database design and SQL.
- Advanced Google Gemini AI integration.
- Error handling and automation monitoring.
- Power Apps and Power Automate.
- Advanced Power BI reporting.
- Application deployment, documentation, and client support.

I also intend to build additional business application projects to gain experience across different industries and business processes.

---

## 9. Conclusion

SmartRetail has been an important practical learning experience.

I moved beyond working with individual analytics tools and developed a connected business application involving a database, user interface, automation workflows, AI integration, and business intelligence reporting.

The most valuable lessons were not limited to configuring software. I learned how to think about business processes, structure data, design workflows, enforce business rules, troubleshoot integrations, and test a complete solution.

I also learned that building a reliable business application requires careful planning, clear responsibilities, validation, failure handling, and documentation.

My goal is to continue developing these capabilities so that I can design and implement practical technology solutions that help small businesses automate operations, improve visibility, and make better decisions.

---

**Project:** SmartRetail – AI-Powered Inventory & Sales Management System

**Developer:** Solomon Pinto

**GitHub:** [KOJOPINTO](https://github.com/KOJOPINTO)
