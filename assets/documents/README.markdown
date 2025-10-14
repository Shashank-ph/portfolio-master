**Word Count: 1295 (excluding references)**

# Secure Music Copyright Management Application

## Overview
The Secure Music Copyright Management Application is a command-line Python tool designed to address challenges in managing music copyright artefacts, such as lyrics and audio files, as outlined in the Secure Music Copyright Management Application – Design Proposal. It provides a secure platform for artists and copyright owners to upload, store, and manage digital content, ensuring data integrity, confidentiality, and traceability. The application supports two predefined roles: Admin and User, with distinct permissions. Admins can create, read, update, and delete artefacts, while Users can create, read, and update but not delete. Security is paramount, with AES-256 encryption, SHA-256 checksums, timestamping, and role-based access control, aligning with ISO 27001 and NIST Cybersecurity Framework standards (Alshar’e, 2023). 
Additional safeguards include account lockout after three failed login attempts, strong password requirements (8+ characters with uppercase, lowercase, numbers, and special characters), and restrictions on file uploads (`.txt` and `.mp3`, max 20MB). A successful login displays "Account logged in successfully." The application utilises text files (`users.txt`, `artefacts.txt`, `logs.txt`) for storage and a command-line interface (CLI) for interaction, deviating from the proposal’s relational database and RESTful API design for the sake of prototype simplicity.

## Installation
To set up the application, follow these detailed steps:
1. **Install Python**: Ensure Python 3.8 or higher is installed. Verify with:
   ```bash
   python --version
   ```

2. **Install Dependencies**: The application requires the `cryptography` library for AES-256 encryption. Install it using pip:
   ```bash
   pip install cryptography
   ```
   - **Source**: `cryptography` is hosted on PyPI (Python Package Index) (Kehrer, 2025). 
   - **Troubleshooting**: If pip fails, ensure it’s updated (`pip install --upgrade pip`) or use a virtual environment.

3. **Prepare Files**: Place the provided Python files (`main.py`, `users.py`, `artefacts.py`, `logs.py`) in a single directory. Ensure the directory has write permissions, as the application creates `users.txt`, `artefacts.txt`, `logs.txt`, and `encryption_key.key` during execution.

4. **Security  Tools**: For code quality checks, install`pylint`, and `bandit`:
   ```bash
   pip install pylint bandit
   ```

## Running the Application
**To run the application:**
1. Navigate to the directory containing the Python files:
   ```bash
   cd /path/to/directory
   ```
2. **Execute the main script:**
   ```bash
   python main.py
   ```
3. **First Run Setup**: The application checks for `users.txt`. If absent, it prompts for initial Admin and User credentials:
   - Enter an email address for each role.
   - Provide a strong password (8+ characters, including uppercase, lowercase, numbers, and special characters like `!@#$%^&*`).
   - Example for this code: 

For Admin, enter `admin@gmail.com` and the password `Admin@123`
For User, enter `user@gmail.com` and the password `User@123`

4. **Command-Line Menu**: After setup, interact via the menu:
   - **Login**: Authenticate as Admin or User.
   - **Create Artefact**: Upload a `.txt` (e.g., lyrics) or `.mp3` (e.g., audio) file, with a maximum size of 20MB, encrypted with AES-256.
   - **Read Artefact**: Decrypt and view an artefact’s content, with an option to save to a file.
   - **Update Artefact**: Replace an artefact’s content with a new file.
   - **Delete Artefact**: Remove an artefact (Admin only).
   - **List Artefacts**: Display all artefacts with ID, Created Date and Modified Date with timestamps.
   - **Exit**: Close the application.

5. **Troubleshooting**:
   - **File Not Found**: Ensure uploaded files exist and paths are correct (e.g., `./song.mp3`).
   - **Permission Errors**: Verify directory write permissions.
   - **Encryption Key Issues**: If `encryption_key.key` is corrupted, delete it and restart to generate a new key (note: existing artefacts may become inaccessible).

