# To-do-app
import tkinter as tk
from tkinter import messagebox
import sqlite3

conn = sqlite3.connect('todo.db')
cursor = conn.cursor()
cursor.execute('''
    CREATE TABLE IF NOT EXISTS tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        description TEXT
    )
''')
conn.commit()

def add_task():
    title = title_entry.get()
    if not title:
        messagebox.showwarning('Error', "Title cannot be empty!")
        return
    cursor.execute("INSERT INTO tasks (title, description) VALUES (?, ?)", (title, description_entry.get()))
    conn.commit()
    refresh_tasks()
    title_entry.delete(0, tk.END)
    description_entry.delete(0, tk.END)

def update_task():
    try:
        task_id = int(task_list.get(task_list.curselection()).split(":")[0])
        cursor.execute("UPDATE tasks SET title = ?, description = ? WHERE id = ?", 
                       (title_entry.get(), description_entry.get(), task_id))
        conn.commit()
        refresh_tasks()
        title_entry.delete(0, tk.END)
        description_entry.delete(0, tk.END)
    except:
        messagebox.showwarning("Error", "Select a task to update!")

def delete_task():
    try:
        task_id = int(task_list.get(task_list.curselection()).split(":")[0])
        cursor.execute("DELETE FROM tasks WHERE id = ?", (task_id,))
        conn.commit()
        refresh_tasks()
    except:
        messagebox.showwarning("Error", "Select a task to delete!")

def refresh_tasks():
    task_list.delete(0, tk.END)
    for row in cursor.execute("SELECT * FROM tasks"):
        task_list.insert(tk.END, f"{row[0]}: {row[1]} - {row[2]}")

def select_task(event):
    try:
        task_id = int(task_list.get(task_list.curselection()).split(":")[0])
        task = cursor.execute("SELECT * FROM tasks WHERE id = ?", (task_id,)).fetchone()
        title_entry.delete(0, tk.END)
        title_entry.insert(0, task[1])
        description_entry.delete(0, tk.END)
        description_entry.insert(0, task[2])
    except:
        pass

root = tk.Tk()
root.title("To-Do Application")

tk.Label(root, text="Title:").grid(row=0, column=0)
title_entry = tk.Entry(root, width=30)
title_entry.grid(row=0, column=1)

tk.Label(root, text="Description:").grid(row=1, column=0)
description_entry = tk.Entry(root, width=30)
description_entry.grid(row=1, column=1)

tk.Button(root, text="Add Task", command=add_task).grid(row=2, column=0)
tk.Button(root, text="Update Task", command=update_task).grid(row=2, column=1)
tk.Button(root, text="Delete Task", command=delete_task).grid(row=3, column=0)

task_list = tk.Listbox(root, width=50, height=15)
task_list.grid(row=4, column=0, columnspan=2)
task_list.bind('<<ListboxSelect>>', select_task)

refresh_tasks()
root.mainloop()
conn.close()