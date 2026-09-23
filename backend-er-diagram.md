# VoltGrid Backend Entity Relationship Diagram

This ER diagram shows the recommended relational database structure for the VoltGrid Java backend.

```mermaid
erDiagram
    USERS {
        BIGINT id PK
        VARCHAR name
        VARCHAR email UK
        VARCHAR password_hash
        VARCHAR status
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    ROLES {
        BIGINT id PK
        VARCHAR name UK
    }

    USER_ROLES {
        BIGINT user_id PK, FK
        BIGINT role_id PK, FK
    }

    VEHICLES {
        BIGINT id PK
        BIGINT user_id FK
        VARCHAR make
        VARCHAR model
        VARCHAR connector_type
        DECIMAL battery_capacity_kwh
        DECIMAL max_charging_speed_kw
        INT current_battery_percent
        BOOLEAN is_default
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    STATIONS {
        BIGINT id PK
        VARCHAR name
        VARCHAR operator_name
        VARCHAR address
        DECIMAL latitude
        DECIMAL longitude
        DECIMAL price_per_kwh
        DECIMAL rating
        BOOLEAN active
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    CHARGERS {
        BIGINT id PK
        BIGINT station_id FK
        VARCHAR connector_type
        VARCHAR charger_type
        DECIMAL power_kw
        VARCHAR status
        INT queue_length
        TIMESTAMP updated_at
    }

    FAVORITE_STATIONS {
        BIGINT id PK
        BIGINT user_id FK
        BIGINT station_id FK
        TIMESTAMP created_at
    }

    STATION_REVIEWS {
        BIGINT id PK
        BIGINT user_id FK
        BIGINT station_id FK
        INT rating
        TEXT comment
        VARCHAR status
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    RECOMMENDATION_PREFERENCES {
        BIGINT id PK
        BIGINT user_id FK, UK
        INT distance_weight
        INT price_weight
        INT speed_weight
        INT rating_weight
        INT queue_weight
        TIMESTAMP updated_at
    }

    CHARGING_SESSIONS {
        BIGINT id PK
        BIGINT user_id FK
        BIGINT vehicle_id FK
        BIGINT station_id FK
        BIGINT charger_id FK
        VARCHAR status
        INT start_battery_percent
        INT target_battery_percent
        INT current_battery_percent
        DECIMAL energy_kwh
        DECIMAL price_per_kwh
        DECIMAL total_cost
        INT duration_minutes
        TIMESTAMP started_at
        TIMESTAMP completed_at
        TIMESTAMP created_at
    }

    PAYMENTS {
        BIGINT id PK
        BIGINT user_id FK
        BIGINT charging_session_id FK, UK
        DECIMAL amount
        VARCHAR provider_reference UK
        VARCHAR status
        TIMESTAMP paid_at
        TIMESTAMP created_at
    }

    PAYMENT_METHODS {
        BIGINT id PK
        BIGINT user_id FK
        VARCHAR method_type
        VARCHAR provider_token
        BOOLEAN is_default
        TIMESTAMP created_at
    }

    SHOP_ITEMS {
        BIGINT id PK
        VARCHAR name
        TEXT description
        VARCHAR amazon_query
        VARCHAR external_url
        BOOLEAN active
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    NOTIFICATIONS {
        BIGINT id PK
        BIGINT user_id FK
        VARCHAR title
        TEXT message
        VARCHAR notification_type
        BOOLEAN is_read
        TIMESTAMP created_at
    }

    USERS ||--o{ USER_ROLES : has
    ROLES ||--o{ USER_ROLES : assigned_to
    USERS ||--o{ VEHICLES : owns
    STATIONS ||--o{ CHARGERS : contains
    USERS ||--o{ FAVORITE_STATIONS : saves
    STATIONS ||--o{ FAVORITE_STATIONS : favorited_as
    USERS ||--o{ STATION_REVIEWS : writes
    STATIONS ||--o{ STATION_REVIEWS : receives
    USERS ||--o| RECOMMENDATION_PREFERENCES : configures
    USERS ||--o{ CHARGING_SESSIONS : starts
    VEHICLES ||--o{ CHARGING_SESSIONS : used_for
    STATIONS ||--o{ CHARGING_SESSIONS : hosts
    CHARGERS ||--o{ CHARGING_SESSIONS : used_by
    USERS ||--o{ PAYMENTS : makes
    CHARGING_SESSIONS ||--o| PAYMENTS : generates
    USERS ||--o{ PAYMENT_METHODS : owns
    USERS ||--o{ NOTIFICATIONS : receives
```

## Relationship summary

| Relationship | Meaning |
|---|---|
| `USERS` to `VEHICLES` | One user can register multiple vehicles. |
| `STATIONS` to `CHARGERS` | One station can contain multiple chargers. |
| `USERS` to `STATIONS` | Users save stations through `FAVORITE_STATIONS`. |
| `USERS` to `STATIONS` | Users review stations through `STATION_REVIEWS`. |
| `USERS` to `CHARGING_SESSIONS` | A user can have multiple charging sessions. |
| `CHARGING_SESSIONS` to `PAYMENTS` | A completed charging session can generate one payment. |
| `USERS` to `ROLES` | Users and roles use the `USER_ROLES` many-to-many join table. |
| `USERS` to `RECOMMENDATION_PREFERENCES` | Each user can have one recommendation preference record. |

## Important database constraints

- `users.email` must be unique.
- `roles.name` must be unique.
- `user_roles(user_id, role_id)` must be a composite primary key.
- `favorite_stations(user_id, station_id)` should be unique to prevent duplicate favorites.
- `recommendation_preferences.user_id` must be unique.
- `payments.charging_session_id` should be unique when one session produces one payment.
- Battery percentages must be between `0` and `100`.
- Station latitude must be between `-90` and `90`.
- Station longitude must be between `-180` and `180`.
- Rating must be between `1` and `5`.
- Charger power, battery capacity, energy, price and payment amount must not be negative.