## System Architecture
The application follows a modular, object-oriented architecture:
- **MusicApp** (`main.py`): The central controller, orchestrating user interactions via CLI and integrating other components using the composition pattern.
- **UserManager** (`users.py`): Manages user authentication, roles, and credentials, stored in `users.txt`. Uses Singleton for single-instance data management.
- **ArtefactManager** (`artefacts.py`): Handles artefact CRUD operations, including encryption, checksums, and timestamping, stored in `artefacts.txt`. Also uses Singleton.
- **LogManager** (`logs.py`): Records audit logs in `logs.txt`, ensuring traceability through the use of the Singleton pattern.
This structure ensures encapsulation, modularity, and testability, aligning with the proposal’s object-oriented principles (Section 6, Secure Music Copyright Management Application – Design Proposal).

## Design Patterns
The application implements four design patterns:
- **Singleton**: `UserManager`, `ArtefactManager`, and `LogManager` ensure single instances for consistent data access (`users.py`, `artefacts.py`, `logs.py`). For example, `ArtefactManager` prevents multiple instances from overwriting `artefacts.txt`, aligning with the proposal’s database connection handling.
- **Factory**: The `RoleFactory` creates role-based permission strategies (`AdminStrategy`, `UserStrategy`) in `users.py`. For instance, `RoleFactory.get_role_strategy('admin')` returns an `AdminStrategy` granting deletion rights, fulfilling the proposal’s artefact instantiation pattern.
- **Strategy**: The `RoleStrategy` pattern defines permissions (`users.py`). `AdminStrategy` allows deletion and updates, while `UserStrategy` restricts deletion, ensuring role-based access as per the proposal’s encryption/checksum logic encapsulation.
- **Composition**: `MusicApp` integrates `UserManager`, `ArtefactManager`, and `LogManager` (`main.py`), enabling cohesive functionality. For example, creating an artefact involves `ArtefactManager` for storage, `UserManager` for authentication, and `LogManager` for logging, reflecting the proposal’s composition-based design.

## Security Features
Security is a cornerstone of the application:
- **AES-256 Encryption**: Artefacts are encrypted using `cryptography’s Fernet`in `artefacts.py`, ensuring confidentiality. For example, an `.mp3` file is encrypted before storage in `artefacts.txt`.
- **SHA-256 Checksums**: Integrity is verified during artefact creation and reading (`artefacts.py`). 
- **Timestamping**: Creation and modification timestamps are recorded in `artefacts.txt` (`artefacts.py`), supporting auditability.
- **Role-Based Access**: Admins delete artefacts; all users create, read, and update (`users.py`), enforced via `RoleStrategy`.
- **Authentication**: Passwords are hashed with SHA-256, with account lockout after three failed attempts (`users.py`), preventing brute-force attacks.
- **Password Strength**: Requires complex passwords (e.g., `My$ecure123!`) to enhance security (`users.py`).
- **Audit Logging**: Actions (e.g., login, artefact creation) are logged in `logs.txt` (`logs.py`), ensuring traceability.
- **File Restrictions**: Uploads are limited to `.txt` and `.mp3` (max 20MB) in `artefacts.py`, reducing attack surfaces.
- **Key Management**: A Fernet key is stored in `encryption_key.key` (`artefacts.py`) for encryption, generated on first run.

## Data Storage
Data is stored in JSON-based text files:
- `users.txt`: user_id, name, role, email, password_hash, login_attempts, locked.
- `artefacts.txt`: artefact_id, user_id, title, encrypted_data, checksum, timestamp_created, timestamp_modified.
- `logs.txt`: log_id, action_type, actor_id, timestamp.
- `encryption_key.key`: Fernet key for AES-256 encryption.

## Deviations from the Design Proposal
The implementation deviates from the proposal to suit a prototype context:
1. **Text Files Instead of Relational Database** :
   - **Deviation**: Uses JSON text files instead of a relational database.
   - **Justification**: Text files simplify deployment by eliminating the need for database setup, making them ideal for prototyping. The schema mirrors the proposal’s structure, ensuring functional equivalence (Buelta, 2022).
2. **CLI excluding RESTful APIs** :
   - **Deviation**: Implements a CLI instead of RESTful endpoints.
   - **Justification**: A CLI reduces complexity, enabling standalone operation without web server dependencies, and supports rapid iteration in secure, agile development (Sharma and Bawa, 2022).
