# ai-attendance-project-app
# Intelligent AI Attendance System

An AI-powered attendance management application that automates classroom attendance using face recognition and voice recognition.

The system provides separate interfaces for teachers and students. Teachers can create and manage subjects, enroll students, capture attendance from classroom photographs, use voice-based attendance, and review previous attendance records. Students can register using facial features, optionally enroll a voice profile, join subjects, and monitor their attendance.

This repository contains the main Streamlit application for the Intelligent AI Attendance System.

---

# Project Overview

Traditional classroom attendance systems often require teachers to manually call student names, circulate attendance sheets, or depend on identity cards. These approaches can consume classroom time and may also be vulnerable to human error or proxy attendance.

The Intelligent AI Attendance System is designed to automate this process by combining computer vision, machine learning, voice processing, database management, and a web-based user interface.

The application supports two major AI-based attendance mechanisms:

1. Face Recognition Attendance
2. Voice Recognition Attendance

The application is developed using Streamlit and stores application data through Supabase.

---

# Problem Statement

Manual attendance systems have several limitations:

* They consume valuable classroom time.
* Attendance records must often be maintained manually.
* Large classrooms make individual attendance verification difficult.
* Manual systems may introduce human errors.
* Proxy attendance can be difficult to detect.
* Reviewing historical attendance records can become inefficient.

The objective of this project is to create an intelligent attendance system capable of identifying students through biometric features and automatically recording attendance.

---

# Objectives

The primary objectives of the project are:

* Automate classroom attendance using artificial intelligence.
* Detect and recognize students from classroom photographs.
* Provide an alternative voice-based attendance mechanism.
* Maintain centralized attendance records.
* Provide separate teacher and student interfaces.
* Simplify subject creation and student enrollment.
* Reduce manual attendance processing.
* Store biometric feature embeddings instead of relying only on raw images.
* Provide teachers with structured attendance summaries.
* Allow students to monitor their enrolled subjects and attendance.

---

# Key Features

## Teacher Authentication

Teachers can create an account and securely log in using a username and password.

Teacher passwords are hashed before being stored in the database.

---

## Student Face Registration

Students can register directly through the application using a camera.

During registration:

1. The student's face is detected.
2. Facial landmarks are extracted.
3. A numerical face embedding is generated.
4. The embedding is stored in the database.
5. The recognition model is refreshed.

---

## Face-Based Student Login

Registered students can log in using Face ID.

The application:

1. Captures an image using the camera.
2. Detects the face.
3. Extracts the face embedding.
4. Compares it with registered student embeddings.
5. Identifies the corresponding student.
6. Opens the student's dashboard.

---

## AI Classroom Attendance

Teachers can upload multiple classroom photographs.

The application processes each image independently and determines which registered students appear in the photographs.

Detected students are marked present, while enrolled students who are not detected are marked absent.

---

## Multi-Image Attendance

Teachers can upload multiple images of the classroom during a single attendance session.

This helps reduce missed detections when students may not be clearly visible in one photograph.

The system combines detections across all uploaded photographs before generating the attendance result.

---

## Voice-Based Attendance

Students can optionally register their voice profile.

Teachers can then record classroom audio in which students speak phrases such as:

```text
I am present.
```

The application extracts speaker embeddings from the audio and compares them against stored student voice embeddings.

Recognized students can then be marked present.

---

## Subject Management

Teachers can:

* Create subjects
* Define subject codes
* Define sections
* View enrolled students
* View the number of conducted attendance sessions

---

## Subject Enrollment

Students can enroll in subjects using subject codes.

Teachers can also share subject information through a generated joining link and QR code.

---

## QR-Code Based Joining

The application generates QR codes using the `segno` library.

Students can scan the code to access the subject enrollment workflow.

---

## Attendance Records

Teachers can review attendance records containing information such as:

* Date and time
* Subject
* Subject code
* Number of students present
* Total enrolled students

---

## Student Attendance Dashboard

Students can view:

* Enrolled subjects
* Total attendance sessions
* Number of sessions attended
* Subject information

Students can also unenroll from subjects.

---

# System Workflow

The overall application workflow can be represented as:

