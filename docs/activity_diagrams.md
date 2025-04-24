# Activity Diagrams

This document outlines the key workflows in the Taiga backend system using activity diagrams.

## Project Lifecycle Workflow

```plantuml
@startuml
start
:User initiates project creation;

if (Check if project can be created?) then (yes)
  if (From template?) then (yes)
    :Load project template;
    :Apply template settings;
  else (no)
    :Use default settings;
  endif
  
  :Create project;
  :Set up default configurations;
  :Create initial project roles;
  :Add creator as admin;
  
else (no)
  :Show error message;
  stop
endif

:Project created successfully;
stop
@enduml
```

## Project Transfer Workflow

```plantuml
@startuml
start
:Owner initiates project transfer;

if (Project can be transferred?) then (yes)
  :Create transfer request;
  :Send notification to new owner;
  
  fork
    :Wait for response;
  fork again
    :Transfer token expires;
    :Cancel transfer;
    stop
  end fork
  
  if (New owner accepts?) then (yes)
    :Validate transfer conditions;
    if (Valid?) then (yes)
      :Transfer ownership;
      :Update project settings;
      :Notify members;
    else (no)
      :Cancel transfer;
    endif
  else (no)
    :Reject transfer;
    :Notify original owner;
  endif
  
else (no)
  :Show error message;
endif

stop
@enduml
```

## Member Management Workflow

```plantuml
@startuml
start
:Admin initiates member action;

if (Action type?) then (invite)
  if (Can add more members?) then (yes)
    :Create invitation;
    :Send invitation email;
    
    fork
      :Wait for response;
    fork again
      :Invitation expires;
      :Remove invitation;
      stop
    end fork
    
    if (User accepts?) then (yes)
      if (Existing user?) then (yes)
        :Add to project members;
      else (no)
        :Create new user;
        :Add to project members;
      endif
    else (no)
      :Mark invitation as rejected;
    endif
    
  else (no)
    :Show member limit error;
  endif
  
else (remove)
  if (Is last admin?) then (yes)
    :Show error - Need at least one admin;
  else (no)
    :Remove member;
    :Update project permissions;
  endif
endif

stop
@enduml
```

## Project Configuration Workflow

```plantuml
@startuml
start
:Admin initiates configuration change;

fork
  :Update privacy settings;
  if (Can change privacy?) then (yes)
    :Apply new privacy settings;
    :Update member permissions;
  else (no)
    :Show privacy change error;
  endif
fork again
  :Update modules config;
  :Enable/disable features;
  :Configure module settings;
fork again
  :Update project details;
  :Modify basic information;
  :Update project metadata;
end fork

:Save configuration changes;
:Notify project members;

stop
@enduml
```

## Task/Issue Management Workflow

```plantuml
@startuml
start
:User creates task/issue;

:Set initial status;
:Assign priority/severity;
:Set due date (optional);

fork
  :Update status;
  :Reorder status items;
fork again
  :Update assignment;
  :Notify assigned users;
fork again
  :Add comments/attachments;
  :Notify watchers;
end fork

:Track activity;
:Update project statistics;

stop
@enduml
```

## Key Points About Workflows

1. **Project Management**
   - Projects can be created from templates or from scratch
   - Ownership can be transferred with proper validation
   - Projects can be duplicated or deleted
   - Privacy settings affect member permissions

2. **Member Management**
   - Supports both direct addition and invitation system
   - Enforces member limits
   - Maintains at least one admin
   - Handles user roles and permissions

3. **Configuration Management**
   - Modular approach to feature enabling/disabling
   - Flexible privacy settings
   - Customizable workflows for different item types
   - Project-specific configurations

4. **Task/Issue Tracking**
   - Customizable workflows
   - Status ordering
   - Assignment and notification system
   - Activity tracking and statistics

5. **Security Considerations**
   - Permission checks at each step
   - Validation of user actions
   - Member limit enforcement
   - Privacy setting restrictions

These workflows demonstrate the complex interactions between different components of the system while maintaining data integrity and user permissions.