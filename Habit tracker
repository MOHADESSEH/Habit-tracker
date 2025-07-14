import sys
import sqlite3
from datetime import date, timedelta
from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QHBoxLayout, QPushButton,
    QLabel, QLineEdit, QListWidget, QListWidgetItem, QMessageBox,
    QInputDialog, QDialog, QTableWidget, QTableWidgetItem
)
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QFont
import matplotlib.pyplot as plt


DB_FILE = "habits.db"


class HabitDatabase:
    def __init__(self):
        self.conn = sqlite3.connect(DB_FILE)
        self.create_tables()

    def create_tables(self):
        cursor = self.conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS habits (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL
            )
        """)
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS history (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                habit_id INTEGER,
                date TEXT,
                FOREIGN KEY(habit_id) REFERENCES habits(id)
            )
        """)
        self.conn.commit()

    def add_habit(self, name):
        self.conn.execute("INSERT INTO habits (name) VALUES (?)", (name,))
        self.conn.commit()

    def get_habits(self):
        return self.conn.execute("SELECT id, name FROM habits").fetchall()

    def mark_done(self, habit_id, day):
        if not self.is_done(habit_id, day):
            self.conn.execute("INSERT INTO history (habit_id, date) VALUES (?, ?)", (habit_id, day))
            self.conn.commit()

    def unmark_done(self, habit_id, day):
        self.conn.execute("DELETE FROM history WHERE habit_id = ? AND date = ?", (habit_id, day))
        self.conn.commit()

    def is_done(self, habit_id, day):
        result = self.conn.execute(
            "SELECT 1 FROM history WHERE habit_id = ? AND date = ?",
            (habit_id, day)
        ).fetchone()
        return bool(result)

    def get_stats(self, habit_id):
        return self.conn.execute(
            "SELECT COUNT(*) FROM history WHERE habit_id = ?",
            (habit_id,)
        ).fetchone()[0]

    def delete_habit(self, habit_id):
        self.conn.execute("DELETE FROM habits WHERE id = ?", (habit_id,))
        self.conn.execute("DELETE FROM history WHERE habit_id = ?", (habit_id,))
        self.conn.commit()

    def update_habit(self, habit_id, new_name):
        self.conn.execute("UPDATE habits SET name = ? WHERE id = ?", (new_name, habit_id))
        self.conn.commit()

    def get_history(self, habit_id):
        return self.conn.execute(
            "SELECT date FROM history WHERE habit_id = ? ORDER BY date DESC", (habit_id,)
        ).fetchall()

    def get_weekly_counts(self, habit_id):
        counts = []
        for i in range(6, -1, -1):  # last 7 days
            day = str(date.today() - timedelta(days=i))
            count = self.conn.execute(
                "SELECT COUNT(*) FROM history WHERE habit_id = ? AND date = ?", (habit_id, day)
            ).fetchone()[0]
            counts.append((day, count))
        return counts


class HabitDetailsDialog(QDialog):
    def __init__(self, db, habit_id, habit_name):
        super().__init__()
        self.setWindowTitle(f"Details - {habit_name}")
        self.setGeometry(300, 300, 500, 400)
        self.db = db
        self.habit_id = habit_id
        self.habit_name = habit_name

        layout = QVBoxLayout()
        self.setLayout(layout)

        # Table
        self.table = QTableWidget()
        layout.addWidget(QLabel("🗓 History"))
        layout.addWidget(self.table)
        self.populate_table()

        # Chart
        layout.addWidget(QLabel("📊 Weekly Performance"))
        chart_button = QPushButton("Show Chart")
        chart_button.clicked.connect(self.show_chart)
        layout.addWidget(chart_button)

    def populate_table(self):
        history = self.db.get_history(self.habit_id)
        self.table.setRowCount(len(history))
        self.table.setColumnCount(1)
        self.table.setHorizontalHeaderLabels(["Date Done"])
        for i, (day,) in enumerate(history):
            self.table.setItem(i, 0, QTableWidgetItem(day))

    def show_chart(self):
        data = self.db.get_weekly_counts(self.habit_id)
        dates = [d for d, _ in data]
        counts = [c for _, c in data]

        plt.bar(dates, counts, color='skyblue')
        plt.xticks(rotation=45)
        plt.title(f"Weekly Progress for {self.habit_name}")
        plt.xlabel("Date")
        plt.ylabel("Completed")
        plt.tight_layout()
        plt.show()


