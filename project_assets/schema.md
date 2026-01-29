# Event Photo Sharing App - Database Schema

This document outlines the database schema for the Event Photo Sharing application. The schema is designed to support Users, Organizations, Events, Media Collections, Assets, and granular Permissions.

## 1. Identity & Access Management

### able: `account`

Stores individual user credentials and verification status.

| Column Name  | Data Type | Nullable | Key    | Description                                 |
| :----------- | :-------- | :------- | :----- | :------------------------------------------ |
| `account_id` | BIGINT    | No       | PK     | Unique User Identifier.                     |
| `type`       | VARCHAR   | Yes      | Unique | Discriminator:`'USER'` or `'ORGANIZATION'`. |
| `disabled`   | BOOLEAN   | False    |        | Account disabled or enabled                 |
| `created_at` | TIMESTAMP | No       |        | Audit timestamp.                            |

### Table: `users`

Stores individual user credentials and verification status.

| Column Name         | Data Type | Nullable | Key    | Description                                                             |
| :------------------ | :-------- | :------- | :----- | :---------------------------------------------------------------------- |
| `user_id`           | BIGINT    | No       | PK     | Unique User Identifier.                                                 |
| `account_id`        | VARCHAR   | Yes      | FK     | Reference to `account.account_id`                                       |
| `email`             | VARCHAR   | Yes      | Unique | User's email address.                                                   |
| `phone_number`      | VARCHAR   | Yes      | Unique | User's phone number.                                                    |
| `is_phone_verified` | BOOLEAN   | No       |        | Default `false`.                                                        |
| `is_email_verified` | BOOLEAN   | No       |        | Default `false`. At least one verification required for service access. |

### Table: `organizations`

Represents groups or companies that can own events.

| Column Name       | Data Type | Nullable | Key | Description                                         |
| :---------------- | :-------- | :------- | :-- | :-------------------------------------------------- |
| `organization_id` | BIGINT    | No       | PK  | Unique Organization Identifier.                     |
| `account_id`      | VARCHAR   | Yes      | FK  | Reference to `account`.account_id.                  |
| `name`            | VARCHAR   | No       |     | Display name of the organization.                   |
| `admin_user_id`   | BIGINT    | No       | FK  | Reference to `users.user_id`. The admin of the org. |

### Table: `organization_members`

A join table handling the Many-to-Many relationship between Users and Organizations.

| Column Name       | Data Type | Nullable | Key | Description                                        |
| :---------------- | :-------- | :------- | :-- | :------------------------------------------------- |
| `organization_id` | BIGINT    | No       | FK  | Reference to `organizations.id`.                   |
| `user_id`         | BIGINT    | No       | FK  | Reference to `users.id`.                           |
| `role`            | VARCHAR   | Yes      |     | Role within the org (e.g., 'MEMBER', 'MODERATOR'). |

---

## 2. Events & Content Structure

### Table: `parent_events`

The primary container for content. Can be owned by a User or an Organization.

| Column Name        | Data Type | Nullable | Key | Description                                                                  |
| :----------------- | :-------- | :------- | :-- | :--------------------------------------------------------------------------- |
| `parent_event_id`  | BIGINT    | No       | PK  | Unique Event Identifier.                                                     |
| `title`            | VARCHAR   | No       |     | Event Title.                                                                 |
| `description`      | TEXT      | Yes      |     | Event Description.                                                           |
| `owner_account_id` | BIGINT    | No       | FK  | ID of the User or Organization who owns this.<br />Reference to `account.id` |
| `creator_id`       | BIGINT    | No       | FK  | Reference to `users.id` (The actual user who created the record).            |

### Table: `child_events`

Children of the parent event.

| Column Name            | Data Type | Nullable | Key | Description                                                                                   |
| :--------------------- | :-------- | :------- | :-- | :-------------------------------------------------------------------------------------------- |
| `child_event_id`       | BIGINT    | No       | PK  | Unique Event Identifier.                                                                      |
| `root_parent_event_id` | BIGINT    | No       | FK  | Reference to `parent_events.parent_event_id`                                                  |
| `immediate_parent`     | BIGINT    | No       | FK  | Self reference to this same table in case of multple heirachies `child_events.child_event_id` |
| `title`                | VARCHAR   | No       |     | Event Title.                                                                                  |
| `description`          | TEXT      | Yes      |     | Event Description.                                                                            |

## 3. Events Management Structure

Meta data on who manages the event

### Table: `event_management_groups`

| Column Name       | Data Type | Nullable | Key | Description                                  |
| :---------------- | :-------- | :------- | :-- | :------------------------------------------- |
| `group_id`        | BIGINT    | No       | PK  | Unique Group Identifier.                     |
| `parent_event_id` | BIGINT    | Yes      | FK  | Reference to `parent_events.parent_event_id` |
| `child_event_id`  | BIGINT    | Yes      | FK  | Reference to `child_events.child_event_id`   |
| `title`           | VARCHAR   | No       |     | Event Title.                                 |
| `description`     | TEXT      | Yes      |     | Event Description.                           |
| `lead_user_id`    | BIGINT    | Yes      | FK  | Reference to `users.user_id`.                |
| `event_type`      | VARCHAR   | Yes      |     | Event Type `parent` or `child`               |