```text
                         Intelligent AI Attendance System
                                      |
                   -----------------------------------------
                   |                                       |
                Teacher                                 Student
                   |                                       |
            Login / Register                         Face ID Login
                   |                                       |
          Teacher Dashboard                       Student Dashboard
                   |                                       |
        -------------------------                ---------------------
        |           |           |                |                   |
     Attendance   Subjects    Records         Enrollment          Records
        |
   -------------------
   |                 |
Face Attendance   Voice Attendance
   |                 |
Face Detection    Audio Recording
   |                 |
Face Embedding    Voice Embedding
   |                 |
Student Matching  Speaker Matching
   |                 |
   -------- Attendance Decision --------
                        |
                 Supabase Database
```

---

# Face Recognition Pipeline

The face recognition module is implemented primarily using:

* dlib
* face_recognition_models
* NumPy
* Scikit-learn

## Step 1: Face Detection

The application uses dlib's frontal face detector:

```python
dlib.get_frontal_face_detector()
```

The detector identifies facial regions within an input image.

---

## Step 2: Facial Landmark Detection

A pre-trained facial landmark predictor is loaded using the `face_recognition_models` package.

The detected landmarks help align the face before calculating its representation.

---

## Step 3: Face Embedding Generation

The dlib face recognition model generates a:

```text
128-dimensional facial embedding
```

for each detected face.

This numerical vector represents important facial characteristics.

---

## Step 4: Store Student Embeddings

During student registration, the generated face embedding is converted into a list and stored in the Supabase database.

Example representation:

```text
Student
   |
   +-- Student ID
   +-- Name
   +-- Face Embedding
   +-- Voice Embedding
```

---

## Step 5: Train Face Classifier

Registered face embeddings are used to train a Support Vector Machine classifier.

The project currently uses:

```python
SVC(
    kernel="linear",
    probability=True,
    class_weight="balanced"
)
```

The classifier maps facial embeddings to student IDs.

---

## Step 6: Similarity Verification

After the classifier predicts a student ID, the detected embedding is compared with the stored embedding using Euclidean distance.

A resemblance threshold is applied before accepting the prediction.

Current threshold:

```text
0.6
```

If the distance is below or equal to the threshold, the student is treated as a valid match.

---

# Voice Recognition Pipeline

The voice recognition module uses:

* Resemblyzer
* Librosa
* NumPy

---

## Voice Enrollment

During registration, students may optionally record a short voice sample.

The audio is:

1. Loaded using Librosa.
2. Resampled to 16 kHz.
3. Preprocessed.
4. Passed through the Resemblyzer VoiceEncoder.
5. Converted into a numerical speaker embedding.
6. Stored in Supabase.

---

## Voice Attendance

During voice attendance:

1. The teacher records classroom audio.
2. The audio is divided into speech segments.
3. Very short segments are ignored.
4. A speaker embedding is generated for each segment.
5. Each embedding is compared against enrolled students.
6. Cosine-like similarity is calculated using a dot product.
7. The best matching student is selected.
8. The student is marked present if the score passes the similarity threshold.

The current matching threshold is:

```text
0.65
```

---

# Teacher Workflow

The teacher interface provides three major modules.

## 1. Take Attendance

The teacher:

```text
Logs In
   |
Selects Subject
   |
Uploads Classroom Photos
   |
Runs Face Analysis
   |
AI Detects Students
   |
Attendance Results Generated
   |
Teacher Reviews Results
   |
Attendance Stored
```

Teachers can alternatively choose the voice attendance option.

---

## 2. Manage Subjects

Teachers can:

* Create new subjects
* Assign subject codes
* Define sections
* View student counts
* View class counts
* Generate subject joining links
* Generate QR codes for subject enrollment

---

## 3. Attendance Records

Teachers can review historical attendance sessions.

The application groups attendance logs according to:

* Timestamp
* Subject
* Subject code

It then calculates:

```text
Present Students / Total Students
```

for each attendance session.

---

# Student Workflow

The student workflow begins with Face ID authentication.

```text
Open Student Portal
       |
Capture Face
       |
Face Detection
       |
Face Recognition
       |
   Recognized?
     /     \
   Yes      No
    |        |
 Dashboard  Registration
             |
        Enter Name
             |
      Save Face Embedding
             |
     Optional Voice Profile
             |
       Create Account
```

After login, students can:

