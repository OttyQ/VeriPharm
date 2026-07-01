# VeriPharm User Manual and Technical Documentation

## Table of Contents
- [Introduction](#introduction)
- [Detailed User Manual](#detailed-user-manual)
  - [For Patients](#for-patients)
  - [For Doctors](#for-doctors)
  - [For Administrators](#for-administrators)
- [Deployment Instructions](#deployment-instructions)
- [Architecture Overview](#architecture-overview)
- [API and Services Description](#api-and-services-description)

---

## Introduction
**VeriPharm** (internal project name *MedCompatibility*) is a cross-platform mobile and desktop application developed using .NET MAUI. The application is designed to check drug compatibility, maintain prescription history, and facilitate interaction between doctors and patients.

---

## Detailed User Manual
The application supports a role-based access model. After registration and authorization, the interface adapts to your assigned role.

### For Patients
- **Home**: Quick access to scanning medication barcodes, viewing current prescriptions, and medication schedules.
- **Scan**: Ability to scan a medication barcode via the device camera for a quick database search.
- **Compatibility**: Allows adding multiple medications and running a cross-analysis for side effects and negative interactions (including AI analysis).
- **Schedule & History**: View the intake schedule of prescribed medications and interaction check history.
- **Profile**: Account settings, personal data, and the ability to export reports (in PDF).

### For Doctors
- **Doctor Home**: Summary statistics on patients and active prescriptions.
- **Patients**: View assigned patients.
- **Patient Card**: View a patient's medical history, current medications, and create new Prescriptions.
- **Cross Analysis**: Advanced functionality to check the compatibility of prescribed medications with a patient's current treatment plan.

### For Administrators
- **Admin Home**: Database management panel.
- **Users**: Role assignment (Doctor, Patient, Admin) and account suspension.
- **Medicines**: Adding, editing, and deleting medications, manufacturers, and active substances.
- **Interactions**: Management of the medication compatibility directory (risks, interaction types).
- **System Logs**: View the system event log and application errors.

---

## Deployment Instructions
The application is a .NET MAUI client with a direct connection to a MySQL database.

### Environment Requirements
- **DBMS**: MySQL Server (version 8.0 or higher).
- **SDK**: .NET 8.0 SDK (or newer) with the MAUI workload installed (`dotnet workload install maui`).
- **IDE**: Visual Studio 2022 / JetBrains Rider / VS Code (with C# and MAUI extensions).

### Startup Steps

1. **Clone the repository:**
   Open the `MedCompatibility` project folder.

2. **Configure Settings:**
   Create an `appsettings.json` file in the root directory of the `MedCompatibility` project (next to `appsettings.Example.json`).
   ```json
   {
     "Database": {
       "Local": {
         "Host": "127.0.0.1",
         "Port": 3306,
         "Name": "med_compat_db",
         "User": "root",
         "Password": "your_password"
       }
     },
     "AI": {
       "ApiKey": "your-ai-api-key"
     }
   }
   ```
   *Note: The application uses the `Database` configuration to build the connection string.*

3. **Database Setup:**
   The application uses Entity Framework Core (`DrugContext`). To initialize the DB, apply migrations:
   ```bash
   dotnet ef database update
   ```

4. **Build and Run:**
   Launch the application on your desired platform:
   - **Windows**: Select `Windows Machine` in Visual Studio or run `dotnet build -t:Run -f net8.0-windows10.0.19041.0`.
   - **Android**: Connect an emulator or device and select the Android profile.

---

## Architecture Overview
VeriPharm is built using the **MVVM** (Model-View-ViewModel) pattern, ensuring a clean separation of business logic and user interface.

### Core Components
- **Views (Pages/Popups)**: XAML files responsible for the UI. Uses `UraniumUI` for Material Design and `CommunityToolkit.Maui` for popups and utilities.
- **ViewModels**: Classes containing presentation logic. They connect to Views via Data Binding.
- **Models**: Domain entities (`User`, `Medicine`, `Prescription`, `Interaction`, `SystemLog`, etc.) mapped to MySQL tables via EF Core.
- **Services (Business Logic)**: A set of Dependency Injection (DI) services encapsulating database work, external APIs (AI), and device features.
- **Data Access Layer**: `DrugContext` — an Entity Framework Core context connecting directly to the database (via `Pomelo.EntityFrameworkCore.MySql`).

### Features
- **Local Notifications**: Implemented via `Plugin.LocalNotification` (medication intake reminders).
- **Barcode Scanner**: Integrated via `ZXing.Net.Maui`.
- **Report Generation**: PDF generation with prescriptions and history via `IPdfReportService`.

---

## API and Services Description
Since the application accesses the DB directly, the role of a classic REST API is fulfilled by internal services (registered in `MauiProgram.cs`).

### Domain Services
- `IAuthService` — handles registration, authorization, and password hashing.
- `IUserService` — profile management, role switching, retrieving patient lists for doctors.
- `IMedicineService` — CRUD operations for the medication directory, active substances, and manufacturers.
- `IInteractionService` — searches for medication conflicts. Checks interaction table relationships based on active substances.
- `IPrescriptionService` — prescribing medications to patients, generating intake schedules.
- `IScanService` — processing barcode scan results.

### Integration Services
- **IAiInteractionService / IAiHealthService**
  Sends requests to external Large Language Models (LLMs) for intelligent prescription analysis, finding non-obvious side effects, and health advice. Uses the key from the `AI:ApiKey` config.
- **IPdfReportService**
  Generates PDF documents on the client-side for printing prescriptions or exporting medical history.
- **INotificationService**
  Registers scheduled push notifications in the OS scheduler (medication reminders).

### External Communication
While the application lacks a custom backend (other than MySQL), it makes outbound HTTPS requests to:
1. AI Provider (OpenAI / Anthropic / etc. depending on `AiInteractionService` implementation).
2. Cloud DBMS (if configured to connect to Cloud MySQL).
