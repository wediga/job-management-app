
# Case Study – Job Management Platform

## Tools Used

- Laravel (Backend Framework)
- PostgreSQL (Database)
- dbdiagram.io (Data Modeling)
- Git (Version Control)
- Markdown (Documentation)

---

## Sitemap

This is the structure of all relevant pages in the application:

### Public

- `/login` – Login page
- `/register` – User registration

### Dashboard (after login)

- `/dashboard` – Overview page after login

### Jobs

- `/jobs` – List all jobs
- `/jobs/{id}` – Show single job details
- `/jobs/create` – Create a new job
- `/jobs/{id}/edit` – Edit an existing job

### Companies

- `/companies` – List all companies
- `/companies/{id}` – Show company details
- `/companies/create` – Create a new company
- `/companies/{id}/edit` – Edit an existing company

### Categories

- `/categories` – List all categories
- `/categories/create` – Create a new category
- `/categories/{id}/edit` – Edit a category

### Users (Admin only)

- `/users` – List all users
- `/users/{id}/edit` – Change role or status

### Other

- `/logout` – Logout action

---

# Database Schema Overview

This document describes the relational database schema designed for a job management platform. The database is fully normalized (3NF) and optimized for clarity, scalability, and integrity. It is designed for PostgreSQL.

## Table: `roles`

### Purpose:
Defines distinct user roles within the application (e.g., `admin`, `editor`, `viewer`). Acts as a central authority for assigning privileges.

## Table: `users`

### Purpose:
Stores all user accounts. Each user is linked to a role and tracks metadata such as activation status and audit fields.

## Table: `passwords`

### Purpose:
Separates password hashes from the main user table for better security and potential auditing. Stores one hashed password per user using a secure algorithm (e.g., Argon2id).

## Table: `permissions`

### Purpose:
Represents individual permission keys (e.g., `job.create`, `user.manage`). Each permission has a label and a description for maintainability.

## Table: `role_to_permissions`

### Purpose:
Many-to-many junction table mapping roles to permissions. Allows flexible and granular access control for all system features.

## Table: `locations`

### Purpose:
Stores normalized location entries used for job postings. Reduces redundancy and enables consistent filtering/grouping.

## Table: `salary_ranges`

### Purpose:
Defines pre-set salary bands (e.g., "50k–70k") to simplify job filtering, searching, and validation.

## Table: `categories`

### Purpose:
Defines job categories or sectors (e.g., IT, Finance, Healthcare). Used for tagging and filtering job listings.

## Table: `companies`

### Purpose:
Stores company data related to job listings. Each company can have a name, website, and optionally a logo path. Used as a reference in job records.

## Table: `jobs`

### Purpose:
The central content unit of the platform. Each job links to a company, category, location, salary range, and a user (creator). This table holds the job title, description, and all relational keys.

## Notes

- All tables include `created_by`, `updated_by`, `created_at`, and `updated_at` fields for full audit traceability.



```sql
-- DDL for PostgreSQL - Normalized Schema

CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT now(),
    created_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL
);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR NOT NULL UNIQUE,
    email VARCHAR NOT NULL UNIQUE,
    role_id INTEGER NOT NULL REFERENCES roles(id),
    is_active BOOLEAN NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    created_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL
);

CREATE TABLE passwords (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    password_hash TEXT NOT NULL,
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE permissions (
    id SERIAL PRIMARY KEY,
    key TEXT NOT NULL UNIQUE,
    label TEXT NOT NULL,
    description TEXT,
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE role_to_permissions (
    id SERIAL PRIMARY KEY,
    role_id INTEGER NOT NULL REFERENCES roles(id),
    permission_id INTEGER NOT NULL REFERENCES permissions(id),
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL UNIQUE,
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE salary_ranges (
    id SERIAL PRIMARY KEY,
    label VARCHAR NOT NULL,
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL UNIQUE,
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE companies (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL UNIQUE,
    website VARCHAR NOT NULL,
    logo_path VARCHAR,
    created_by INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL,
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE jobs (
    id SERIAL PRIMARY KEY,
    title VARCHAR NOT NULL UNIQUE,
    description TEXT,
    location_id INTEGER NOT NULL REFERENCES locations(id),
    salary_range_id INTEGER NOT NULL REFERENCES salary_ranges(id),
    category_id INTEGER NOT NULL REFERENCES categories(id),
    created_by INTEGER NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT now(),
    updated_by INTEGER NOT NULL REFERENCES users(id),
    updated_at TIMESTAMP DEFAULT now()
);
```

---

## Model Relationships

The following relationships exist between the core models:

### `User` → `Role`
- **Type:** Many-to-One
- **Each user has exactly one role**
- **Each role can be assigned to multiple users**

### `Role` → `Permission` (via `role_to_permissions`)
- **Type:** Many-to-Many
- **Each role can have many permissions**
- **Each permission can belong to many roles**

### `User` → `Password`
- **Type:** One-to-One
- **Each user has one password record**
- **Passwords are stored separately for security purposes**

### `Job` → `User` (`created_by`, `updated_by`)
- **Type:** Many-to-One (audit)
- **Each job is created and updated by a user**
- **Users can create multiple jobs**

### `Job` → `Category`
- **Type:** Many-to-One
- **Each job belongs to one category**
- **Each category can be used for many jobs**

### `Job` → `Company`
- **Type:** Many-to-One
- **Each job is posted under one company**
- **Each company can have multiple job postings**

### `Job` → `Location`
- **Type:** Many-to-One
- **Each job is assigned to one location**
- **Each location can be reused by multiple jobs**

### `Job` → `SalaryRange`
- **Type:** Many-to-One
- **Each job has one predefined salary range**
- **Each salary range can be referenced by multiple jobs**

### `Category`, `Company`, `Location`, `SalaryRange` → `User`
- **Type:** Many-to-One (audit: `created_by`, `updated_by`)
- **All reference the user who created or modified the record**

### Visual Summary

- One `User` can create many `Jobs`, `Companies`, `Categories`, etc.
- One `Role` can have many `Permissions` and many `Users`
- `Jobs` reference multiple other models but are central to the system

---

## User Roles and Permissions

The system uses a role-based access control (RBAC) mechanism. Each user is assigned exactly one role. Roles are mapped to granular permissions through a many-to-many relationship.

### Role: `admin`

#### Typical Permissions:
- `user.view`, `user.edit`, `user.activate`
- `role.assign`, `permission.manage`
- `job.create`, `job.edit`, `job.delete`
- `company.create`, `company.edit`, `company.delete`
- `category.manage`
- `system.audit`

#### Description:
Full access to all resources and management tools. Responsible for user administration, data integrity, and application configuration.

### Role: `editor`

#### Typical Permissions:
- `job.create`, `job.edit`
- `company.create`, `company.edit`
- `category.view`
- `dashboard.access`

#### Description:
Can create and manage job listings and companies. Has limited access to categories and no rights to manage users or roles.

### Role: `viewer`

#### Typical Permissions:
- `job.view`
- `company.view`
- `category.view`
- `dashboard.access`

#### Description:
Read-only access to the system. Cannot modify or create content. Intended for external users, auditors, or restricted team members.

### Permission Assignment

Permissions are not hardcoded but stored in the `permissions` table. Roles are linked via the `role_to_permissions` table. This allows dynamic control and future extensibility.

### Auditability

All changes (create/update) across entities are tracked via `created_by` and `updated_by` fields, referencing the `users` table.
