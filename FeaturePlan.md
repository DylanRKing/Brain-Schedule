# .NET MAUI To-Do App: Feature and Architecture Plan

This document outlines the plan for building a cross-platform to-do application using .NET MAUI, covering key features like data synchronization, recurring reminders, sub-tasks, and categorization.

### High-Level Plan & Architecture

We will follow the **Model-View-ViewModel (MVVM)** pattern, which is the standard for .NET MAUI. This separates the application into three logical layers:

1.  **Model:** The data structures for your app (e.g., `ToDoTask`, `Category`).
2.  **View:** The UI the user sees (the XAML pages).
3.  **ViewModel:** The logic that connects the Model to the View, preparing data for display and handling user interactions.

**Recommended Project Structure:**

*   **`BrainSchedule.GUI` (Existing Project):** Contains the Views and ViewModels.
*   **`BrainSchedule.Core` (New Class Library):** Will contain the Models, business logic, and service interfaces. This keeps core logic separate from the UI.
*   **`BrainSchedule.Api` (New ASP.NET Core Web API):** The backend server to handle data storage and synchronization.

---

### 1. Data Models (The "Model" in MVVM)

These classes will be created in the `BrainSchedule.Core` project.

*   **`Category.cs`**: Represents a task category.
    ```csharp
    public class Category
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }
    ```

*   **`SubTask.cs`**: Represents a single sub-task.
    ```csharp
    public class SubTask
    {
        public Guid Id { get; set; }
        public string Title { get; set; }
        public bool IsCompleted { get; set; }
    }
    ```

*   **`ToDoTask.cs`**: The main task object.
    ```csharp
    public enum RecurrenceType { None, Daily, Weekly, Monthly, Custom }

    public class ToDoTask
    {
        public Guid Id { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        public DateTime? DueDate { get; set; }
        public Guid CategoryId { get; set; }

        // For reminders
        public DateTime? ReminderTime { get; set; }
        public RecurrenceType Recurrence { get; set; }

        // For sub-tasks
        public List<SubTask> SubTasks { get; set; } = new();

        // Business logic: A task is only complete if all its sub-tasks are complete.
        public bool IsCompleted => SubTasks.Any() && SubTasks.All(s => s.IsCompleted);
    }
    ```

---

### 2. Syncing Tasks Across Devices

This requires a central backend service.

*   **Backend:** Create an **ASP.NET Core Web API** project.
    *   Use **Entity Framework Core** to connect to a database (e.g., SQL Server, PostgreSQL).
    *   Create API endpoints (e.g., `GET /tasks`, `POST /tasks`) for CRUD operations.
*   **MAUI App (Client-side):**
    *   Use `HttpClient` to make requests to your Web API.
    *   **Offline Support:** Use a local **SQLite** database (`sqlite-net-pcl` NuGet package) for offline access.
    *   **Sync Logic:** The app will save changes locally first, then periodically push local changes to the backend and fetch remote changes from it.

---

### 3. Recurring Reminders

Schedule notifications with the operating system so they fire even when the app is closed.

*   **Implementation:** Use a cross-platform notification library like **`Plugin.LocalNotification`** (NuGet package).
*   **How it works:**
    1.  Initialize the plugin and request notification permissions on startup.
    2.  When a task with a reminder is created, schedule a `NotificationRequest` with the plugin.
    3.  Use the `RepeatType` property for simple recurring schedules (Daily, Weekly).
    4.  For custom schedules, schedule the next occurrence; when it fires, the app's handler will calculate and schedule the *next* one after that.

---

### 4. Sub-Tasks

*   **Data Model:** The `ToDoTask` model contains a `List<SubTask>`.
*   **Business Logic:** The `ToDoTask.IsCompleted` property is a read-only calculated property that checks if all sub-tasks are completed. This enforces the rule automatically.
*   **UI (View/ViewModel):**
    *   Display sub-tasks in a `CollectionView` with `CheckBoxes` bound to `SubTask.IsCompleted`.
    *   The parent task's completion status will update automatically in the UI.

---

### 5. Task Categorization

*   **Data Model:** `ToDoTask` has a `CategoryId`.
*   **UI (View/ViewModel):**
    *   **Creating/Editing:** Use a `Picker` control to select a category for a task.
    *   **Organizing:** Use a `CollectionView` with `IsGrouped="True"` on the main page. The ViewModel will be responsible for grouping the tasks by category before displaying them.
