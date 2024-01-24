# healthcare-records
Automatic Synchronization of relevant healthcare documents from a patient's Primary Healthcare Provider into External CRUD Platform APIs ( Carepatron for example in this case )

System Design:

![Healthcare Records Simplified System Design](https://github.com/ProteanDev/healthcare-records/assets/6328775/9768d007-20a3-46d9-8b90-4f85b761c5be)

Data Models:

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

CRUD REST Endpoints for the said Data Models:

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
- 
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

Non-visual ERD ( to save us time ) This is how the interactions will proceed based on the data.

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

Estimated Mandays for User Stories

Please note: These are just estimates and the actual time may vary depending on several factors like developer experience, complexity of integrations, and chosen technologies.

User Story, Estimated Mandays &	Notes

- Receive email notification for new patient inquiries:	1-2 Mandays	Relatively straightforward integration with email API.
- Create new patient record in Carepatron:	2-3 Mandays	Requires API integration with Carepatron and data mapping.
- Add additional patient information:	1-2 Mandays	Depends on complexity of the information and UI design.
- View summary of synced healthcare documents:	2-3 Mandays	Requires integration with patient's Primary Healthcare Provider API and data parsing.
- See detailed list of synced healthcare documents:	2-3 Mandays	Similar complexity to the previous story.
- Manually trigger document synchronization:	1-2 Mandays	Depends on implementation details and UI design.
- Get notified of synchronization errors:	1-2 Mandays	Requires error handling and notification logic.
- Update patient record and reflect in documents:	3-4 Mandays	Needs two-way data synchronization with Carepatron and potentially the Primary Healthcare Provider.
- Automatic document updates from Primary Healthcare Provider:	3-4 Mandays	Complex integration and data handling, potentially requiring background processes.
- History log/timeline of synchronization events:	2-3 Mandays	Database design and UI implementation for displaying the log.
- Filter and search healthcare documents:	3-4 Mandays	Requires advanced search functionality and data filtering logic.
- User-friendly presentation of healthcare documents:	2-3 Mandays	Focus on UI design and data visualization for clarity.
- Secure document sharing with other practitioners:	2-3 Mandays	Requires secure data access controls and user management within Carepatron.
- Fast and efficient document synchronization:	2-3 Mandays	Optimization of API calls and data transfer processes.
- Request additional documents from Primary Healthcare Provider:	2-3 Mandays	Design and implementation of a communication channel within Carepatron.

Total estimated mandays: 30-40 Mandays

Additional factors to consider:

- API availability and documentation: Well-documented and easily accessible APIs can significantly reduce development time.
- Security requirements: Implementing robust security measures might add additional complexity and time.
- Testing and bug fixing: Allocate sufficient time for thorough testing and addressing any issues that arise.



