# {{FEATURE_NAME}} — data model

Source of truth for columns: `laravel-pos-api-dev_guid/db_script/tables/tables.sql` and `database/migrations/`. This file must match them.

## Tables

| Table | Primary key | Purpose | Created / altered / relied on |
|---|---|---|---|
| <!-- `zones` --> | <!-- `zoneId` binary(16) --> | <!-- … --> | <!-- relied on --> |

## Columns of new or altered tables

| Column | Type | Null | Default | Rule |
|---|---|---|---|---|
| <!-- --> | | | | |

## Rules the service enforces

- Every read and write filters on `restaurantId` = the tenant from the token.
- <!-- uniqueness (e.g. name unique per restaurant among non-deleted rows), foreign keys that must belong to the same restaurant, status transitions -->

## Row lifecycle

<!-- isActive / isDelete meaning for this table; what other screens hide; cascade effects (e.g. switching a zone off switches its tables off). -->

## Status values

<!-- Column → value → meaning, if the feature uses a status column. -->

## Migrations and scripts

<!-- Migration file names added or changed, and the matching db_script change. Re-runnable. -->

## Indexes that matter

<!-- -->

## Seed data

<!-- or "None." -->
