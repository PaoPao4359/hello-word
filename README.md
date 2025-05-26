import tkinter as tk
from tkinter import messagebox
import json
import os
from datetime import datetime

# File to store tasks
TASKS_FILE = 'tasks.json'

# Load tasks from a JSON file
def load_tasks():
    if os.path.exists(TASKS_FILE):
        with open(TASKS_FILE, 'r') as file:
            return json.load(file)
    return []

# Save tasks to a JSON file
def save_tasks(tasks):
    with open(TASKS_FILE, 'w') as file:
        json.dump(tasks, file, indent=4)

# Add a task to the list
def add_task():
    task = task_entry.get()
    due_date = due_date_entry.get()
    if task.strip() == "":
        messagebox.showwarning("Warning", "Task cannot be empty!")
        return
    try:
        due_date_obj = datetime.strptime(due_date, "%Y-%m-%d") if due_date else None
    except ValueError:
        messagebox.showwarning("Warning", "Invalid date format! Use YYYY-MM-DD.")
        return
    tasks.append({"task": task, "completed": False, "due_date": due_date})
    task_entry.delete(0, tk.END)
    due_date_entry.delete(0, tk.END)
    update_task_list()

# Update the displayed task list
def update_task_list():
    task_list.delete(0, tk.END)
    for task in tasks:
        display_text = "[X] " + task["task"] if task["completed"] else task["task"]
        if task["due_date"]:
            display_text += f" (Due: {task['due_date']})"
        task_list.insert(tk.END, display_text)

# Toggle task completion
def toggle_task_completion():
    try:
        selected_index = task_list.curselection()[0]
        task = tasks[selected_index]
        task["completed"] = not task["completed"]
        update_task_list()
    except IndexError:
        messagebox.showwarning("Warning", "Select a task to mark as complete.")

# Delete a task from the list
def delete_task():
    try:
        selected_index = task_list.curselection()[0]
        del tasks[selected_index]
        update_task_list()
    except IndexError:
        messagebox.showwarning("Warning", "Select a task to delete.")

# Sort tasks by due date
def sort_tasks_by_due_date():
    tasks.sort(key=lambda x: x["due_date"] or "")
    update_task_list()

# Start drag-and-drop
def start_drag(event):
    global drag_start_index
    drag_start_index = task_list.nearest(event.y)

# Perform drag-and-drop
def perform_drag(event):
    global drag_start_index
    drag_end_index = task_list.nearest(event.y)
    if drag_start_index != drag_end_index:
        tasks.insert(drag_end_index, tasks.pop(drag_start_index))
        update_task_list()
        drag_start_index = drag_end_index

# Initialize the main application window
root = tk.Tk()
root.title("To-Do List Manager")

# Create GUI components
task_entry = tk.Entry(root, width=50)
task_entry.pack(pady=10)

due_date_entry = tk.Entry(root, width=50)
due_date_entry.pack(pady=5)
due_date_entry.insert(0, "Enter due date (YYYY-MM-DD)")

add_button = tk.Button(root, text="Add Task", command=add_task)
add_button.pack(pady=5)

toggle_button = tk.Button(root, text="Mark Complete", command=toggle_task_completion)
toggle_button.pack(pady=5)

delete_button = tk.Button(root, text="Delete Task", command=delete_task)
delete_button.pack(pady=5)

sort_button = tk.Button(root, text="Sort by Due Date", command=sort_tasks_by_due_date)
sort_button.pack(pady=5)

task_list = tk.Listbox(root, width=50, height=10)
task_list.pack(pady=10)

# Load existing tasks
tasks = load_tasks()
update_task_list()

# Save tasks on close
def on_closing():
    save_tasks(tasks)
    root.destroy()

root.protocol("WM_DELETE_WINDOW", on_closing)

# Bind drag-and-drop events
task_list.bind("<Button-1>", start_drag)
task_list.bind("<B1-Motion>", perform_drag)

# Start the GUI event loop
root.mainloop()
