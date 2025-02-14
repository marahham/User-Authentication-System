import hashlib
import sqlite3
import getpass

def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

def create_db():
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS users (username TEXT PRIMARY KEY, password TEXT)")
    conn.commit()
    conn.close()

def register():
    print("\nWelcome to our website! Let's create a new account for you.")
    username = input("Enter a new username: ")
    password = getpass.getpass("Enter a new password: ")
    
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, hash_password(password)))
        conn.commit()
        print("Registration successful! You can now log in.")
    except sqlite3.IntegrityError:
        print("Username already exists! Try a different one.")
    conn.close()

def login():
    print("\nWelcome back! Please log in to continue.")
    username = input("Please enter your username: ")
    
    # First password attempt
    password = getpass.getpass("Please enter your password: ")
    
    # Check if username exists
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    cursor.execute("SELECT password FROM users WHERE username = ?", (username,))
    result = cursor.fetchone()
    conn.close()
    
    # If username found, check password
    if result:
        if result[0] == hash_password(password):
            print(f"\nWelcome {username} to our website! We are happy to have you here.")
        else:
            print("\nIncorrect password. Please try again.")
            # Second password attempt
            password = getpass.getpass("Please enter your password again: ")
            # Recheck password
            conn = sqlite3.connect("users.db")
            cursor = conn.cursor()
            cursor.execute("SELECT password FROM users WHERE username = ?", (username,))
            result = cursor.fetchone()
            conn.close()
            
            if result and result[0] == hash_password(password):
                print(f"\nWelcome {username} to our website! We are happy to have you here.")
            else:
                print("\nInvalid password again. Access denied.")
    else:
        print("\nInvalid username. Please check your username and try again.")

create_db()
choice = input("Do you want to (1) Register or (2) Login? Enter 1 or 2: ")
if choice == "1":
    register()
elif choice == "2":
    login()
else:
    print("Invalid choice. Please restart the program.")