* View enrolled subjects
* Join new subjects
* View total classes
* View classes attended
* Unenroll from subjects

---

# Technology Stack

| Category               | Technology              |
| ---------------------- | ----------------------- |
| Programming Language   | Python                  |
| Web Application        | Streamlit               |
| Computer Vision        | dlib                    |
| Face Recognition Model | face_recognition_models |
| Machine Learning       | Scikit-learn            |
| Face Classifier        | Support Vector Machine  |
| Numerical Computing    | NumPy                   |
| Data Processing        | Pandas                  |
| Voice Recognition      | Resemblyzer             |
| Audio Processing       | Librosa                 |
| Database               | Supabase                |
| Password Security      | bcrypt                  |
| QR Generation          | Segno                   |
| Image Processing       | Pillow                  |
| Version Control        | Git                     |
| Repository Hosting     | GitHub                  |

---

# Machine Learning Components

The project contains two separate biometric AI pipelines.

## Face Recognition

Techniques used:

* Face detection
* Facial landmark detection
* Feature extraction
* 128-dimensional facial embeddings
* Support Vector Machine classification
* Euclidean distance-based identity validation

---

## Speaker Recognition

Techniques used:

* Audio preprocessing
* Speech segmentation
* Speaker embeddings
* Vector similarity comparison
* Threshold-based speaker identification

---

# Database

Supabase is used as the application's backend database.

The code interacts with the following main tables.

## teachers

Stores teacher information.

Typical fields include:

```text
teacher_id
username
password
name
```

---

## students

Stores student profiles.

Typical fields include:

```text
student_id
name
face_embedding
voice_embedding
```

---

## subjects

Stores course information.

Typical fields include:

```text
subject_id
subject_code
name
section
teacher_id
```

---

## subject_students

Maintains the many-to-many relationship between students and subjects.

Typical fields include:

```text
student_id
subject_id
```

---

## attendance_logs

Stores attendance information.

Typical fields include:

```text
student_id
subject_id
timestamp
is_present
```

---

# Project Structure

```text
ai-attendance-project-app/
|
|-- app.py
|-- requirements.txt
|-- README.md
|
`-- src/
    |
    |-- components/
    |   |-- dialog_add_photo.py
    |   |-- dialog_attendance_results.py
    |   |-- dialog_auto_enroll.py
    |   |-- dialog_create_subject.py
    |   |-- dialog_enroll.py
    |   |-- dialog_share_subject.py
    |   |-- dialog_voice_attendance.py
    |   |-- footer.py
    |   |-- header.py
    |   `-- subject_card.py
    |
    |-- database/
    |   |-- config.py
    |   `-- db.py
    |
    |-- pipelines/
    |   |-- face_pipeline.py
    |   `-- voice_pipeline.py
    |
    |-- screens/
    |   |-- home_screen.py
    |   |-- student_screen.py
    |   `-- teacher_screen.py
    |
    `-- ui/
        `-- base_layout.py
```

---

# Module Description

## `app.py`

Main application entry point.

Responsibilities include:

* Streamlit application configuration
* Teacher/student navigation
* Session-state handling
* Subject joining through URL parameters

---

## `src/screens/`

Contains the major application interfaces.

### `home_screen.py`

Provides the main landing page and role selection.

### `teacher_screen.py`

Handles:

* Teacher login
* Teacher registration
* Teacher dashboard
* Attendance processing
* Subject management
* Attendance history

### `student_screen.py`

Handles:

* Face authentication
* Student registration
* Face enrollment
* Optional voice enrollment
* Student dashboard
* Subject enrollment
* Attendance statistics

---

## `src/pipelines/face_pipeline.py`

Responsible for:

* Loading dlib models
* Detecting faces
* Generating face embeddings
* Training the SVM classifier
* Identifying students
* Applying face-distance verification

---

## `src/pipelines/voice_pipeline.py`

Responsible for:

* Loading the speaker encoder
* Processing recorded audio
* Creating voice embeddings
* Segmenting classroom audio
* Comparing speaker embeddings
* Identifying registered speakers

---

## `src/database/`

Contains Supabase configuration and database operations.

Functions include:

* Teacher registration
* Teacher authentication
* Student creation
* Subject creation
* Student enrollment
* Student unenrollment
* Attendance creation
* Attendance retrieval

---

## `src/components/`

Contains reusable UI components and Streamlit dialogs.

Examples include:

* Photo upload dialog
* Attendance result dialog
* Subject creation dialog
* Subject enrollment dialog
* Voice attendance dialog
* QR sharing dialog
* Dashboard header
* Subject cards

---

# Installation

## Prerequisites

Before running the project, make sure you have:

* Python installed
* Git installed
* A Supabase account/project
* Internet connection for initial dependency installation
* Webcam for student Face ID registration/login
* Microphone for voice attendance features

Python 3.10 or a compatible Python version is recommended.

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/bibekn414/ai-attendance-project-app.git
```

