import mysql.connector

# Establish a connection to MySQL
mydb = mysql.connector.connect(
    host="localhost",
    user="root",
    password="oob",
    database="bus_reservation_system"
)

# Create necessary tables
cursor = mydb.cursor()

# Create users table
cursor.execute("CREATE TABLE IF NOT EXISTS users (user_id INT PRIMARY KEY, username VARCHAR(50) NOT NULL, password VARCHAR(255) NOT NULL)")

# Create locations table
cursor.execute("CREATE TABLE IF NOT EXISTS locations (location_id varchar(10) PRIMARY KEY, location_name VARCHAR(50) NOT NULL)")

# Create buses table
cursor.execute("CREATE TABLE IF NOT EXISTS buses (bus_id INT PRIMARY KEY, bus_name VARCHAR(50) NOT NULL, location_id varchar(10), timing TIME, price DECIMAL(10, 2), seats_available INT)")

# Create seats table
cursor.execute("CREATE TABLE IF NOT EXISTS seats (seat_id INT  PRIMARY KEY, bus_id INT, seat_number INT, booked INT)")

# Create bookings table
cursor.execute("CREATE TABLE IF NOT EXISTS bookings (booking_id INT  PRIMARY KEY, user_id INT, seat_id INT)")

# Insert sample data into locations table
locations_data = [
    ("1", "Dubai"),
    ("2", "Abu Dhabi"),
    ("3", "Sharjah"),
    ("4", "Al Ain"),
    ("5", "Ajman"),
    ("6", "Ras Al Khaimah"),
    ("7", "Fujairah"),
    ("8", "Umm Al Quwain")
]
for i in locations_data:
    cursor.execute("INSERT INTO locations VALUES ('%s', '%s')"%(i[0],i[1]))

# Insert sample data into buses table
buses_data = [
    ("1", "Dubai Bus 1", 1, '08:00:00', 20.00, 50),
    ("2", "Dubai Bus 2", 1, '10:00:00', 25.00, 40),
    ("3", "Abu Dhabi Bus 1", 2, '12:00:00', 30.00, 30),
    ("4", "Abu Dhabi Bus 2", 2, '14:00:00', 35.00, 35),
    ("5", "Sharjah Bus 1", 3, '08:30:00', 18.00, 45),
    ("6", "Sharjah Bus 2", 3, '10:30:00', 22.00, 38),
    ("7", "Al Ain Bus 1", 4, '09:00:00', 27.00, 25),
    ("8", "Ajman Bus 1", 5, '11:00:00', 24.00, 42),
    ("9", "Ras Al Khaimah Bus 1", 6, '13:00:00', 30.00, 30),
    ("10", "Fujairah Bus 1", 7, '12:30:00', 28.00, 28)
]
for i in buses_data:
    cursor.execute("INSERT INTO buses VALUES (%s, %s, '%s', %s, %s, %s)"%(i[0],i[1],i[2],i[3],i[4],i[5]))

# Insert sample data into seats table
seats_data = [
    (1, 1, 1, 0),
    (2, 1, 2, 0),
    (3, 2, 1, 0),
    (4, 2, 2, 1),
    (5, 3, 1, 0),
    (6, 3, 2, 0)
]
cursor.executemany("INSERT INTO seats (seat_id, bus_id, seat_number, booked) VALUES (%s, %s, %s, %s)", seats_data)

mydb.commit()
cursor.close()

# Establish a new connection to MySQL
mydb = mysql.connector.connect(
    host="localhost",
    user="yourusername",
    password="yourpassword",
    database="bus_reservation_system"
)

