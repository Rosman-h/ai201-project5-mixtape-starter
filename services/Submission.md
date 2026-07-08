# Project 5 – Mixtape Bug Hunt Submission

## AI Usage

I used ChatGPT as a debugging and learning assistant throughout this project. I first used AI to understand the structure of the codebase by identifying the roles of the routes, services, and models and tracing how requests flowed through the application. During debugging, I asked AI to explain specific functions and discuss possible edge cases after I had already located the relevant code. I verified every suggested fix by reading the code myself, testing the application, and confirming the reported bug no longer occurred before committing the changes. AI also helped me organize my root cause analyses and improve the clarity of my documentation.

---

# Codebase Map

## Main Files

### app.py

Creates and configures the Flask application, initializes extensions such as SQLAlchemy, and registers the application's route blueprints.

### models.py

Defines the application's database models, including users, songs, playlists, notifications, listening history, and the relationships between them.

### routes/

Contains the API endpoints. The routes receive HTTP requests, validate input, call the appropriate service functions, and return JSON responses.

### services/

Contains the application's business logic.

- `playlist_service.py` manages playlist creation and retrieval.
- `notification_service.py` creates and retrieves user notifications.
- `search_service.py` performs song searching.
- `feed_service.py` builds activity feeds.
- `streak_service.py` manages listening streak calculations.

## Example Data Flow

When a user views a playlist:

1. The client sends a request to the playlist endpoint.
2. The route calls the appropriate function in `playlist_service.py`.
3. The service queries the database for the playlist's songs.
4. Each song is converted into a dictionary representation.
5. The route returns the JSON response to the client.

## Architecture Pattern

The project separates responsibilities into layers.

- Routes handle HTTP requests and responses.
- Services contain business logic.
- Models define the database schema and relationships.
- SQLAlchemy handles communication with the database.

This separation keeps routes lightweight while concentrating application logic inside the service layer.

---

# Root Cause Analysis

## Issue #5 – The last song in a playlist never shows up

### How you reproduced it

I opened a playlist, counted the songs returned, added another song, refreshed the playlist, and observed that the newest song never appeared even though the playlist count increased.

### How you found the root cause

I started at the playlist route and followed the request into `playlist_service.py`, where the function responsible for returning playlist songs converted the query results into the API response.

### Root cause

The service returned `songs[:-1]`, which excluded the final element of the list. Since newly added songs appear at the end of the playlist, the most recently added song was always omitted from the response.

### Fix and side-effect check

I removed the list slice so every song in the playlist is returned. After making the change, I verified that existing playlist functionality still worked and that newly added songs appeared correctly.

---

## Issue #4 – Rating notifications are never created

### How you reproduced it

I rated a song that had been shared by another user and then checked the original sharer's notification list. The rating was saved successfully, but no notification appeared.

### How you found the root cause

I compared the notification flow used when songs are added to playlists with the rating workflow inside `notification_service.py`. The playlist action created a notification, while the rating action did not.

### Root cause

The application successfully saved the rating but never created a notification for the song owner after the rating operation completed.

### Fix and side-effect check

I added the missing notification creation step after successfully saving the rating and verified that notifications were generated only when another user rated the song.

---

## Issue #3 – Duplicate songs appear in search

### How you reproduced it

I searched for songs that matched multiple related records and observed that identical songs appeared multiple times in the search results.

### How you found the root cause

I traced the search endpoint into `search_service.py` and examined the SQLAlchemy query responsible for returning matching songs.

### Root cause

The query performed joins that could return multiple rows for the same song, allowing duplicate songs to appear in the final result set.

### Fix and side-effect check

I modified the query so duplicate songs were removed before returning the results. I tested several searches to confirm that each matching song appeared only once while valid search results were still returned.
