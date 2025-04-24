# Sequence Diagrams

This document illustrates the key interactions between different components of the Taiga backend system.

## Project Creation Sequence

```plantuml
@startuml
actor Client
participant "API Layer" as API
participant "Validator" as Validator
participant "Permission Service" as PermService
participant "Project Service" as ProjService
database Database

Client -> API: POST /api/v1/projects/
activate API

API -> Validator: validate_project_data()
activate Validator
Validator --> API: validated_data
deactivate Validator

API -> PermService: check_permissions()
activate PermService
PermService --> API: permission_granted
deactivate PermService

API -> ProjService: create_project()
activate ProjService
ProjService -> Database: save_project()
Database --> ProjService: project_created
ProjService -> ProjService: setup_default_config()
ProjService -> ProjService: create_default_roles()
ProjService --> API: project_data
deactivate ProjService

API --> Client: 201 Created
deactivate API
@enduml
```

## Project Transfer Sequence

```plantuml
@startuml
actor "Current Owner" as Owner
actor "New Owner" as NewOwner
participant "API Layer" as API
participant "Transfer Service" as Transfer
database Database

Owner -> API: POST /projects/{id}/transfer_start
activate API
API -> Transfer: start_project_transfer()
Transfer -> Database: create_transfer_request()
Transfer -> NewOwner: send_transfer_notification()
API --> Owner: 200 OK
deactivate API

NewOwner -> API: POST /projects/{id}/transfer_accept
activate API
API -> Transfer: validate_transfer_token()
API -> Transfer: check_transfer_limitations()
Transfer -> Database: update_project_owner()
Transfer -> Database: update_memberships()
Transfer -> Owner: send_transfer_accepted_notification()
API --> NewOwner: 200 OK
deactivate API
@enduml
```

## Project Duplication Sequence

```plantuml
@startuml
actor Client
participant "API Layer" as API
participant "Validator" as Validator
participant "Project Service" as ProjService
database Database

Client -> API: POST /projects/{id}/duplicate
activate API

API -> Validator: validate_duplication_data()
activate Validator
Validator --> API: validated_data
deactivate Validator

API -> ProjService: check_duplication_limits()
activate ProjService
ProjService --> API: limits_ok
deactivate ProjService

API -> ProjService: duplicate_project()
activate ProjService
ProjService -> Database: create_new_project()
ProjService -> ProjService: copy_project_settings()
ProjService -> ProjService: copy_project_data()
ProjService -> ProjService: setup_memberships()
ProjService --> API: new_project_data
deactivate ProjService

API --> Client: 201 Created
deactivate API
@enduml
```

## Member Management Sequence

```plantuml
@startuml
actor Admin
participant "API Layer" as API
participant "Member Service" as MemberService
participant "Notification Service" as NotifyService
database Database

Admin -> API: POST /projects/{id}/memberships
activate API

API -> MemberService: check_member_limits()
activate MemberService
MemberService --> API: limits_ok
deactivate MemberService

alt New User
    API -> MemberService: create_invitation()
    MemberService -> NotifyService: send_invitation_email()
else Existing User
    API -> MemberService: add_member()
    MemberService -> Database: create_membership()
    MemberService -> NotifyService: notify_new_membership()
end

API --> Admin: 201 Created
deactivate API

@enduml
```

## Module Configuration Sequence

```plantuml
@startuml
actor Admin
participant "API Layer" as API
participant "Module Service" as ModService
database Database

Admin -> API: PATCH /projects/{id}/modules
activate API

API -> ModService: get_current_config()
activate ModService
ModService -> Database: fetch_config()
Database --> ModService: current_config
ModService --> API: config
deactivate ModService

API -> ModService: update_config()
activate ModService
ModService -> ModService: validate_config()
ModService -> Database: save_config()
ModService --> API: updated_config
deactivate ModService

API --> Admin: 204 No Content
deactivate API
@enduml
```

## Key Points About Sequences

1. **Authentication & Authorization**
   - All endpoints require authentication
   - Permission checks are performed before any operation
   - Role-based access control is enforced

2. **Data Validation**
   - Input data is validated before processing
   - Business rules are checked (e.g., member limits)
   - Appropriate error responses are returned

3. **Notifications**
   - System sends notifications for important events
   - Email notifications for invitations
   - In-app notifications for changes

4. **Transaction Management**
   - Complex operations are wrapped in transactions
   - Rollback on failure
   - Data consistency is maintained

5. **Error Handling**
   - Validation errors return 400 Bad Request
   - Permission errors return 403 Forbidden
   - Not found errors return 404 Not Found
   - Server errors return 500 Internal Server Error

These sequence diagrams illustrate the main interactions between different components of the system, showing how data flows and how different services collaborate to accomplish complex tasks.