Move into the project directory:

```bash
cd ai-attendance-project-app
```

---

## Step 2: Create a Virtual Environment

### Windows

```bash
python -m venv venv
```

Activate it:

```bash
venv\Scripts\activate
```

### Linux/macOS

```bash
python3 -m venv venv
```

Activate it:

```bash
source venv/bin/activate
```

---

## Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

The current project dependencies include:

```text
streamlit
numpy
pandas
scikit-learn
dlib-bin
face_recognition_models
supabase
bcrypt
segno
pillow
librosa
resemblyzer
```

---

# Supabase Configuration

The application reads Supabase credentials from Streamlit secrets.

The following values are required:

```text
SUPABASE_URL
SUPABASE_KEY
```

Create the Streamlit configuration directory:

```text
.streamlit/
```

Inside it, create:

```text
secrets.toml
```

The project structure should then contain:

```text
ai-attendance-project-app/
|
|-- .streamlit/
|   `-- secrets.toml
|
|-- app.py
|-- requirements.txt
`-- src/
```

Add your Supabase credentials:

```toml
SUPABASE_URL = "your_supabase_project_url"
SUPABASE_KEY = "your_supabase_key"
```

Do not commit your real Supabase credentials to a public GitHub repository.

Add `.streamlit/secrets.toml` to `.gitignore` when storing production credentials locally.

---

# Running the Application

After installing the dependencies and configuring Supabase, run:

```bash
streamlit run app.py
```

Streamlit will normally start the application on:

```text
http://localhost:8501
```

Open the displayed address in your browser.

---

# Application Usage

## Teacher

### Register

1. Open the application.
2. Select Teacher.
3. Select Register.
4. Enter your name.
5. Choose a username.
6. Create a password.
7. Complete registration.
8. Log in using your credentials.

---

### Create a Subject

After logging in:

```text
Manage Subjects
      |
Create New Subject
      |
Enter Subject Information
      |
Subject Created
```

---

### Share Subject

The teacher can generate:

* Subject code
* Joining link
* QR code

Students can use these to enroll.

---

### Take Face Attendance

1. Open `Take Attendance`.
2. Select a subject.
3. Upload one or more classroom photographs.
4. Select `Run Face Analysis`.
5. Wait for face detection and recognition.
6. Review detected students.
7. Save attendance.

---

### Take Voice Attendance

1. Select the required subject.
2. Select `Use Voice Attendance`.
3. Record classroom audio.
4. Ask students to speak individually.
5. Analyze the audio.
6. Review recognized students.
7. Save attendance.

Students must have registered voice embeddings before voice recognition can identify them.

---

# Student Usage

## New Student Registration

1. Open the Student section.
2. Capture your face.
3. If no registered profile matches, registration becomes available.
4. Enter your name.
5. Optionally record a voice sample.
6. Create your account.

The application extracts and stores your biometric embeddings.

---

## Returning Student Login

1. Open Student login.
2. Position your face in front of the camera.
3. Capture the image.
4. The AI system compares the face with registered profiles.
5. A successful match opens the student dashboard.

---

## Join a Subject

Students can enroll using:

* Subject code
* Shared enrollment link
* QR code

---

# Attendance Processing

For face attendance, the system processes every uploaded classroom photograph.

The pipeline follows:

```text
Classroom Image
       |
Face Detection
       |
Detected Faces
       |
128-D Face Embeddings
       |
SVM Classification
       |
Distance Validation
       |
Student IDs
       |
Compare with Subject Enrollment
       |
Present / Absent
       |
Attendance Logs
       |
Supabase Database
```

If a student appears in any one of the uploaded photographs, the system can mark that student as detected for the session.