### Table: `event_management_group_members`

| Column Name | Data Type | Nullable | Key | Description                                      |
| :---------- | :-------- | :------- | :-- | :----------------------------------------------- |
| `group_id`  | BIGINT    | No       | FK  | Reference to `event_management_groups.group_id`. |
| `user_id`   | BIGINT    | No       | FK  | Reference to `users.user_id`.                    |
| `role`      | VARCHAR   | Yes      |     | Role within the group.                           |

## 4. Events Media Structure

Media data related to the event

### Table: `event_media`

| Column Name       | Data Type | Nullable | Key | Description                                  |
| :---------------- | :-------- | :------- | :-- | :------------------------------------------- |
| `event_media_id`  | BIGINT    | No       | PK  | Unique Collection Identifier.                |
| `parent_event_id` | BIGINT    | Yes      | FK  | Reference to `parent_events.parent_event_id` |
| `child_event_id`  | BIGINT    | Yes      | FK  | Reference to `child_events.child_event_id`   |
| `media_id`        | BIGINT    | Yes      | FK  | Reference to `media.media_id`                |
| `name`            | VARCHAR   | Yes      |     | name of the media                            |
| `description`     | TEXT      | Yes      |     | Description of the media                     |

### Table: `event_media_collection`

Logical groupings of media within an event (e.g., "Wedding Ceremony", "Party").

| Column Name          | Data Type | Nullable | Key | Description                                  |
| :------------------- | :-------- | :------- | :-- | :------------------------------------------- |
| `event_media_col_id` | BIGINT    | No       | PK  | Unique Collection Identifier.                |
| `parent_event_id`    | BIGINT    | Yes      | FK  | Reference to `parent_events.parent_event_id` |
| `child_event_id`     | BIGINT    | Yes      | FK  | Reference to `child_events.child_event_id`   |
| `collection_name`    | VARCHAR   | Yes      |     | name of the collection                       |
| `description`        | TEXT      | Yes      |     | Description of the collection                |

### Table: `event_media_collection_media_relation`

Links event media collection with each media

| Column Name          | Data Type | Nullable | Key | Description                                              |
| :------------------- | :-------- | :------- | :-- | :------------------------------------------------------- |
| `id`                 | BIGINT    | No       | PK  | Unique Collection Identifier.                            |
| `event_media_col_id` | BIGINT    | No       |     | Reference to `event_media_collection.event_media_col_id` |
| `media_id`           | BIGINT    | Yes      | FK  | Reference to `media.media_id`                            |
| `name`               | VARCHAR   | Yes      |     | name of the media                                        |
| `description`        | TEXT      | Yes      |     | Description of the media                                 |

### Table: `media`

Media within an event, this does not belong to collection

| Column Name         | Data Type | Nullable | Key | Description                                  |
| :------------------ | :-------- | :------- | :-- | :------------------------------------------- |
| `media_id`          | BIGINT    | No       | PK  | Unique Collection Identifier.                |
| `parent_event_id`   | BIGINT    | No       | FK  | Reference to `parent_events.parent_event_id` |
| `asset_id`          | BIGINT    | Yes      | FK  | Reference to `assets.asset_id`               |
| `media_name`        | VARCHAR   | Yes      |     | Name of the media                            |
| `media_description` | TEXT      | Yes      |     | Description of the media                     |

---

## 3. Assets & Storage

This section decouples asset from physical storage location (Storage Location).

### Table: `assets`

Metadata for media files. Linked polymorphically to Events (banners/thumbnails) or MediaCollections (photos).

| Column Name      | Data Type | Nullable | Key | Description                           |
| :--------------- | :-------- | :------- | :-- | :------------------------------------ |
| `asset_id`       | BIGINT    | No       | PK  | Unique Asset Identifier.              |
| `type`           | VARCHAR   | No       |     | Enum:`'IMAGE'`, `'AUDIO'`, `'VIDEO'`. |
| `file_name`      | VARCHAR   | Yes      |     | Optional display name for the file.   |
| `file_extension` | VARCHAR   | Yes      |     | `pdf, parquet`                        |

### Table: `storage_locations`

Represents the physical location of a file in the cloud.

| Column Name | Data Type | Nullable | Key | Description                                 |
| :---------- | :-------- | :------- | :-- | :------------------------------------------ |
| `id`        | BIGINT    | No       | PK  | Unique Storage Identifier.                  |
| `s3_url`    | VARCHAR   | No       |     | Public or Presigned URL to the object.      |
| `arn`       | VARCHAR   | Yes      |     | Amazon Resource Name (for IAM/Permissions). |
| `asset_id`  | BIGINT    | No       | FK  | Reference to `assets.asset_id`              |

---

## 4. Permissions System (RBAC & Granular Control)

A hierarchical Role-Based Access Control system supporting direct assignments, inheritance, denial blocks, and default role mappings.

### Table: `permissions`

Definitions of atomic actions that can be performed.

