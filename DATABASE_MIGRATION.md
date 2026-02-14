# Database Migration Documentation

## Overview
This migration replaces the in-memory data storage with a persistent SQLite database using SQLAlchemy ORM.

## Changes Made

### 1. New Dependencies
Added to `requirements.txt`:
- `sqlalchemy` - ORM for database operations
- `alembic` - Database migration tool
- `python-multipart` - For form data handling

### 2. New Files Created

#### `src/database.py`
- Database configuration and connection management
- Session factory for database operations
- Base class for all models
- `get_db()` dependency for FastAPI endpoints

#### `src/models.py`
- `Activity` model - stores activity information
- `Participant` model - stores student/participant information
- Many-to-many relationship through `activity_participants` association table

#### `src/init_db.py`
- Database initialization script
- Creates tables on first run
- Populates database with default activity data
- Idempotent - safe to run multiple times

### 3. Updated Files

#### `src/app.py`
- Removed in-memory `activities` dictionary
- Added database dependency injection to all endpoints
- Updated `/activities` endpoint to query from database
- Updated `/activities/{activity_name}/signup` to use database transactions
- Updated `/activities/{activity_name}/unregister` to use database transactions
- Added startup event to initialize database automatically

#### `.gitignore`
- Added `*.db` to exclude database files from version control

## Database Schema

### Activities Table
- `id` - Primary key
- `name` - Unique activity name (e.g., "Chess Club")
- `description` - Activity description
- `schedule` - Meeting schedule
- `max_participants` - Maximum number of participants

### Participants Table
- `id` - Primary key
- `email` - Unique student email

### Activity_Participants Table (Join Table)
- `activity_id` - Foreign key to activities
- `participant_id` - Foreign key to participants

## Migration Process

### Automatic Migration
The database is automatically initialized when the application starts:
1. On startup, the app calls `init_database()`
2. Tables are created if they don't exist
3. Default data is loaded if the database is empty

### Manual Migration (if needed)
```bash
cd src
python init_db.py
```

## Benefits

✅ **Persistent Storage** - Data survives application restarts
✅ **Data Integrity** - Database constraints prevent invalid data
✅ **Concurrent Access** - Multiple users can access safely
✅ **Scalability** - Easy to switch to PostgreSQL/MySQL for production
✅ **Relationships** - Proper modeling of student-activity relationships
✅ **Backup Ready** - Simple file-based backups (SQLite) or standard DB backups

## Testing

To test the migration:

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Start the application:
   ```bash
   cd src
   uvicorn app:app --reload
   ```

3. Verify functionality:
   - Visit http://localhost:8000
   - Check that activities are displayed
   - Test signup/unregister functionality
   - Restart the app and verify data persists

## Rollback Plan

If issues occur, rollback is straightforward:
1. Switch back to the `main` branch
2. Delete the `mergington_school.db` file
3. The application will use in-memory storage again

## Future Enhancements

- [ ] Add Alembic migrations for schema changes
- [ ] Add database connection pooling for production
- [ ] Add backup/restore scripts
- [ ] Switch to PostgreSQL for production deployment
- [ ] Add database indexes for query optimization
