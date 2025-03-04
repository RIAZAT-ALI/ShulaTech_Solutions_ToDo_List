# ShulaTech_Solutions_ToDo_List
This is my 1st task at intern ShulaTech


from tkinter import *
import sqlite3

# Personal Details
student_name = "Riazat Ali"
friends = ["Ahmed", "Bilal", "Hassan"]
university = "University of Sindh Jamshoro"
department = "Department of Data Science"

# Setup Tkinter Window
root = Tk()
root.title(f"{student_name}'s To-Do List")  # Personalized title
root.geometry('500x500')

# Database Connection
conn = sqlite3.connect('todo.db')
c = conn.cursor()

c.execute("""
    CREATE TABLE if not exists todo(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
        description TEXT NOT NULL,
        completed BOOLEAN NOT NULL
    );
""")
conn.commit()

# Function to Mark Task as Completed
def complete(id):
    def _complete():
        todo = c.execute("SELECT * from todo WHERE id = ?", (id, )).fetchone()
        c.execute("UPDATE todo SET completed = ? WHERE id = ?",(not todo[3], id))
        conn.commit()
        renderTodo()
    return _complete

# Function to Remove Task
def remove(id):
    def _remove():
        c.execute("DELETE FROM todo WHERE id = ?",(id, ))
        conn.commit()
        renderTodo()
    return _remove

# Function to Display Tasks
def renderTodo():
    rows = c.execute("SELECT * FROM todo").fetchall()
    for widget in frame.winfo_children():
        widget.destroy()

    for i in range(0, len(rows)):
        id = rows[i][0]
        completed = rows[i][3]
        description = rows[i][2]
        color = '#A0A0A0' if completed else '#000000'
        l = Checkbutton(frame, text= description, width=42, fg=color , anchor="w", command=complete(id))
        l.grid(row=i, column=0, sticky="w")
        l.select() if completed else l.deselect()
        btn =  Button(frame, text="Delete", command=remove(id))
        btn.grid(row=i, column=1)

# Function to Add Task
def addTodo():
    todo = e.get()
    if todo:
        c.execute("""
            INSERT INTO todo (description, completed) VALUES(?, ?)
                """, (todo, False))
        conn.commit()
        e.delete(0, END)
        renderTodo()

# GUI Elements
Label(root, text=f"{student_name} - {university} ({department})", font=("Arial", 10, "bold")).grid(row=0, column=0, columnspan=3, pady=10)

Label(root, text="Task").grid(row=1, column=0)

e = Entry(root, width=40)
e.grid(row=1, column=1)

btn = Button(root, text="Add", command= addTodo)
btn.grid(row=1, column=2)

frame = LabelFrame(root, text=f"{student_name}'s Tasks", padx=5, pady=5)
frame.grid(row=2, column=0, columnspan=3, sticky="nswe", padx=5)

e.focus()
root.bind('<Return>', lambda x: addTodo())

renderTodo()

root.mainloop()