| Column Name   | Data Type | Nullable | Key    | Description                                            |
| :------------ | :-------- | :------- | :----- | :----------------------------------------------------- |
| `id`          | BIGINT    | No       | PK     | Unique Permission Identifier.                          |
| `slug`        | VARCHAR   | No       | Unique | System identifier (e.g.,`event.delete`, `org.create`). |
| `description` | TEXT      | Yes      |        | Human-readable description.                            |

### Table: `roles`

Named collections of permissions (e.g., 'ORG_ADMIN', 'EDITOR').

| Column Name   | Data Type | Nullable | Key    | Description               |
| :------------ | :-------- | :------- | :----- | :------------------------ |
| `id`          | BIGINT    | No       | PK     | Unique Role Identifier.   |
| `name`        | VARCHAR   | No       | Unique | Display name of the role. |
| `description` | TEXT      | Yes      |        | Optional description.     |

### Table: `role_permissions`

Many-to-Many link between Roles and Permissions.

| Column Name     | Data Type | Nullable | Key | Description                    |
| :-------------- | :-------- | :------- | :-- | :----------------------------- |
| `role_id`       | BIGINT    | No       | FK  | Reference to `roles.id`.       |
| `permission_id` | BIGINT    | No       | FK  | Reference to `permissions.id`. |

### Table: `default_role_assignments`

Defines which roles are automatically granted to strict entity types (User, Org) by default.
_Example: All Users get 'ORG_CREATOR' role._

| Column Name     | Data Type | Nullable | Key | Description                                                    |
| :-------------- | :-------- | :------- | :-- | :------------------------------------------------------------- |
| `id`            | BIGINT    | No       | PK  | Unique ID.                                                     |
| `assignee_type` | VARCHAR   | No       |     | Discriminator:`USER`, `ORGANIZATION`.                          |
| `role_id`       | BIGINT    | No       | FK  | Reference to `roles.id`. The role granted to all of this type. |

### Table: `access_controls` (Assignments)

The primary table for granting Roles to Subjects (User/Group/Org) on Resources (Event/Collection).

| Column Name     | Data Type | Nullable | Key | Description                                            |
| :-------------- | :-------- | :------- | :-- | :----------------------------------------------------- |
| `id`            | BIGINT    | No       | PK  | Unique ID.                                             |
| `subject_type`  | VARCHAR   | No       |     | `USER`, `ORGANIZATION`, `GROUP`.                       |
| `subject_id`    | BIGINT    | No       |     | ID of the assignee.                                    |
| `resource_type` | VARCHAR   | No       |     | `PARENT_EVENT`, `CHILD_EVENT`, `COLLECTION`, `SYSTEM`. |
| `resource_id`   | BIGINT    | Yes      |     | ID of the target resource (NULL/0 for Global/System).  |
| `role_id`       | BIGINT    | No       | FK  | Reference to `roles.id`.                               |

### Table: `permission_blocks` (Denials)

Explicitly denies a specific Permission (NOT Role) to a Subject.
**Priority:** This table is checked _first_. If a match is found, access is DENIED immediately, overriding any grant.

| Column Name     | Data Type | Nullable | Key | Description                                                         |
| :-------------- | :-------- | :------- | :-- | :------------------------------------------------------------------ |
| `id`            | BIGINT    | No       | PK  | Unique ID.                                                          |
| `subject_type`  | VARCHAR   | No       |     | `USER`, `ORGANIZATION`, `GROUP`.                                    |
| `subject_id`    | BIGINT    | No       |     | ID of the assignee.                                                 |
| `resource_type` | VARCHAR   | No       |     | `PARENT_EVENT`, `CHILD_EVENT`, `COLLECTION`, `SYSTEM`.              |
| `resource_id`   | BIGINT    | Yes      |     | ID of the target resource.                                          |
| `permission_id` | BIGINT    | No       | FK  | Reference to `permissions.id`. The specific atomic action to block. |

---

## 5. Groups

Reusable user groups for permission assignment.

### Table: `user_groups`

| Column Name    | Data Type | Nullable | Key | Description                               |
| :------------- | :-------- | :------- | :-- | :---------------------------------------- |
| `id`           | BIGINT    | No       | PK  | Unique Group Identifier.                  |
| `name`         | VARCHAR   | No       |     | Group Name (e.g., "Marketing Team").      |
| `owner_org_id` | BIGINT    | Yes      | FK  | Optional Reference to `organizations.id`. |

### Table: `user_group_members`

| Column Name | Data Type | Nullable | Key | Description                    |
| :---------- | :-------- | :------- | :-- | :----------------------------- |
| `group_id`  | BIGINT    | No       | FK  | Reference to `user_groups.id`. |
| `user_id`   | BIGINT    | No       | FK  | Reference to `users.id`.       |

---

## Relationship Summary

- **User -> Organization:** Many-to-Many
- **Organization -> User (Admin):** One-to-One
- **Event -> MediaCollection:** One-to-Many
- **Event/MediaCollection -> Asset:** One-to-Many (Polymorphic)
- **Role -> Permissions:** Many-to-Many
- **Access Control:** Polymorphically links Subjects (User/Org/Group) to Resources via Roles.
- **Blocks:** Polymorphically denies Permissions to Subjects.