---

# Authentication and Security

## Teacher Password Security

Teacher passwords are not directly stored as plain text by the application logic.

The project uses:

```text
bcrypt
```

for password hashing.

During registration:

```text
Password
   |
bcrypt hashing
   |
Hashed Password
   |
Database
```

During login:

```text
Entered Password
       |
bcrypt verification
       |
Stored Hash
       |
Authentication Result
```

---

## Biometric Data

The application stores generated face and voice embeddings for recognition.

Biometric information should be treated as sensitive data.

A production implementation should include:

* Clear user consent
* Secure database policies
* Restricted database access
* Appropriate retention policies
* Encryption where appropriate
* Compliance with applicable privacy regulations

This repository should primarily be treated as an educational or prototype implementation unless additional production security controls are configured.

---

# Current Limitations

The current implementation has several areas that can be improved.

## Face Recognition Conditions

Recognition accuracy may be affected by:

* Poor lighting
* Large face angles
* Blurred images
* Partial face obstruction
* Low-resolution images
* Large classroom distances

---

## Voice Recognition Conditions

Voice recognition can be affected by:

* Background noise
* Multiple students speaking simultaneously
* Poor microphone quality
* Echo
* Very short speech segments

---

## Training Strategy

The current face-recognition model trains an SVM using stored student embeddings.

A larger number of images per student and more robust embedding aggregation could improve recognition reliability.

---

## Liveness Detection

The current prototype does not implement advanced anti-spoofing or liveness detection.

Production biometric systems should include mechanisms to reduce spoofing using photographs, recordings, or generated media.

---

## Deployment Configuration

The subject sharing implementation currently contains an application domain in the code.

For scalable deployment, the base application URL should preferably be stored as configuration or an environment variable.

---

# Future Improvements

Possible improvements include:

* Real-time webcam classroom attendance
* Automatic face tracking
* Face anti-spoofing
* Liveness detection
* Multiple face embeddings per student
* Deep-learning-based face classifiers
* Improved speaker diarization
* Noise suppression for classroom audio
* Automatic voice activity detection
* Attendance percentage visualization
* Student attendance alerts
* Minimum attendance warning system
* Teacher analytics dashboard
* Export attendance to CSV
* Export attendance to Excel
* Attendance reports in PDF format
* Email notifications
* Role-based access control
* Administrator dashboard
* Institution-level user management
* Cloud object storage for approved media
* Advanced Supabase Row Level Security
* REST API backend
* Mobile application
* Docker deployment
* Continuous integration and deployment
* Automated testing
* Recognition confidence visualization

---

# Possible Production Architecture

A future production version could follow:

```text
Web / Mobile Client
        |
        v
Authentication Service
        |
        v
Backend API
        |
----------------------------------
|                |               |
Face AI        Voice AI       Analytics
Service        Service        Service
|                |               |
----------------------------------
        |
        v
Database + Secure Storage
        |
        v
Reporting Dashboard
```

---

# Use Cases

The system can potentially be adapted for:

* Universities
* Colleges
* Schools
* Coaching institutes
* Training centers
* Workshops
* Corporate training sessions
* Laboratory classes
* Academic seminars

Any real-world biometric deployment should be implemented only with appropriate consent, privacy safeguards, and institutional approval.

---

# Skills Demonstrated

This project demonstrates practical implementation of concepts from:

## Artificial Intelligence

* Biometric recognition
* Feature extraction
* Similarity matching

## Machine Learning

* Support Vector Machines
* Classification
* Embedding-based learning
* Distance-based verification

## Computer Vision

* Face detection
* Facial landmark processing
* Face embeddings
* Image processing

## Audio Processing

* Audio preprocessing
* Speech segmentation
* Speaker embeddings
* Speaker recognition

## Data Engineering

* Structured database design
* Database queries
* Attendance logging
* Relationship management

## Software Development

* Modular Python programming
* Streamlit application development
* Session-state management
* Reusable UI components
* Database integration

## Security

* Password hashing using bcrypt
* Authentication workflow
* Secure secret configuration

---

# Repository

GitHub Repository:

```text
https://github.com/bibekn414/ai-attendance-project-app
```

---

# Author

Bibek Nayak

GitHub:

```text
https://github.com/bibekn414
```

---