# Function for user registration
def register(username, password):
    cursor = mydb.cursor()
    try:
        sql = "INSERT INTO users (username, password) VALUES (%s, %s)"
        val = (username, password)
        cursor.execute(sql, val)
        mydb.commit()
        print("Registration successful!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for user login
def login(username, password):
    cursor = mydb.cursor()
    try:
        sql = "SELECT * FROM users WHERE username = %s AND password = %s"
        val = (username, password)
        cursor.execute(sql, val)
        user = cursor.fetchone()
        if user:
            print("Login successful!")
        else:
            print("Invalid username or password!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for showing available locations
def show_locations():
    cursor = mydb.cursor()
    try:
        cursor.execute("SELECT * FROM locations")
        locations = cursor.fetchall()
        if locations:
            print("Available Locations:")
            for location in locations:
                print(location[0], location[1])
        else:
            print("No locations found.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for showing bus details for a location
def show_bus_details(location_id):
    cursor = mydb.cursor()
    try:
        sql = "SELECT * FROM buses WHERE location_id = %s"%(location_id)
        val = (location_id,)
        cursor.execute(sql, val)
        buses = cursor.fetchall()
        if buses:
            print("Available Buses:")
            for bus in buses:
                print(bus[0], bus[1], bus[3], bus[4], bus[5])
        else:
            print("No buses found for this location.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# # Function for showing available seats for a bus
def show_bus_seats(bus_id):
    cursor = mydb.cursor()
    try:
        sql = "SELECT * FROM seats WHERE bus_id = %s"
        val = (bus_id,)
        cursor.execute(sql, val)
        seats = cursor.fetchall()
        if seats:
            print("Available Seats:")
            for seat in seats:
                print(seat[0], seat[2], "Booked" if seat[3] else "Available")
        else:
            print("No seats found for this bus.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for booking a seat
def book_seat(user_id, bus_id, seat_number):
    cursor = mydb.cursor()
    try:
        sql = "INSERT INTO bookings (user_id, seat_id) VALUES (%s, %s)"
        val = (user_id, seat_number)
        cursor.execute(sql, val)
        mydb.commit()
        print("Seat booked successfully!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for canceling a booking
def cancel_booking(user_id, seat_id):
    cursor = mydb.cursor()
    try:
        sql = "DELETE FROM bookings WHERE user_id = %s AND seat_id = %s"
        val = (user_id, seat_id)
        cursor.execute(sql, val)
        mydb.commit()
        print("Booking canceled successfully!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for selecting a seat
def select_seat(user_id, bus_id, seat_number):
    cursor = mydb.cursor()
    try:
        sql = "UPDATE seats SET booked = 1 WHERE bus_id = %s AND seat_number = %s"
        val = (bus_id, seat_number)
        cursor.execute(sql, val)
        mydb.commit()
        print("Seat selected successfully!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for updating user location
def update_location(user_id, location_id):
    cursor = mydb.cursor()
    try:
        sql = "UPDATE users SET location_id = %s WHERE user_id = %s"
        val = (location_id, user_id)
        cursor.execute(sql, val)
        mydb.commit()
        print("Location updated successfully!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Function for updating seat
def update_seat(user_id, seat_id):
    cursor = mydb.cursor()
    try:
        sql = "UPDATE bookings SET seat_id = %s WHERE user_id = %s"
        val = (seat_id, user_id)
        cursor.execute(sql, val)
        mydb.commit()
        print("Seat updated successfully!")
    except mysql.connector.Error as err:
        print(f"Error: {err}")
    finally:
        cursor.close()

# Main function
def main():
    while True:
        print("City: Dubai")
        print("1. Register")
        print("2. Login")
        print("3. Show Bus Locations")
        print("4. Show Bus Details for a Location")
        print("5. Show Bus Seats")
        print("6. Book Seats")
        print("7. Cancel Booking")
        print("8. Select Seat")
        print("9. Update Location")
        print("10. Update Seat")
        print("11. Exit")
        choice = input("Enter your choice: ")
        if choice == "1":
            username = input("Enter username: ")
            password = input("Enter password: ")
            register(username, password)
        elif choice == "2":
            username = input("Enter username: ")
            password = input("Enter password: ")
            login(username, password)
        elif choice == "3":
            show_locations()
        elif choice == "4":
            location_id = (input("Enter location ID: "))
            show_bus_details(location_id)
        elif choice == "5":
            bus_id = input("Enter bus ID: ")
            show_bus_seats(bus_id)
        elif choice == "6":
            user_id = input("Enter your user ID: ")
            bus_id = input("Enter the bus ID you want to book: ")
            seat_number = input("Enter the seat number you want to book: ")
            book_seat(user_id, bus_id, seat_number)
        elif choice == "7":
            user_id = input("Enter your user ID: ")
            seat_id = input("Enter the seat ID you want to cancel: ")
            cancel_booking(user_id, seat_id)
        elif choice == "8":
            user_id = input("Enter your user ID: ")
            bus_id = input("Enter the bus ID: ")
            seat_number = input("Enter the seat number you want to select: ")
            select_seat(user_id, bus_id, seat_number)
        elif choice == "9":
            user_id = input("Enter your user ID: ")
            location_id = input("Enter the location ID you want to update: ")
            update_location(user_id, location_id)
        elif choice == "10":
            user_id = input("Enter your user ID: ")
            seat_id = input("Enter the new seat ID you want to update: ")
            update_seat(user_id, seat_id)
        elif choice == "11":
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()
