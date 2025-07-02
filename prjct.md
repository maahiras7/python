import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3
from datetime import datetime

class StudentManagementSystem:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Management System")
        self.root.geometry("800x600")
        
        # Database setup
        self.conn = sqlite3.connect("students.db")
        self.cursor = self.conn.cursor()
        self.create_table()
        
        # GUI Setup
        self.setup_gui()
        
    def create_table(self):
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS students (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                roll_no TEXT NOT NULL UNIQUE,
                email TEXT,
                phone TEXT,
                department TEXT,
                year INTEGER,
                created_at TEXT
            )
        ''')
        self.conn.commit()
    
    def setup_gui(self):
        # Title Label
        title_label = tk.Label(self.root, text="Student Management System", font=("Arial", 16, "bold"))
        title_label.pack(pady=10)
        
        # Input Frame
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10)
        
        # Input Fields
        tk.Label(input_frame, text="Name:").grid(row=0, column=0, padx=5, pady=5)
        self.name_entry = tk.Entry(input_frame, width=30)
        self.name_entry.grid(row=0, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Roll No:").grid(row=1, column=0, padx=5, pady=5)
        self.roll_entry = tk.Entry(input_frame, width=30)
        self.roll_entry.grid(row=1, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Email:").grid(row=2, column=0, padx=5, pady=5)
        self.email_entry = tk.Entry(input_frame, width=30)
        self.email_entry.grid(row=2, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Phone:").grid(row=3, column=0, padx=5, pady=5)
        self.phone_entry = tk.Entry(input_frame, width=30)
        self.phone_entry.grid(row=3, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Department:").grid(row=4, column=0, padx=5, pady=5)
        self.dept_entry = tk.Entry(input_frame, width=30)
        self.dept_entry.grid(row=4, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Year:").grid(row=5, column=0, padx=5, pady=5)
        self.year_entry = tk.Entry(input_frame, width=30)
        self.year_entry.grid(row=5, column=1, padx=5, pady=5)
        
        # Buttons
        button_frame = tk.Frame(self.root)
        button_frame.pack(pady=10)
        
        tk.Button(button_frame, text="Add Student", command=self.add_student).grid(row=0, column=0, padx=5)
        tk.Button(button_frame, text="Update Student", command=self.update_student).grid(row=0, column=1, padx=5)
        tk.Button(button_frame, text="Delete Student", command=self.delete_student).grid(row=0, column=2, padx=5)
        tk.Button(button_frame, text="Clear Fields", command=self.clear_fields).grid(row=0, column=3, padx=5)
        
        # Treeview for displaying students
        self.tree = ttk.Treeview(self.root, columns=("ID", "Name", "Roll No", "Email", "Phone", "Dept", "Year", "Created"), show="headings")
        self.tree.heading("ID", text="ID")
        self.tree.heading("Name", text="Name")
        self.tree.heading("Roll No", text="Roll No")
        self.tree.heading("Email", text="Email")
        self.tree.heading("Phone", text="Phone")
        self.tree.heading("Dept", text="Department")
        self.tree.heading("Year", text="Year")
        self.tree.heading("Created", text="Created At")
        
        self.tree.column("ID", width=30)
        self.tree.column("Name", width=100)
        self.tree.column("Roll No", width=80)
        self.tree.column("Email", width=120)
        self.tree.column("Phone", width=100)
        self.tree.column("Dept", width=100)
        self.tree.column("Year", width=50)
        self.tree.column("Created", width=120)
        
        self.tree.pack(pady=10, fill="both", expand=True)
        self.tree.bind('<<TreeviewSelect>>', self.on_tree_select)
        
        self.refresh_treeview()
    
    def add_student(self):
        name = self.name_entry.get()
        roll_no = self.roll_entry.get()
        email = self.email_entry.get()
        phone = self.phone_entry.get()
        dept = self.dept_entry.get()
        year = self.year_entry.get()
        
        if not name or not roll_no:
            messagebox.showerror("Error", "Name and Roll No are required!")
            return
            
        try:
            year = int(year) if year else 0
            created_at = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            
            self.cursor.execute('''
                INSERT INTO students (name, roll_no, email, phone, department, year, created_at)
                VALUES (?, ?, ?, ?, ?, ?, ?)
            ''', (name, roll_no, email, phone, dept, year, created_at))
            self.conn.commit()
            
            messagebox.showinfo("Success", "Student added successfully!")
            self.clear_fields()
            self.refresh_treeview()
            
        except sqlite3.IntegrityError:
            messagebox.showerror("Error", "Roll No already exists!")
        except ValueError:
            messagebox.showerror("Error", "Year must be a number!")
    
    def update_student(self):
        selected = self.tree.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a student to update!")
            return
            
        student_id = self.tree.item(selected)['values'][0]
        name = self.name_entry.get()
        roll_no = self.roll_entry.get()
        email = self.email_entry.get()
        phone = self.phone_entry.get()
        dept = self.dept_entry.get()
        year = self.year_entry.get()
        
        if not name or not roll_no:
            messagebox.showerror("Error", "Name and Roll No are required!")
            return
            
        try:
            year = int(year) if year else 0
            self.cursor.execute('''
                UPDATE students SET name=?, roll_no=?, email=?, phone=?, department=?, year=?
                WHERE id=?
            ''', (name, roll_no, email, phone, dept, year, student_id))
            self.conn.commit()
            
            messagebox.showinfo("Success", "Student updated successfully!")
            self.clear_fields()
            self.refresh_treeview()
            
        except sqlite3.IntegrityError:
            messagebox.showerror("Error", "Roll No already exists!")
        except ValueError:
            messagebox.showerror("Error", "Year must be a number!")
    
    def delete_student(self):
        selected = self.tree.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a student to delete!")
            return
            
        if messagebox.askyesno("Confirm", "Are you sure you want to delete this student?"):
            student_id = self.tree.item(selected)['values'][0]
            self.cursor.execute("DELETE FROM students WHERE id=?", (student_id,))
            self.conn.commit()
            messagebox.showinfo("Success", "Student deleted successfully!")
            self.clear_fields()
            self.refresh_treeview()
    
    def clear_fields(self):
        self.name_entry.delete(0, tk.END)
        self.roll_entry.delete(0, tk.END)
        self.email_entry.delete(0, tk.END)
        self.phone_entry.delete(0, tk.END)
        self.dept_entry.delete(0, tk.END)
        self.year_entry.delete(0, tk.END)
    
    def refresh_treeview(self):
        for item in self.tree.get_children():
            self.tree.delete(item)
            
        self.cursor.execute("SELECT * FROM students")
        for row in self.cursor.fetchall():
            self.tree.insert("", tk.END, values=row)
    
    def on_tree_select(self, event):
        selected = self.tree.selection()
        if selected:
            values = self.tree.item(selected)['values']
            self.clear_fields()
            self.name_entry.insert(0, values[1])
            self.roll_entry.insert(0, values[2])
            self.email_entry.insert(0, values[3])
            self.phone_entry.insert(0, values[4])
            self.dept_entry.insert(0, values[5])
            self.year_entry.insert(0, str(values[6]))
    
    def __del__(self):
        self.conn.close()

if __name__ == "__main__":
    root = tk.Tk()
    app = StudentManagementSystem(root)
    root.mainloop()
