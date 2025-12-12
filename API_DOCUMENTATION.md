# API Documentation

## Base URL
```
https://api.peer-learning.com/api
```

## Authentication

All endpoints require JWT token in Authorization header:
```
Authorization: Bearer <token>
```

## Endpoints

### Sessions

#### GET /sessions
Get all sessions

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "_id": "123",
      "title": "Physics",
      "description": "Learn relativity",
      "instructor": "user_id",
      "participants": [],
      "createdAt": "2024-12-01T10:00:00Z"
    }
  ]
}
```

#### POST /sessions
Create new session

**Request Body:**
```json
{
  "title": "Physics",
  "description": "Learn relativity",
  "maxParticipants": 10
}
```

### Quiz

#### POST /quiz/generate
Generate AI-powered quiz

**Request Body:**
```json
{
  "topic": "Physics - Relativity",
  "difficulty": "intermediate",
  "numQuestions": 5
}
```

#### GET /quiz/:id
Get quiz details

## Error Responses

**401 Unauthorized:**
```json
{
  "success": false,
  "error": "Invalid token"
}
```

**400 Bad Request:**
```json
{
  "success": false,
  "error": "Missing required fields"
}
```

## Rate Limiting

- 100 requests per minute for authenticated users
- 10 requests per minute for anonymous users
