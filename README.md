# 🧠 Habit Tracker

A desktop habit-tracking application built with **Python, PyQt5, SQLite, and Matplotlib**.

The application allows users to create and manage daily habits, track completion history, view statistics, and visualize weekly performance.

## ✨ Features

* ➕ Add new habits
* ✅ Mark habits as completed for the current day
* ↩️ Unmark completed habits
* ✏️ Edit existing habits
* 🗑️ Delete habits
* 📄 View habit completion history
* 📊 Visualize weekly habit performance
* 💾 Persistent data storage using SQLite
* 🖥️ Desktop graphical user interface built with PyQt5

## 🛠️ Technologies

* **Python 3**
* **PyQt5** — graphical user interface
* **SQLite** — local database management
* **Matplotlib** — weekly performance visualization

## 📁 Project Structure

```text
Habit-tracker/
│
├── habit_tracker.py      # Main application
├── requirements.txt      # Python dependencies
├── .gitignore             # Files excluded from Git
└── README.md              # Project documentation
```

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/MOHADESSEH/Habit-tracker.git
cd Habit-tracker
```

### 2. Install the dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the application

```bash
python habit_tracker.py
```

## 🗄️ Data Storage

The application uses **SQLite** to store habits and completion history.

The database file (`habits.db`) is generated automatically when the application is run.

The database file is intentionally excluded from the Git repository because it contains local application data.

## 📊 Weekly Performance

For each habit, the application can display a weekly performance chart showing the number of completed days over the previous seven days.

## 🎯 Purpose

This project was developed as a practical Python desktop application to demonstrate:

* Object-oriented programming
* GUI development
* SQLite database integration
* CRUD operations
* Event-driven programming
* Data visualization
* Basic software project organization

## 🚀 Future Improvements

Possible future improvements include:

* Habit categories
* Daily reminders and notifications
* Streak tracking
* Monthly and yearly statistics
* Progress dashboards
* Custom themes
* Exporting statistics
* More advanced data visualization

## 👩‍💻 Author

**Mohadesse Ramesh**

GitHub: [@MOHADESSEH](https://github.com/MOHADESSEH)
