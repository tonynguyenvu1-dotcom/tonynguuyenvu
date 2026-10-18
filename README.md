# NIT3213 Final Assignment — Android Application

An Android application developed for the **NIT3213 Mobile Application Development** final assignment.

The application connects to the NIT3213 REST API and provides a complete authentication and data-display workflow. Users can log in using their student details, retrieve their assigned dashboard entities, and view detailed information for each entity.

---

## Features

### Login & Authentication

- Student ID and student name input fields.
- Client-side validation to ensure required fields are entered.
- Authentication using the NIT3213 REST API.
- Receives a `keypass` from the authentication endpoint after successful login.
- Displays user-friendly error messages when authentication fails.

### Dashboard

- Retrieves entities from the API using the authenticated `keypass`.
- Displays entities using a `RecyclerView`.
- Provides loading, success, empty and error states.
- Allows the user to select an entity to view more information.

### Entity Details

- Displays the complete information for a selected entity.
- Dynamically displays entity properties returned by the API.
- Includes the required `description` information.
- Provides navigation back to the dashboard.

### Network & Error Handling

- Uses Retrofit and OkHttp for API communication.
- Uses extended network timeouts to accommodate Render server cold starts.
- Retries requests when a socket timeout occurs.
- Converts common network and HTTP errors into understandable messages for the user.

---

## Application Flow

The application follows this general flow:

```text
Login
  ↓
POST /footscray/auth
  ↓
keypass returned
  ↓
Dashboard
  ↓
GET /dashboard/{keypass}
  ↓
Entities returned
  ↓
RecyclerView
  ↓
User selects entity
  ↓
Details screen
```

---

## Architecture

The application uses the **MVVM (Model-View-ViewModel)** architecture together with the Repository Pattern.

```text
Fragment
   ↓
ViewModel
   ↓
Repository
   ↓
ApiService
   ↓
REST API
```

### View

The Fragments are responsible for displaying the user interface and responding to user interactions.

Main UI components include:

- `LoginFragment`
- `DashboardFragment`
- `DetailsFragment`

### ViewModel

The ViewModels contain UI-related application logic and expose the current state to the Fragments.

Main ViewModels include:

- `LoginViewModel`
- `DashboardViewModel`

### Repository

`Repository.kt` provides a central location for data operations.

It:

- Communicates with the API through `ApiService`.
- Handles API calls.
- Handles network errors.
- Converts technical exceptions into user-friendly messages.
- Implements the retry behaviour used for API cold starts.

### API Service

`ApiService.kt` defines the REST API endpoints used by Retrofit.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Kotlin | Main programming language |
| Android SDK | Android application development |
| MVVM | Application architecture |
| Repository Pattern | Separation of data operations |
| Hilt | Dependency injection |
| Retrofit | REST API communication |
| OkHttp | HTTP client and networking |
| Moshi | JSON serialisation/deserialisation |
| Kotlin Coroutines | Asynchronous operations |
| StateFlow | UI state management |
| Navigation Component | Navigation between screens |
| SafeArgs | Type-safe navigation arguments |
| RecyclerView | Displaying dashboard entities |
| View Binding | Accessing XML views safely |
| Material Components | User interface components |
| JUnit | Unit testing |
| MockK | Mocking dependencies in tests |
| MockWebServer | Testing network requests |

---

## API

### Base URL

```text
https://nit3213apinew.onrender.com/
```

### Authentication

**Method:**

```text
POST
```

**Endpoint:**

```text
/footscray/auth
```

**Request:**

```json
{
  "username": "YourStudentID",
  "password": "YourFirstName"
}
```

**Successful response:**

```json
{
  "keypass": "topicName"
}
```

The returned `keypass` is then used to retrieve the dashboard data.

### Dashboard

**Method:**

```text
GET
```

**Endpoint:**

```text
/dashboard/{keypass}
```

**Example response:**

```json
{
  "entities": [
    {
      "property1": "value1",
      "property2": "value2",
      "description": "Detailed description"
    }
  ],
  "entityTotal": 7
}
```

---

## Project Structure

```text
app/src/main/java/com/example/myapplication
│
├── App.kt
│
├── data/
│   ├── Repository.kt
│   └── remote/
│       └── ApiService.kt
│
├── di/
│   └── NetworkModule.kt
│
├── model/
│   ├── LoginRequest.kt
│   ├── LoginResponse.kt
│   └── DashboardResponse.kt
│
├── ui/
│   ├── MainActivity.kt
│   │
│   ├── login/
│   │   ├── LoginFragment.kt
│   │   └── LoginViewModel.kt
│   │
│   ├── dashboard/
│   │   ├── DashboardFragment.kt
│   │   ├── DashboardViewModel.kt
│   │   └── EntityAdapter.kt
│   │
│   └── details/
│       └── DetailsFragment.kt
│
└── util/
    └── UiState.kt
```