class HabitTracker(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("🧠 Habit Tracker (Advanced)")
        self.setGeometry(100, 100, 500, 600)
        self.db = HabitDatabase()
        self.today = str(date.today())

        self.layout = QVBoxLayout()
        self.setLayout(self.layout)

        self.habit_input = QLineEdit()
        self.habit_list_widget = QListWidget()

        self.init_ui()

    def init_ui(self):
        title = QLabel("🧠 Habit Tracker")
        title.setFont(QFont("Arial", 20, QFont.Bold))
        title.setAlignment(Qt.AlignCenter)
        self.layout.addWidget(title)

        input_layout = QHBoxLayout()
        self.habit_input.setPlaceholderText("Enter new habit")
        input_layout.addWidget(self.habit_input)

        add_button = QPushButton("➕ Add")
        add_button.clicked.connect(self.add_habit)
        input_layout.addWidget(add_button)
        self.layout.addLayout(input_layout)

        self.layout.addWidget(self.habit_list_widget)
        self.refresh_list()

    def add_habit(self):
        name = self.habit_input.text().strip()
        if not name:
            return
        self.db.add_habit(name)
        self.habit_input.clear()
        self.refresh_list()

    def refresh_list(self):
        self.habit_list_widget.clear()
        habits = self.db.get_habits()

        for habit_id, name in habits:
            is_done = self.db.is_done(habit_id, self.today)
            count = self.db.get_stats(habit_id)
            item_text = f"{name} — {'✅ Today' if is_done else '⬜ Not done'} | Total: {count}"

            item = QListWidgetItem(item_text)
            item.setData(Qt.UserRole, (habit_id, name))
            item.setFont(QFont("Arial", 12))

            color = Qt.darkGreen if is_done else Qt.darkRed
            item.setForeground(color)

            self.habit_list_widget.addItem(item)

        self.habit_list_widget.itemClicked.connect(self.toggle_done)
        self.habit_list_widget.setContextMenuPolicy(Qt.CustomContextMenu)
        self.habit_list_widget.customContextMenuRequested.connect(self.show_context_menu)

    def toggle_done(self, item):
        habit_id, _ = item.data(Qt.UserRole)
        if self.db.is_done(habit_id, self.today):
            self.db.unmark_done(habit_id, self.today)
        else:
            self.db.mark_done(habit_id, self.today)
        self.refresh_list()

    def show_context_menu(self, pos):
        item = self.habit_list_widget.itemAt(pos)
        if item is None:
            return

        habit_id, name = item.data(Qt.UserRole)
        menu = QMessageBox()
        menu.setText("⏬ Actions")
        menu.setInformativeText(f"What would you like to do with '{name}'?")
        details_btn = menu.addButton("📄 View Details", QMessageBox.ActionRole)
        edit_btn = menu.addButton("✏️ Edit", QMessageBox.ActionRole)
        delete_btn = menu.addButton("🗑 Delete", QMessageBox.DestructiveRole)
        cancel_btn = menu.addButton("Cancel", QMessageBox.RejectRole)
        menu.exec_()

        if menu.clickedButton() == details_btn:
            self.view_details(habit_id, name)
        elif menu.clickedButton() == delete_btn:
            self.db.delete_habit(habit_id)
            self.refresh_list()
        elif menu.clickedButton() == edit_btn:
            self.edit_habit(habit_id)

    def view_details(self, habit_id, name):
        dialog = HabitDetailsDialog(self.db, habit_id, name)
        dialog.exec_()

    def edit_habit(self, habit_id):
        new_name, ok = QInputDialog.getText(self, "Edit Habit", "Enter new name:")
        if ok and new_name.strip():
            self.db.update_habit(habit_id, new_name.strip())
            self.refresh_list()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = HabitTracker()
    window.show()
    sys.exit(app.exec_())
