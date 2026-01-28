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
| `account_id`        | VARCHAR   | Yes      | FK     | Reference to `account`.account_id.                                      |
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

| Column Name       | Data Type | Nullable | Key | Description                     |
| :---------------- | :-------- | :------- | :-- | :------------------------------ |
| `child_event_id`  | BIGINT    | No       | PK  | Unique Event Identifier.        |
| `parent_event_id` | BIGINT    | No       | FK  | Reference to `parent_events.id` |
| `title`           | VARCHAR   | No       |     | Event Title.                    |
| `description`     | TEXT      | Yes      |     | Event Description.              |

## 3. Events Management Structure

Meta data on who manages the event

### Table: `parent_event_management_groups`

| Column Name       | Data Type | Nullable | Key | Description                     |
| :---------------- | :-------- | :------- | :-- | :------------------------------ |
| `group_id`        | BIGINT    | No       | PK  | Unique Group Identifier.        |
| `parent_event_id` | BIGINT    | No       | FK  | Reference to `parent_events.id` |
| `title`           | VARCHAR   | No       |     | Event Title.                    |
| `description`     | TEXT      | Yes      |     | Event Description.              |
| `lead_user_id`    | BIGINT    | Yes      | FK  | Reference to `users.id`.        |

### Table: `parent_event_management_group_members`

| Column Name | Data Type | Nullable | Key | Description                                             |
| :---------- | :-------- | :------- | :-- | :------------------------------------------------------ |
| `group_id`  | BIGINT    | No       | FK  | Reference to `parent_event_management_groups.group_id`. |
| `user_id`   | BIGINT    | No       | FK  | Reference to `users.user_id`.                           |
| `role`      | VARCHAR   | Yes      |     | Role within the group.                                  |

### Table: `child_event_management_groups`

Media within an event, this does not belong to collection

| Column Name      | Data Type | Nullable | Key | Description                    |
| :--------------- | :-------- | :------- | :-- | :----------------------------- |
| `group_id`       | BIGINT    | No       | PK  | Unique Group Identifier.       |
| `child_event_id` | BIGINT    | No       | FK  | Reference to `child_events.id` |
| `title`          | VARCHAR   | No       |     | Event Title.                   |
| `description`    | TEXT      | Yes      |     | Event Description.             |
| `lead_user_id`   | BIGINT    | Yes      | FK  | Reference to `users.id`.       |

### Table: `child_event_management_group_members`

| Column Name | Data Type | Nullable | Key | Description                                            |
| :---------- | :-------- | :------- | :-- | :----------------------------------------------------- |
| `group_id`  | BIGINT    | No       | FK  | Reference to `child_event_management_groups.group_id`. |
| `user_id`   | BIGINT    | No       | FK  | Reference to `users.user_id`.                          |
| `role`      | VARCHAR   | Yes      |     | Role within the group.                                 |

## 4. Events Media Structure

Media data related to the event

### Table: `parent_event_linked_media_collection`

Logical groupings of media within an event (e.g., "Wedding Ceremony", "Party").

| Column Name           | Data Type | Nullable | Key | Description                          |
| :-------------------- | :-------- | :------- | :-- | :----------------------------------- |
| `id`                  | BIGINT    | No       | PK  | Unique Collection Identifier.        |
| `event_id`            | BIGINT    | No       | FK  | Reference to `events.id`.            |
| `asset_collection_id` | BIGINT    | Yes      | FK  | Reference to `asset_collections.id ` |

### Table: `child_event_linked_media`

Media within an event, this does not belong to collection

| Column Name | Data Type | Nullable | Key | Description                   |
| :---------- | :-------- | :------- | :-- | :---------------------------- |
| `id`        | BIGINT    | No       | PK  | Unique Collection Identifier. |
| `event_id`  | BIGINT    | No       | FK  | Reference to `events.id`.     |
| `asset_id`  | BIGINT    | Yes      | FK  | Reference to `asset.id `      |

### Table: `child_event_linked_media_collection`

Logical groupings of media within an event (e.g., "Wedding Ceremony", "Party").

| Column Name           | Data Type | Nullable | Key | Description                          |
| :-------------------- | :-------- | :------- | :-- | :----------------------------------- |
| `id`                  | BIGINT    | No       | PK  | Unique Collection Identifier.        |
| `event_id`            | BIGINT    | No       | FK  | Reference to `events.id`.            |
| `asset_collection_id` | BIGINT    | Yes      | FK  | Reference to `asset_collections.id ` |

---

## 3. Assets & Storage

This section decouples asset from physical storage location (Storage Location).

### Table: `assets`

Metadata for media files. Linked polymorphically to Events (banners/thumbnails) or MediaCollections (photos).

| Column Name      | Data Type | Nullable | Key | Description                                        |
| :--------------- | :-------- | :------- | :-- | :------------------------------------------------- |
| `id`             | BIGINT    | No       | PK  | Unique Asset Identifier.                           |
| `storage_id`     | BIGINT    | No       | FK  | Reference to `storage_locations.id`.               |
| `type`           | VARCHAR   | No       |     | Enum:`'THUMBNAIL'`, `'FEATURE_BANNER'`, `'IMAGE'`. |
| `name`           | VARCHAR   | Yes      |     | Optional display name for the file.                |
| `description`    | TEXT      | Yes      |     | Optional caption/description.                      |
| `reference_id`   | BIGINT    | No       |     | ID of the entity this asset belongs to.            |
| `reference_type` | VARCHAR   | No       |     | Discriminator:`'EVENT'` or `'MEDIA_COLLECTION'`.   |

### Table: `storage_locations`

Represents the physical location of a file in the cloud.

| Column Name | Data Type | Nullable | Key | Description                                 |
| :---------- | :-------- | :------- | :-- | :------------------------------------------ |
| `id`        | BIGINT    | No       | PK  | Unique Storage Identifier.                  |
| `s3_url`    | VARCHAR   | No       |     | Public or Presigned URL to the object.      |
| `arn`       | VARCHAR   | Yes      |     | Amazon Resource Name (for IAM/Permissions). |

---

## 4. Permissions System

### Table: `permissions`

Stores fine-grained access control logic using expression language.

| Column Name   | Data Type | Nullable | Key | Description                                        |
| :------------ | :-------- | :------- | :-- | :------------------------------------------------- |
| `id`          | BIGINT    | No       | PK  | Unique Permission Identifier.                      |
| `name`        | VARCHAR   | No       |     | System name (e.g.,`CAN_EDIT_EVENT`).               |
| `description` | TEXT      | Yes      |     | Human-readable description.                        |
| `expression`  | TEXT      | No       |     | Logic string (e.g.,`resource.ownerId == user.id`). |

### Table: `permission_assignee_types`

Stores fine-grained access control logic using expression language.

| Column Name     | Data Type | Nullable | Key | Description                            |
| :-------------- | :-------- | :------- | :-- | :------------------------------------- |
| `permission_id` | BIGINT    | No       | PK  | Unique Permission Identifier.          |
| `assignee_type` | VARCHAR   | No       |     | System name (e.g.,`USER` or `SYSTEM`). |

---

## Relationship Summary

- **User -> Organization:** Many-to-Many
- **Organization -> User (Admin):** One-to-One
- **Event -> MediaCollection:** One-to-Many
- **Event/MediaCollection -> Asset:** One-to-Many (Polymorphic)
- **Asset -> StorageLocation:** One-to-One (Strictly) or Many-to-One (If reusing files)
