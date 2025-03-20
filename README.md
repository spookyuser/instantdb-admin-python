# InstantDB Admin Python Client

A Python client for the [InstantDB](https://www.instantdb.com/) Admin API, providing functionality for data operations, user management, and file storage.

InstantDB is a realtime database that makes it easy to build collaborative apps. This client provides Python developers with access to InstantDB's admin capabilities.

This client is built based on the [unofficial InstantDB Admin HTTP API](https://www.dropbox.com/scl/fi/2yjy6xvqa0459hqeqg950/Unofficial-Admin-HTTP-API.paper?dl=0) and is adapted from the official TypeScript InstantDB admin client.

## Installation

This client is designed to be a single, self-contained file that you can drop directly into your Python project:

1. Download `instantdb_admin_client.py` from this repository
2. Copy it into your project directory
3. Import it in your code

The only dependencies are:
- Python 3.7+
- `aiohttp`
- `typing_extensions`

## Authentication

To use the client, you need your app's `APP_ID` and `ADMIN_TOKEN`, which you can get from your InstantDB dashboard.

```python
from instantdb_admin_client import InstantDBAdminAPI

# Initialize the client
db = InstantDBAdminAPI(
    app_id="your-app-id",
    admin_token="your-admin-token"
)
```

## Making Queries

Query your data using InstantDB's InstaQL query language:

```python
# Fetch all goals
result = await db.query({"goals": {}})

# Goals where title is "Get Fit"
result = await db.query({
    "goals": {
        "$": {
            "where": {"title": "Get Fit"}
        }
    }
})

# All goals alongside their todos
result = await db.query({"goals": {"todos": {}}})
```

### Real-world Example: Querying a User's Profile

Here's a real-world example of querying a user's profile from associated user data:

```python
async def get_user_profile_id(user_id: str) -> str:
    """Get the profile ID for the user."""
    try:
        # Query the user's profile ID
        result = await db.query({
            "$users": {
                "$": {
                    "where": {"id": user_id},
                },
                "profile": {},
            },
        })

        user_profile_id = result.get("$users", [{}])[0].get("profile", {})[0].get("id")
        if not user_profile_id:
            raise ValueError(f"No profile found for user {user_id}")

        return user_profile_id
    except Exception as e:
        raise
```

## Making Transactions

Perform database operations using type-safe transaction steps:

```python
from instantdb_admin_client import Update, Link, Delete, Unlink

# Create and link objects
await db.transact([
    Update(
        collection="todos",
        id="todo-123",
        data={"title": "Go running"}
    ),
    Link(
        collection="goals",
        id="goal-123",
        links={"todos": "todo-123"}
    )
])

# Delete an object
await db.transact([
    Delete(
        collection="goals",
        id="goal-123"
    )
])
```

## Impersonating Users

Make requests on behalf of specific users:

```python
# As a specific user by email
user_client = db.as_user(email="user@example.com")
result = await user_client.query({"goals": {}})

# As a user with a token
user_client = db.as_user(token="user-refresh-token")
result = await user_client.query({"goals": {}})

# As a guest (unauthenticated) user
guest_client = db.as_user(guest=True)
result = await guest_client.query({"goals": {}})
```

## User Management

```python
# Create a new user and get a refresh token
token = await db.auth.create_token("new-user@example.com")

# Verify a token
user = await db.auth.verify_token("user-refresh-token")

# Get user information
user = await db.auth.get_user(email="user@example.com")

# Delete a user
deleted_user = await db.auth.delete_user(email="user@example.com")

# Sign out a user (invalidate their tokens)
await db.auth.sign_out("user@example.com")
```

## File Storage

```python
# Upload a file
with open("photo.jpg", "rb") as f:
    result = await db.storage.uploadFile(
        "photos/profile.jpg",
        f,
        {"contentType": "image/jpeg"}
    )

# Get a download URL for a file
url = await db.storage.get_download_url("photos/profile.jpg")

# List all files in storage
files = await db.storage.list()
for file in files:
    print(f"File: {file.name}, Size: {file.size}")

# Delete a file
await db.storage.delete("photos/old-profile.jpg")

# Delete multiple files
await db.storage.delete_many([
    "photos/1.jpg",
    "photos/2.jpg",
    "photos/3.jpg"
])
```

## Debug Tools

Inspect permission checks for queries and transactions:

```python
# Debug a query
result = await db.as_user(guest=True).debug_query(
    {"goals": {}},
    rules={"goals": {"allow": {"read": "auth.id != null"}}}
)

# Debug a transaction
result = await db.as_user(guest=True).debug_transact([
    ["update", "goals", "goal-123", {"title": "Get fit"}]
])
```

## License

MIT