3. **Integrated Services** :
   - **Deviation**: Combines `ChecksumService`, `EncryptionService`, and `TimeStampService` into `ArtefactManager`.
   - **Justification**: Integration reduces code complexity and maintenance overhead, maintaining functionality in a single module (Shahin et al., 2017).
4. **Key Management** :
   - **Deviation**: Stores Fernet key in `encryption_key.key` instead of a cryptographically secure source.
   - **Justification**: File-based storage simplifies prototyping; production requires a secure vault as per OWASP & NIST guidelines (Souppaya et. al., 2022).
5. **File Type Restrictions** :
   - **Deviation**: Limits uploads to `.txt` and `.mp3` (max 20MB).
   - **Justification**: Restricting types enhances security and focuses on music artefacts, aligning with NIST guidelines (Souppaya et al., 2022).
6. **Additional Security Features** :
   - **Deviation**: Adds account lockout and strong password requirements.
   - **Justification**: Enhances protection against unauthorised access, complying with global security standards (Sarrar et al., 2024).
7. **Admin-Only Deletion** :
   - **Deviation**: Restricts deletion to Admins.
   - **Justification**: Prevents unauthorized data loss, reinforcing role-based security.

## Testing
- **Code Quality**:
  ```bash
Pylint main.py users.py artefacts.py logs.py
  ```
- **Security**:
  ```bash
bandit -r main.py users.py artefacts.py logs.py
  ```
- **Functional**: Tested CRUD operations, encryption/decryption, role-based access, logging, and security features.

## External Libraries
- **cryptography**: AES-256 encryption (Kehrer, 2025). Source: https://pypi.org/project/cryptography/ 
- **Standard Libraries**: `hashlib`, `json`, `os`, `datetime`, `getpass`, `re`.

**References**

Alshar’e, M. (2023) ‘CYBER SECURITY FRAMEWORK SELECTION: COMPARISION OF NIST AND ISO27001’, Applied computing Journal, pp. 245–255. Available at: https://doi.org/10.52098/acj.202364.

Buelta, J. (2022) PYTHON ARCHITECTURE PATTERNS: master api design, event-driven structures, and package management... in python. S.l.: PACKT PUBLISHING LIMITED.

Kehrer, H. (2025) cryptography 45.0.3 Available from: https://pypi.org/project/cryptography/ [Accessed 30th May 2025]

Phatak, S. (2025) Secure Music Copyright Management Application – Design Proposal. Secure Software Development April 2025 [SSD_PCOM7E April 2025] Development Team Project: Design Document. University of Essex Online.

Sarrar, A. et al. (2024) ‘Secure Coding Practices for Web Applications: Addressing Cyber Threats and Safeguarding User Data - A Comprehensive Review’, in 2024 10th International Conference on Computing, Engineering and Design (ICCED). 2024 10th International Conference on Computing, Engineering and Design (ICCED), Jeddah, Saudi Arabia: IEEE, pp. 1–6. Available at: https://doi.org/10.1109/ICCED64257.2024.10983330.

Shahin, M., Ali Babar, M. and Zhu, L. (2017) ‘Continuous Integration, Delivery and Deployment: A Systematic Review on Approaches, Tools, Challenges and Practices’, IEEE Access, 5, pp. 3909–3943. Available at: https://doi.org/10.1109/ACCESS.2017.2685629.

Sharma, A. and Bawa, R.K. (2022) ‘Identification and integration of security activities for secure agile development’, International Journal of Information Technology, 14(2), pp. 1117–1130. Available at: https://doi.org/10.1007/s41870-020-00446-4.

Souppaya, M., Scarfone, K. and Dodson, D. (2022) Secure Software Development Framework (SSDF) version 1.1 : recommendations for mitigating the risk of software vulnerabilities. NIST SP 800-218. Gaithersburg, MD: National Institute of Standards and Technology (U.S.), p. NIST SP 800-218. Available at: https://doi.org/10.6028/NIST.SP.800-218.