---

## UI State Management

The application uses a sealed `UiState` class to represent the current state of API operations.

The possible states are:

```text
Idle
Loading
Success
Error
```

This allows the UI to respond appropriately depending on what is happening.

For example:

```text
User presses Login
       ↓
Loading
       ↓
   API request
       ↓
 ┌─────┴─────┐
 ↓           ↓
Success     Error
 ↓           ↓
Dashboard   Error message
```

---

## Dependency Injection

The application uses **Hilt** for dependency injection.

Hilt provides the application's networking dependencies, including:

- `OkHttpClient`
- `Moshi`
- `Retrofit`
- `ApiService`
- `Repository`
- ViewModels

This reduces manual object creation and keeps the application components loosely coupled.

---

## Networking

Retrofit is used to define the API endpoints.

The authentication request is defined as:

```kotlin
@POST("footscray/auth")
suspend fun login(@Body body: LoginRequest): LoginResponse
```

The dashboard request is defined as:

```kotlin
@GET("dashboard/{keypass}")
suspend fun getDashboard(
    @Path("keypass") keypass: String
): DashboardResponse
```

OkHttp is used underneath Retrofit for HTTP communication and timeout configuration.

Moshi is used to convert JSON responses into Kotlin objects.

---

## Asynchronous Programming

Kotlin Coroutines are used for network operations so that API requests do not block the main UI thread.

The ViewModels use `viewModelScope` to launch asynchronous operations.

`StateFlow` is used to expose state changes from the ViewModels to the Fragments.

---

## Navigation

The application uses the Android Jetpack Navigation Component.

The `keypass` returned after authentication is passed from the Login screen to the Dashboard using **SafeArgs**.

The selected entity is passed from the Dashboard to the Details screen.

---

## RecyclerView

The Dashboard uses a `RecyclerView` to display the entities returned by the API.

`EntityAdapter` is responsible for:

- Creating list item views.
- Binding entity information to each item.
- Handling item selection.
- Updating the displayed list efficiently.

---

## Dynamic Entity Details

The API entities can contain different properties.

For this reason, the application uses a dynamic structure rather than relying entirely on a fixed set of properties.

When an entity is selected:

```text
Entity Map
   ↓
JSON
   ↓
Navigation
   ↓
DetailsFragment
   ↓
JSON converted back to Map
   ↓
Dynamic UI
```

This allows the Details screen to display the properties returned by the API without requiring a separate model for every possible entity structure.

---

## Error Handling

The Repository converts common technical errors into user-friendly messages.

Examples include:

- Invalid authentication details.
- No internet connection.
- Connection timeout.
- Server unavailable.
- HTTP 401 errors.
- HTTP 403 errors.
- HTTP 404 errors.
- HTTP 500-series server errors.

The application also handles the Render server's cold-start behaviour by using extended network timeouts and retrying a request after a socket timeout.

---

## Testing

The project includes unit tests for the ViewModels and Repository.

### LoginViewModelTest

Tests:

- Blank Student ID validation.
- Blank Student Name validation.
- Successful login behaviour.
- Correct UI state changes.

### DashboardViewModelTest

Tests:

- Blank keypass validation.
- Successful dashboard loading.
- Correct UI state changes.

### RepositoryTest

Tests:

- API communication.
- Endpoint behaviour.
- HTTP error handling.
- Conversion of technical errors into user-friendly messages.

### Running Tests

From Android Studio:

```text
Right-click app/src/test
→ Run Tests
```

Or from the command line:

```bash
./gradlew test
```

---

## Requirements

To build and run the application:

- Android Studio
- JDK 17
- Android SDK 35
- Android device or emulator
- Android API 24 or higher
- Internet connection

---

## How to Run

1. Open the project in Android Studio.
2. Allow Gradle to synchronise.
3. Connect an Android device or start an emulator.
4. Run the application.
5. Enter the required Student ID.
6. Enter the required first name.
7. Press **Sign In**.
8. The application authenticates with the API.
9. The dashboard entities are retrieved.
10. Select an entity to view its details.

---

## Assignment Technologies

This project demonstrates the following Android development concepts:

- Kotlin
- Android Fragments
- MVVM architecture
- Repository Pattern
- Dependency Injection
- Hilt
- REST APIs
- Retrofit
- OkHttp
- Moshi
- Coroutines
- StateFlow
- Navigation Component
- SafeArgs
- RecyclerView
- View Binding
- Error handling
- Unit testing

---

## Tony Nguyen

**NIT3213 Mobile Application Development**

Final Assignment
