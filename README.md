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


We then a the tables in Platform Database using this queries ( This will be a seed for the microservice ):

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




