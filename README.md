# healthcare-records
Automatic Synchronization of relevant healthcare documents from a patient's Primary Healthcare Provider into External CRUD Platform APIs ( Carepatron for example in this case )

## System Design:

![Healthcare Records Simplified System Design](https://github.com/ProteanDev/healthcare-records/assets/6328775/9768d007-20a3-46d9-8b90-4f85b761c5be)

## Process Diagram for Dietician Patient Onboarding and Document Management ( Microservice ) in Platform ( Carepatron )

![Process Flow Page 1](https://github.com/ProteanDev/healthcare-records/assets/6328775/5660b32b-48da-4500-adb3-2a8c23f3a67e)

![Process Flow Page 2](https://github.com/ProteanDev/healthcare-records/assets/6328775/db644ac7-8082-478e-953f-ac761fc16852)

Start: A prospective patient contacts the Dietician through email.

1. New Patient Inquiry:

- System automatically sends an email notification to the Dietician.
- Dietician clicks on the link in the email to open the patient record creation page.

2. Patient Record Creation:

- Dietician enters the patient's name and email address.
- System searches for existing patient records based on the information.
  - If a match exists, redirect to the existing record and display a warning.
  - If no match exists, proceed to create a new record.
- Dietician fills in additional information such as contact details, medical history, and dietary preferences.

3. Healthcare Document Sync:

- System automatically triggers a synchronization process with the patient's Primary Healthcare Provider.
- While syncing:
-- Show a loading indicator.
-- Optionally, display a progress bar.
- Upon successful sync:
-- Display a summary of the synced documents (lab results, progress notes, physical assessments).
-- Provide links to view the full documents.
- Indicate the source (Primary Healthcare Provider) of each document.

4. Accessing and Managing Documents:

- Dietician can view a detailed list of all synced documents for the patient, categorized by type.
- Each document has metadata like date, author, and source.
- Documents can be downloaded for reference or sharing.
- Dietician can manually trigger a synchronization process to refresh document information.
- System notifies the Dietician of any synchronization errors or issues with troubleshooting steps.

5. Updating Patient Information and Documents:

- Dietician can edit patient information in Carepatron.
- Edits automatically trigger an update request to the Primary Healthcare Provider.
- Upon successful update:
-- Synced documents reflect the latest changes.
-- Dietician receives a notification.
- If update fails, the Dietician receives a notification with options to retry or contact support.

6. Automatic Document Updates and Tracking:

- System automatically updates Carepatron documents when changes are made by the Primary Healthcare Provider.
- Dietician receives a notification highlighting significant changes or updates.
- A history log displays all document synchronization events with timestamps, details, and status.
- Dietician can filter and search the log by various criteria.

7. Additional Features:

- Dietician can filter and search for specific documents based on date, type, or keyword.
- Healthcare documents are presented in a user-friendly format for easy interpretation and analysis.
- Dietician can securely share relevant documents with other healthcare practitioners within Carepatron.
- The system prioritizes fast and efficient document synchronization.
- Dietician can request additional documents or information from the Primary Healthcare Provider through Carepatron.

End: Dietician has access to the patient's complete healthcare information for informed decision-making and collaborative care.

Note: This is a high-level overview, and the specific details and interactions within each step might vary depending on technical implementation and user interface design.

## Data Models:

1. User Model:

- id: (primary key) unique identifier for the user
- email: user's email address
- name: user's full name
- type: user type (e.g., Dietician, Patient)
- carepatron_id: unique identifier for the user in Carepatron (if applicable)

2. Patient Model:

- id: (primary key) unique identifier for the patient
- name: patient's full name
- email: patient's email address
- contact_details: phone number, address, etc.
- medical_history: relevant medical information
- dietary_preferences: food allergies, restrictions, etc.
- carepatron_patient_id: unique identifier for the patient in Carepatron

3. Healthcare Document Model:

- id: (primary key) unique identifier for the document
- patient_id: foreign key referencing the Patient model
- type: type of document (e.g., lab report, progress note)
- date: date the document was created
- content: actual content of the document (text, image, etc.)
- source: source of the document (e.g., Primary Healthcare Provider, Dietician)

4. Synchronization Log Model:

- id: (primary key) unique identifier for the log entry
- patient_id: foreign key referencing the Patient model
- document_id: foreign key referencing the Healthcare Document model (if applicable)
- action: type of action (e.g., manual sync, automatic update)
- timestamp: date and time of the action
- status: success, error, etc.
- details: additional information about the action (e.g., error message)

Relationships:

- One User can have many Patients.
- One Patient can have many Healthcare Documents.
- One Healthcare Document belongs to one Patient and has one source (User or Primary Healthcare Provider).
- One Synchronization Log entry is associated with one Patient and might reference a specific Healthcare Document.


We then create the tables in the Platform Database using these queries ( This will be the seed for the microservice, also the Schemas ):

```
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) UNIQUE,
  name VARCHAR(255),
  type VARCHAR(255),
  carepatron_id INT
);

CREATE TABLE patients (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255),
  email VARCHAR(255),
  contact_details TEXT,
  medical_history TEXT,
  dietary_preferences TEXT,
  carepatron_patient_id INT,
  user_id INT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE healthcare_documents (
  id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT,
  type VARCHAR(255),
  date DATE,
  content TEXT,
  source VARCHAR(255),
  FOREIGN KEY (patient_id) REFERENCES patients(id)
);

CREATE TABLE synchronization_log (
  id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT,
  document_id INT,
  action VARCHAR(255),
  timestamp DATETIME,
  status VARCHAR(255),
  details TEXT,
  FOREIGN KEY (patient_id) REFERENCES patients(id),
  FOREIGN KEY (document_id) REFERENCES healthcare_documents(id)
);
```

## CRUD REST Endpoints for the said Data Models:

Base URL: /api/v1

Authentication: Implement appropriate authentication and authorization mechanisms (e.g., JWT tokens) before accessing these endpoints.

1. User Endpoints:

- GET /users: Retrieve all users (for authorized admins)
- GET /users/{id}: Retrieve a specific user by ID
- POST /users: Create a new user (limited access based on user type)
- PUT /users/{id}: Update an existing user (limited to self or authorized admins)
- DELETE /users/{id}: Delete a user (restricted to authorized admins)

2. Patient Endpoints:

- GET /patients: Retrieve all patients for a specific Dietician (based on user authentication)
- GET /patients/{id}: Retrieve a specific patient by ID
- POST /patients: Create a new patient (restricted to Dieticians)
- PUT /patients/{id}: Update an existing patient (restricted to Dieticians who created the patient)
- DELETE /patients/{id}: Delete a patient (restricted to Dieticians who created the patient)

3. Healthcare Document Endpoints:

- GET /patients/{patient_id}/documents: Retrieve all healthcare documents for a specific patient
- GET /documents/{id}: Retrieve a specific healthcare document by ID
- POST /patients/{patient_id}/documents: Upload a new healthcare document for a patient (restricted to Dieticians or authorized healthcare providers)
- PUT /documents/{id}: Update an existing healthcare document (restricted to the source of the document)
- DELETE /documents/{id}: Delete a healthcare document (restricted to the source of the document)

4. Synchronization Log Endpoints:

- GET /patients/{patient_id}/logs: Retrieve all synchronization logs for a specific patient
- GET /logs/{id}: Retrieve a specific synchronization log entry by ID (restricted to authorized users)
- POST /patients/{patient_id}/sync: Trigger manual synchronization for a patient's healthcare documents (restricted to Dieticians)

Additional Endpoints:

- POST /patients/{patient_id}/request-documents: Request additional documents from the patient's Primary Healthcare Provider (restricted to Dieticians)
- GET /patients/{patient_id}/shared-documents: View shared healthcare documents with other practitioners (restricted to authorized practitioners)
- POST /documents/{id}/share: Share a specific healthcare document with another practitioner (restricted to the document source)

## Non-visual ERD ( to save us time ) This is how the interactions will proceed based on the data.

Entities:

- User: Represents a user of the system, including Dieticians and Patients.
- Patient: Represents a patient under the care of a Dietician.
- Healthcare Document: Represents a healthcare document associated with a patient, such as lab reports, progress notes, and physical assessments.
- Synchronization Log: Records information about synchronization events for patient healthcare documents.

Relationships:

- One User can have many Patients (as a Dietician).
- One Patient belongs to one User (Dietician).
- One Patient can have many Healthcare Documents.
- One Healthcare Document belongs to one Patient.
- One Healthcare Document has one source (User or Primary Healthcare Provider).
- One Synchronization Log entry is associated with one Patient and might reference a specific Healthcare Document.

Additional Notes:

- The ERD does not show all attributes of each entity for simplicity.

## Estimated Mandays for User Stories ( Delivery Plan )

Please note: These are just estimates and the actual time may vary depending on several factors like developer experience, complexity of integrations, and chosen technologies.

User Story, Estimated Mandays &	Notes

1. Receive email notification for new patient inquiries:	1-2 Mandays	Relatively straightforward integration with email API.
2. Create new patient record in Carepatron:	2-3 Mandays	Requires API integration with Carepatron and data mapping.
3. Add additional patient information:	1-2 Mandays	Depends on complexity of the information and UI design.
4. View summary of synced healthcare documents:	2-3 Mandays	Requires integration with patient's Primary Healthcare Provider API and data parsing.
5. See detailed list of synced healthcare documents:	2-3 Mandays	Similar complexity to the previous story.
6. Manually trigger document synchronization:	1-2 Mandays	Depends on implementation details and UI design.
7. Get notified of synchronization errors:	1-2 Mandays	Requires error handling and notification logic.
8. Update patient record and reflect in documents:	3-4 Mandays	Needs two-way data synchronization with Carepatron and potentially the Primary Healthcare Provider.
9. Automatic document updates from Primary Healthcare Provider:	3-4 Mandays	Complex integration and data handling, potentially requiring background processes.
10. History log/timeline of synchronization events:	2-3 Mandays	Database design and UI implementation for displaying the log.
11. Filter and search healthcare documents:	3-4 Mandays	Requires advanced search functionality and data filtering logic.
12. User-friendly presentation of healthcare documents:	2-3 Mandays	Focus on UI design and data visualization for clarity.
13. Secure document sharing with other practitioners:	2-3 Mandays	Requires secure data access controls and user management within Carepatron.
14. Fast and efficient document synchronization:	2-3 Mandays	Optimization of API calls and data transfer processes.
15. Request additional documents from Primary Healthcare Provider:	2-3 Mandays	Design and implementation of a communication channel within Carepatron.

Total estimated mandays: 30-40 Mandays

Week 1:

- Setup Carepatron API and data models.
- Implement patient record CRUD operations.

Week 2:

- Implement background job processing for document synchronization.
- Design and implement Document Integration Service.

Week 3:

- Integrate with a mock PHP system for testing.
- Implement document storage and retrieval.

Week 4:

- Conduct end-to-end testing.
- Optimize for performance and scalability.

Additional factors to consider:

- API availability and documentation: Well-documented and easily accessible APIs can significantly reduce development time.
- Security requirements: Implementing robust security measures might add additional complexity and time.
- Testing and bug fixing: Allocate sufficient time for thorough testing and addressing any issues that arise.

## UI/UX Components:

General Components:

- Navigation: Header or sidebar for accessing different sections of the application (e.g., Patients, Documents, Logs).
- Search: Field for searching patients, documents, or logs (depending on user permissions).
- Notifications: Area to display system messages, errors, or success notifications.

User-Specific Components:

- User Profile: Page for viewing and editing user information (name, email, contact details, preferences).
- User Management (Admins): List of users with options to view, edit, or delete, along with role management.

Patient-Specific Components:

- Patient List: Table or list of patients with key information (name, email, primary healthcare provider).
- Patient Detail: Page for viewing and editing patient information (contact details, medical history, dietary preferences).
- New Patient Form: Form for creating new patients with required fields (name, email, etc.).
- Document List: List of healthcare documents associated with a patient, with options to view, download, or share.
- Document Upload: Form for uploading new healthcare documents for a patient.
- Synchronization Log: List of synchronization events for a patient's documents, including timestamps, status, and details.
- Manual Sync Button: Button to trigger manual synchronization for a patient's documents.
- Request Documents Button: Button to request additional documents from the patient's primary healthcare provider.

Document-Specific Components:

- Document Viewer: Component for displaying healthcare documents in a user-friendly format (e.g., PDF viewer).
- Document Edit: Form for editing existing documents (restricted to the source of the document).
- Document Share: Form for sharing documents with other practitioners (restricted to the document source).

Additional Components:

- Error Handling: Display informative error messages when issues arise.
- Loading Indicators: Show progress indicators while data is being fetched.
- Pagination: Paginate long lists of patients or documents.
- Filtering and Sorting: Allow users to filter and sort data based on different criteria.
- Accessibility Features: Ensure the UI is accessible to users with disabilities.
- Responsive Design: Adapt the UI to different screen sizes and devices.

Considerations:

- Design for User Roles: Tailor the UI components and access rights based on user roles (Dietician, Patient, Admin).
- Data Visualization: Consider using charts or graphs to visualize patient data or document trends.
- User Experience: Prioritize intuitive navigation, clear labels, and helpful feedback mechanisms.
- Security: Implement proper authentication and authorization controls to protect sensitive data.

## Test Plan for the Microcservice.

Here's a roadmap for developing a comprehensive test plan for this system:

1. Define Testing Scope and Objectives:

- Identify features and functionalities to be tested: Prioritize based on user needs, criticality, and risk.
- Outline testing goals: Aim for comprehensive coverage of features, user experience, performance, security, and data integrity.
- Specify testing types: Include unit tests, integration tests, functional tests, UI/UX tests, performance tests, security tests, etc.

2. Design Test Cases:

- Develop detailed test cases for each feature or functionality: Define expected inputs, outputs, and pass/fail criteria.
- Cover positive and negative scenarios: Include valid, invalid, and edge-case inputs.
- Consider different user roles and permissions: Test access control and data visibility for Dieticians and Patients.
- Utilize test case management tools: Organize and document test cases effectively.

3. Prepare Test Environment:

- Set up a dedicated testing environment: Replicate the production environment as closely as possible.
- Prepare test data: Use realistic data sets that reflect various patient profiles and healthcare documents.
- Ensure access to necessary tools and resources: Testing frameworks, automation scripts, and documentation.

4. Execute Test Cases:

- Run manual and automated tests: Follow the test plan and document execution steps and results.
- Log defects and bugs: Capture details, severity, and reproduction steps for each issue.
- Track test progress and coverage: Monitor overall testing completion and identify any gaps.

5. Analyze Results and Retest:

- Evaluate test execution outcomes: Analyze pass/fail rates and identify areas for improvement.
- Investigate and fix defects: Prioritize critical bugs and retest affected areas.
- Perform regression testing: Ensure bug fixes haven't introduced new issues.

6. Document and Report:

- Create a comprehensive test report: Summarize testing activities, results, defects identified, and resolutions implemented.
- Include recommendations for future testing and improvements: Suggest ongoing monitoring and test automation strategies.

Additional Considerations:

- Security testing: Conduct penetration testing and vulnerability assessments to ensure data privacy and system security.
- Performance testing: Evaluate system responsiveness, load handling, and scalability under different user loads.
- Accessibility testing: Ensure the platform is accessible for users with disabilities.
- User acceptance testing (UAT): Involve Dieticians and Patients in testing to validate usability and overall satisfaction.




This Design Document is specifically made for Carepatron's Lead Developer Exercise, but can also be used for a different platform.
If you wish for me to execute these, for your company's healthcare platform, you're gonna have to give me a team to work with.
