# Technical Document
  

Owned by [Luis Deras](https://guababit.atlassian.net/wiki/people/557058:ddc5d920-dc75-4d6d-9e1d-5c7ede41748c?ref=confluence&src=profilecard)

Last updated: [Oct 29, 2024](https://guababit.atlassian.net/wiki/pages/diffpagesbyversion.action?pageId=91619346&selectedPageVersions=1&selectedPageVersions=2)

3 min read

See how many people viewed this page

**Objective**  
Define data models and endpoints for implementing the announcements feature in the Luqa app, using DynamoDB and NestJS. The feature includes filtering for unread announcements, restricting actions based on active status, and sending notifications upon activation.

---

### 1. **Data Model**

#### **Table**: `Announcement`

- **Partition Key**: `TenantId` (string) – Identifier for each client or community.
    
- **Sort Key**: `AnnouncementId` (string, UUID) – Unique identifier for each announcement.
    
- **Attributes**:
    
    - `Title` (string) – Title of the announcement.
        
    - `Body` (string) – Main content of the announcement.
        
    - `CreatedAt` (number, Unix timestamp) – Timestamp of when the announcement was created.
        
    - `CreatedBy` (string) – User ID of the admin who created the announcement.
        
    - `IsActive` (boolean) – Flag indicating if the announcement is active (defaults to `false` upon creation).
        

#### **Table**: `AnnouncementReadStatus`

- **Partition Key**: `AnnouncementId` (string) – Identifier of the announcement.
    
- **Sort Key**: `UserId` (string) – Unique identifier for each resident.
    
- **Attributes**:
    
    - `ReadAt` (number, Unix timestamp) – Timestamp indicating when the announcement was marked as read.
        
    - `TenantId` (string) – Ensures isolation between tenants/clients.
        

This structure enables efficient querying for unread announcements per user while ensuring tenant isolation.

---

### 2. **Endpoints**

#### **1. Create an Announcement**

- **Endpoint**: `POST /tenant/:tenantId/announcement`
    
- **Purpose**: Allows an admin to create a new announcement.
    
- **Method**: POST
    
- **Authorization**: Admin only
    
- **Path Parameters**:
    
    - `tenantId` (string) – Identifier for the tenant.
        
- **Request Body**:
    
    `{ "title": "string", "body": "string" }`
    
- **Response**:
    
    - **201 Created**: Announcement successfully created with `IsActive` set to `false`.
        
    - **Error Codes**: 400 Bad Request, 401 Unauthorized, 500 Internal Server Error
        
- **DynamoDB Actions**:
    
    - Add a new item to the `Announcement` table.
        
- **Notes**:
    
    - Upon creation, the announcement is inactive by default. Activation is managed through the `PATCH` endpoint.
        

---

#### **2. Retrieve Announcements with Filter Options**

- **Endpoint**: `GET /tenant/:tenantId/announcement`
    
- **Purpose**: Retrieve a list of announcements for a specific tenant, with options to filter based on unread status.
    
- **Method**: GET
    
- **Authorization**: Resident and admin
    
- **Path Parameters**:
    
    - `tenantId` (string) – Identifier for the tenant.
        
- **Query Parameters**:
    
    - `unread` (boolean, optional) – If set to `true`, retrieves only unread announcements for the authenticated user.
        
- **Headers**:
    
    - **Authorization**: Bearer token containing `userId`
        
- **Response**:
    
    `{ "announcements": [ { "announcementId": "string", "title": "string", "body": "string", "createdAt": "number", "createdBy": "string", "isRead": "boolean" // Optional, indicates if announcement is read for this user } ] }`
    
- **DynamoDB Actions**:
    
    1. **Step 1**: Query the `Announcement` table to retrieve all announcements for the specified `tenantId`, filtering for active announcements.
        
    2. **Step 2**: If `unread=true`, query the `AnnouncementReadStatus` table with `AnnouncementId` and `UserId` to get announcements the user has already read.
        
    3. **Step 3**: Filter the announcements from Step 1 by excluding those found in Step 2, returning only unread announcements if `unread=true`.
        

---

#### **3. Delete an Announcement**

- **Endpoint**: `DELETE /tenant/:tenantId/announcement/:announcementId`
    
- **Purpose**: Allows an admin to delete an announcement if it has not yet been activated.
    
- **Method**: DELETE
    
- **Authorization**: Admin only
    
- **Path Parameters**:
    
    - `tenantId` (string) – Identifier for the tenant.
        
    - `announcementId` (string) – Identifier for the announcement to delete.
        
- **Response**:
    
    - **200 OK**: Announcement deleted successfully.
        
    - **Error Codes**:
        
        - **400 Bad Request**: If the announcement is already active.
            
        - **404 Not Found**: If the announcement or tenant does not exist.
            
        - **401 Unauthorized**: If the user does not have sufficient permissions.
            
        - **500 Internal Server Error**: For any unexpected server issues.
            
- **DynamoDB Actions**:
    
    - Check the `IsActive` attribute:
        
        - **If** `IsActive=false`: Proceed with deletion by removing the item from the `Announcement` table.
            
        - **If** `IsActive=true`: Return a `400 Bad Request` error, indicating that active announcements cannot be deleted.
            

---

#### **4. Activate an Announcement**

- **Endpoint**: `PATCH /tenant/:tenantId/announcement/:announcementId`
    
- **Purpose**: Allows an admin to activate an announcement and triggers a push notification to all users in the tenant.
    
- **Method**: PATCH
    
- **Authorization**: Admin only
    
- **Path Parameters**:
    
    - `tenantId` (string) – Identifier for the tenant.
        
    - `announcementId` (string) – Identifier for the announcement.
        
- **Request Body**:
    
    - Empty (activation only, no other updates allowed).
        
- **Response**:
    
    - **200 OK**: Announcement activated successfully, and notification sent.
        
    - **Error Codes**: 400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Internal Server Error
        
- **DynamoDB Actions**:
    
    - Update the `IsActive` attribute to `true` for the specified announcement.
        
- **Notification Logic**:
    
    - **Trigger**: Upon successful activation, trigger a push notification to all users associated with the `tenantId`.
        
    - **Notification Message**:
        
        - **Content**: If `Title` exceeds 20 characters, send only the first 20 characters followed by `"..."`.
            
        - **Example**:
            
            - Title: `"Community Pool Closing Early"`
                
            - Notification Message: `"Community Pool Closin..."`
                

---

#### **5. Mark Announcement as Read**

- **Endpoint**: `POST /tenant/:tenantId/announcement/:announcementId/read`
    
- **Purpose**: Allows a resident to mark an announcement as read.
    
- **Method**: POST
    
- **Authorization**: Resident only
    
- **Path Parameters**:
    
    - `TenantId` (string) – Identifier for the tenant.
        
    - `AnnouncementId` (string) – Identifier for the announcement.
        
- **Headers**:
    
    - **Authorization**: Bearer token containing `userId`
        
- **Request Body**:
    
    - Empty (no need for `userId` in the body).
        
- **Response**:
    
    - **200 OK**: Successfully marked as read.
        
    - **Error Codes**: 404 Not Found, 401 Unauthorized, 500 Internal Server Error
        
- **DynamoDB Actions**:
    
    - Extract `userId` from the authorization token.
        
    - Insert a new item into `AnnouncementReadStatus` with `AnnouncementId`, `UserId`, `ReadAt`, and `TenantId`.