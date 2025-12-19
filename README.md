# Veterinary_clinic
# vet_clinic_complete.py
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
from datetime import datetime, timedelta
import random

class DatabaseHandler:
    
    def __init__(self, db_name="clinic.db"): #method
        """Initialize database connection and create tables if they don't exist."""
        self.db_name = db_name # this line is the attribute
        self.create_tables()
    
    def connect(self):
        """Establish connection to SQLite database."""
        return sqlite3.connect(self.db_name)
    
    def create_tables(self):
        """
        Create all necessary tables for the veterinary clinic system.
        Tables include: pets, veterinarians, inventory, appointments, billing, and records.
        """
        conn = self.connect()
        cursor = conn.cursor()

        # Create pets table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS pets (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            species TEXT NOT NULL,
            age INTEGER NOT NULL,
            owner_name TEXT NOT NULL,
            weight REAL,
            owner_contact TEXT
        )
        ''')

        # Create veterinarians table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS veterinarians (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            specialization TEXT NOT NULL,
            phone TEXT,
            email TEXT,
            license_number TEXT,
            is_available BOOLEAN DEFAULT 1
        )
        ''')

        # Create inventory table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS inventory (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            type TEXT NOT NULL,
            quantity INTEGER NOT NULL,
            price REAL NOT NULL,
            supplier TEXT,
            expiration_date TEXT,
            min_stock INTEGER DEFAULT 5
        )
        ''')

        # Create appointments table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS appointments (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            pet_id INTEGER NOT NULL,
            vet_id INTEGER,
            date TEXT NOT NULL,
            time TEXT NOT NULL,
            reason TEXT NOT NULL,
            status TEXT DEFAULT 'Scheduled',
            billing_amount REAL DEFAULT 0,
            FOREIGN KEY (pet_id) REFERENCES pets(id),
            FOREIGN KEY (vet_id) REFERENCES veterinarians(id)
        )
        ''')

        # Create billing table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS billing (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            appointment_id INTEGER NOT NULL,
            pet_id INTEGER NOT NULL,
            total_amount REAL NOT NULL,
            status TEXT DEFAULT 'Pending',
            payment_date TEXT,
            items TEXT,
            FOREIGN KEY (appointment_id) REFERENCES appointments(id),
            FOREIGN KEY (pet_id) REFERENCES pets(id)
        )
        ''')

        # Create records table for activity tracking
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS records (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL,
            record TEXT NOT NULL
        )
        ''')

        conn.commit()
        conn.close()

    # ============= DATABASE OPERATIONS =============
    
    def get_all_pets(self):
        """Retrieve all pets from the database in proper order."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM pets ORDER BY id")
        pets = cursor.fetchall()
        conn.close()
        return pets
    
    def get_all_veterinarians(self):
        """Retrieve all veterinarians from the database in proper order."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM veterinarians ORDER BY id")
        vets = cursor.fetchall()
        conn.close()
        return vets
    
    def get_all_appointments(self):
        """Retrieve all appointments from the database in proper order."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM appointments ORDER BY id")
        appointments = cursor.fetchall()
        conn.close()
        return appointments
    
    def get_all_inventory(self):
        """Retrieve all inventory items from the database in proper order."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM inventory ORDER BY id")
        inventory = cursor.fetchall()
        conn.close()
        return inventory
    
    def get_all_billing(self):
        """Retrieve all billing records from the database in proper order."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM billing ORDER BY id")
        billing = cursor.fetchall()
        conn.close()
        return billing
    
    def get_all_records(self):
        """Retrieve all activity records from the database, sorted by timestamp."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM records ORDER BY timestamp DESC")
        records = cursor.fetchall()
        conn.close()
        return records
    
    # ============= PETS OPERATIONS =============
    def add_pet(self, name, species, age, owner_name, weight=None, owner_contact=None):
        """Add a new pet to the database and return its ID."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        INSERT INTO pets (name, species, age, owner_name, weight, owner_contact)
        VALUES (?, ?, ?, ?, ?, ?)
        ''', (name, species, age, owner_name, weight, owner_contact))
        pet_id = cursor.lastrowid
        conn.commit()
        conn.close()
        return pet_id
    
    def update_pet(self, pet_id, name, species, age, owner_name, weight=None, owner_contact=None):
        """Update an existing pet in the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        UPDATE pets 
        SET name = ?, species = ?, age = ?, owner_name = ?, weight = ?, owner_contact = ?
        WHERE id = ?
        ''', (name, species, age, owner_name, weight, owner_contact, pet_id))
        conn.commit()
        conn.close()
    
    def delete_pet(self, pet_id):
        """Delete a pet from the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM pets WHERE id = ?", (pet_id,))
        conn.commit()
        conn.close()
    
    def get_pet_by_id(self, pet_id):
        """Get pet details by ID."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM pets WHERE id = ?", (pet_id,))
        pet = cursor.fetchone()
        conn.close()
        return pet
    
    # ============= VETERINARIANS OPERATIONS =============
    def add_veterinarian(self, name, specialization, phone=None, email=None, license_number=None, is_available=True):
        """Add a new veterinarian to the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        INSERT INTO veterinarians (name, specialization, phone, email, license_number, is_available)
        VALUES (?, ?, ?, ?, ?, ?)
        ''', (name, specialization, phone, email, license_number, is_available))
        conn.commit()
        conn.close()
    
    def update_veterinarian(self, vet_id, name, specialization, phone=None, email=None, license_number=None, is_available=True):
        """Update an existing veterinarian in the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        UPDATE veterinarians 
        SET name = ?, specialization = ?, phone = ?, email = ?, license_number = ?, is_available = ?
        WHERE id = ?
        ''', (name, specialization, phone, email, license_number, is_available, vet_id))
        conn.commit()
        conn.close()
    
    def delete_veterinarian(self, vet_id):
        """Delete a veterinarian from the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM veterinarians WHERE id = ?", (vet_id,))
        conn.commit()
        conn.close()
    
    def get_vet_by_id(self, vet_id):
        """Get veterinarian details by ID."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM veterinarians WHERE id = ?", (vet_id,))
        vet = cursor.fetchone()
        conn.close()
        return vet
    
    # ============= INVENTORY OPERATIONS =============
    def add_inventory_item(self, name, item_type, quantity, price, supplier=None, expiration_date=None, min_stock=5):
        """Add a new inventory item to the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        INSERT INTO inventory (name, type, quantity, price, supplier, expiration_date, min_stock)
        VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (name, item_type, quantity, price, supplier, expiration_date, min_stock))
        conn.commit()
        conn.close()
    
    def update_inventory_item(self, item_id, name, item_type, quantity, price, supplier=None, expiration_date=None, min_stock=5):
        """Update an existing inventory item in the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        UPDATE inventory 
        SET name = ?, type = ?, quantity = ?, price = ?, supplier = ?, expiration_date = ?, min_stock = ?
        WHERE id = ?
        ''', (name, item_type, quantity, price, supplier, expiration_date, min_stock, item_id))
        conn.commit()
        conn.close()
    
    def delete_inventory_item(self, item_id):
        """Delete an inventory item from the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM inventory WHERE id = ?", (item_id,))
        conn.commit()
        conn.close()
    
    def get_inventory_item_by_id(self, item_id):
        """Get inventory item details by ID."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM inventory WHERE id = ?", (item_id,))
        item = cursor.fetchone()
        conn.close()
        return item
    
    # ============= APPOINTMENTS OPERATIONS =============
    def add_appointment(self, pet_id, vet_id, date, time, reason, status="Scheduled", billing_amount=0):
        """Add a new appointment to the database and return its ID."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        INSERT INTO appointments (pet_id, vet_id, date, time, reason, status, billing_amount)
        VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (pet_id, vet_id, date, time, reason, status, billing_amount))
        appointment_id = cursor.lastrowid
        conn.commit()
        conn.close()
        return appointment_id
    
    def update_appointment(self, appointment_id, pet_id, vet_id, date, time, reason, status, billing_amount=0):
        """Update an existing appointment in the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        UPDATE appointments
        SET pet_id = ?, vet_id = ?, date = ?, time = ?, reason = ?, status = ?, billing_amount = ?
        WHERE id = ?
        ''', (pet_id, vet_id, date, time, reason, status, billing_amount, appointment_id))
        conn.commit()
        conn.close()

    def delete_appointment(self, appointment_id):
        """Delete an appointment from the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM appointments WHERE id = ?", (appointment_id,))
        conn.commit()
        conn.close()
    
    # ============= BILLING OPERATIONS =============
    def add_billing(self, appointment_id, pet_id, total_amount, status="Pending", items="", payment_date=None):
        """Add a new billing record to the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        INSERT INTO billing (appointment_id, pet_id, total_amount, status, items, payment_date)
        VALUES (?, ?, ?, ?, ?, ?)
        ''', (appointment_id, pet_id, total_amount, status, items, payment_date))
        conn.commit()
        conn.close()
    
    def update_billing(self, billing_id, appointment_id, pet_id, total_amount, status="Pending", items="", payment_date=None):
        """Update an existing billing record in the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        UPDATE billing
        SET appointment_id = ?, pet_id = ?, total_amount = ?, status = ?, items = ?, payment_date = ?
        WHERE id = ?
        ''', (appointment_id, pet_id, total_amount, status, items, payment_date, billing_id))
        conn.commit()
        conn.close()
    
    def delete_billing(self, billing_id):
        """Delete a billing record from the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM billing WHERE id = ?", (billing_id,))
        conn.commit()
        conn.close()
    
    # ============= RECORDS OPERATIONS =============
    def add_record(self, record):
        """Add an activity record to the database with current timestamp."""
        conn = self.connect()
        cursor = conn.cursor()
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        cursor.execute('''
        INSERT INTO records (timestamp, record)
        VALUES (?, ?)
        ''', (timestamp, record))
        conn.commit()
        conn.close()
    
    def delete_record(self, record_id):
        """Delete an activity record from the database."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM records WHERE id = ?", (record_id,))
        conn.commit()
        conn.close()
    
    # ============= HELPER METHODS =============
    def get_customer_name(self, pet_id):
        """Fetch customer (owner) name by pet ID."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT owner_name FROM pets WHERE id = ?", (pet_id,))
        result = cursor.fetchone()
        conn.close()
        return result[0] if result else None

    def get_appointment_details(self, appointment_id):
        """Get detailed appointment information including pet and vet names."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        SELECT 
            a.id, 
            a.pet_id,
            a.vet_id,
            a.date,
            a.time,
            a.reason,
            a.status,
            a.billing_amount,
            p.name as pet_name,
            p.owner_name,
            v.name as vet_name,
            v.specialization
        FROM appointments a
        LEFT JOIN pets p ON a.pet_id = p.id
        LEFT JOIN veterinarians v ON a.vet_id = v.id
        WHERE a.id = ?
        ''', (appointment_id,))
        result = cursor.fetchone()
        conn.close()
        return result

    def get_billing_with_customer_info(self):
        """Get billing information with customer details."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute('''
        SELECT 
            b.id,
            b.appointment_id,
            b.pet_id,
            b.total_amount,
            b.status,
            b.payment_date,
            b.items,
            COALESCE(p.owner_name, 'Unknown Customer') as owner_name,
            COALESCE(p.name, 'General Service') as pet_name,
            COALESCE(a.reason, 'General Service') as reason
        FROM billing b
        LEFT JOIN pets p ON b.pet_id = p.id
        LEFT JOIN appointments a ON b.appointment_id = a.id
        ORDER BY b.id
        ''')
        billing = cursor.fetchall()
        conn.close()
        return billing
    
    def get_pets_for_dropdown(self):
        """Get pets for dropdown selection (id, name, owner) in proper order."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT id, name, owner_name FROM pets ORDER BY owner_name, name")
        pets = cursor.fetchall()
        conn.close()
        return pets
    
    def get_pets_with_owners(self):
        """Get pets with owner information for dropdown selection."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT id, name, owner_name FROM pets ORDER BY owner_name, name")
        pets = cursor.fetchall()
        conn.close()
        return pets
    
    def get_vets_for_dropdown(self):
        """Get veterinarians for dropdown selection with specialization."""
        conn = self.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT id, name, specialization FROM veterinarians WHERE is_available = 1 ORDER BY id")
        vets = cursor.fetchall()
        conn.close()
        return vets

    def reset_auto_increment(self, table_name):
        """Reset auto-increment counter for a table."""
        conn = self.connect()
        cursor = conn.cursor()
        
        # Get maximum ID
        cursor.execute(f"SELECT MAX(id) FROM {table_name}")
        max_id = cursor.fetchone()[0]
        
        if max_id is not None:
            # Update SQLite sequence
            cursor.execute(f"UPDATE sqlite_sequence SET seq = ? WHERE name = ?", (max_id, table_name))
        
        conn.commit()
        conn.close()


class VetClinicGUI:
    """
    Main GUI class for the veterinary clinic management system.
    Handles all user interface components and interactions.
    """
    
    def __init__(self, root):
        """Initialize the main application window and setup GUI components."""
        self.root = root
        self.db = DatabaseHandler()
        
        # Configure window to open in fullscreen (zoomed state)
        self.root.state('zoomed')
        self.root.title("Paws & Claws Veterinary Clinic Management System")
        self.root.configure(bg="black")
        
        # Modern color scheme for the application
        self.colors = {  #value pairs
                       #used dictionary to store gui color settings so they can be accesed easily using keys.
            'primary': '#1a1a2e',      # Dark blue for headers and sidebars
            'secondary': '#16213e',     # Slightly lighter blue for cards
            'accent': '#e94560',        # Red accent color for important elements
            'light': '#ecf0f1',         # Light color for backgrounds
            'success': '#27ae60',       # Green for success actions
            'warning': '#f39c12',       # Orange for warnings
            'info': '#3498db',          # Blue for information
            'content_bg': '#0f3460',    # Background color for content areas
            'dark': '#1a1a1a'           # Dark color for listboxes
        }
        
        self.setup_gui()
        self.load_initial_data()
    
    def setup_gui(self):
        """Setup all GUI components including header, sidebar, and content area."""
        # ============= HEADER SECTION =============
        header_frame = tk.Frame(self.root, bg=self.colors['primary'], height=80)
        header_frame.pack(fill="x")
        header_frame.pack_propagate(False)  # Prevent frame from shrinking
        
        # Application title
        title_label = tk.Label(header_frame, text="🐾 Paws & Claws Veterinary Clinic", 
                              font=("Arial", 24, "bold"), fg="white", bg=self.colors['primary'])
        title_label.pack(side="left", padx=30, pady=20)
        
        # Date and time display
        self.date_label = tk.Label(header_frame, font=("Arial", 10), fg="white", bg=self.colors['primary'])
        self.date_label.pack(side="right", padx=30, pady=20)
        self.update_datetime()  # Start updating date/time
        
        # ============= MAIN CONTAINER =============
        main_container = tk.Frame(self.root, bg=self.colors['light'])
        main_container.pack(fill="both", expand=True, padx=20, pady=20)
        
        # ============= SIDEBAR =============
        sidebar = tk.Frame(main_container, bg=self.colors['primary'], width=220)
        sidebar.pack(side="left", fill="y", padx=(0, 20))
        sidebar.pack_propagate(False)
        
        # Menu buttons with icons
        menu_buttons = [
            ("📊 Dashboard", self.show_dashboard),
            ("📅 Appointments", self.show_appointments),
            ("🐕 Pets", self.show_pets),
            ("👨‍⚕️ Veterinarians", self.show_veterinarians),
            ("📦 Inventory", self.show_inventory),
            ("💰 Billing", self.show_billing),
            ("📋 Records", self.show_records),
        ]
        
        # Create menu buttons dynamically
        for i, (text, command) in enumerate(menu_buttons):
            btn = tk.Button(sidebar, text=text, font=("Arial", 12),
                           bg=self.colors['primary'], fg="white",
                           activebackground=self.colors['accent'],
                           activeforeground="white",
                           bd=0, padx=25, pady=18,
                           anchor="w", width=20,
                           relief="flat",
                           cursor="hand2",
                           command=command)
            btn.pack(fill="x", padx=15, pady=2)
        
        # Exit button at bottom of sidebar
        exit_btn = tk.Button(sidebar, text="🚪 Exit", font=("Arial", 12, "bold"),
                            bg=self.colors['accent'], fg="white",
                            activebackground="#c0392b",
                            bd=0, padx=25, pady=18,
                            anchor="w", width=20,
                            relief="flat",
                            cursor="hand2",
                            command=self.exit_system)
        exit_btn.pack(side="bottom", fill="x", padx=15, pady=(0, 20))
        
        # ============= CONTENT AREA =============
        self.content_frame = tk.Frame(main_container, bg=self.colors['content_bg'], relief="flat")
        self.content_frame.pack(side="right", fill="both", expand=True)
        
        # ============= STATUS BAR =============
        self.status_bar = tk.Label(self.root, text="Ready", bg=self.colors['primary'], 
                                  fg="white", anchor="w", font=("Arial", 10))
        self.status_bar.pack(side="bottom", fill="x")
        
        # Show dashboard initially when application starts
        self.show_dashboard()
    
    def load_initial_data(self):
        """Load initial data from database and add default inventory if needed."""
        self.db.add_record("System: Application started")
        
        # Check if inventory is empty and add default items if needed
        inventory = self.db.get_all_inventory()
        if len(inventory) == 0:
            self.add_default_inventory()
        
        # Add sample veterinarians if empty
        vets = self.db.get_all_veterinarians()
        if len(vets) == 0:
            self.add_sample_veterinarians()
        
        # Add sample pets if empty
        pets = self.db.get_all_pets()
        if len(pets) == 0:
            self.add_sample_pets()
            self.add_more_sample_pets()  # Add more pets with real names
        
        # Add sample billing data if empty
        billing = self.db.get_all_billing()
        if len(billing) == 0:
            self.add_real_billing_data()
    
    def add_default_inventory(self):
        """Add default inventory items if database is empty."""
        default_items = [
            ("Amoxicillin 250mg", "Medication - Antibiotic", 100, 60.00, "VetPharma", "2024-12-31", 20),
            ("Carprofen 75mg", "Medication - Pain Relief", 50, 125.00, "Zoetis", "2025-06-30", 15),
            ("Rabies Vaccine", "Vaccine", 30, 750.00, "Merial", "2024-09-15", 10),
            ("Distemper-Parvo Vaccine", "Vaccine", 25, 625.00, "Merial", "2024-10-31", 10),
            ("3ml Syringes", "Medical Supply", 200, 12.50, "MedSupply", "", 50),
            ("Exam Gloves Medium", "Medical Supply", 500, 5.00, "SafetyFirst", "", 100),
        ]
        
        for item in default_items:
            self.db.add_inventory_item(*item)
        
        self.db.add_record("System: Default inventory added")
    
    def add_sample_veterinarians(self):
        """Add sample veterinarians if database is empty."""
        sample_vets = [
            ("Dr. Sarah Johnson", "Surgery", "0917-123-4567", "sarah.j@clinic.com", "VET-001", True),
            ("Dr. Michael Chen", "Dermatology", "0918-987-6543", "m.chen@clinic.com", "VET-002", True),
            ("Dr. Maria Santos", "Internal Medicine", "0919-555-1234", "m.santos@clinic.com", "VET-003", True),
            ("Dr. Robert Lim", "Orthopedics", "0920-777-8888", "r.lim@clinic.com", "VET-004", False),
            ("Dr. James Martinez", "General Practice", "0921-111-2222", "j.martinez@clinic.com", "VET-005", True),
            ("Dr. Jennifer Kim", "Dentistry", "0922-333-4444", "j.kim@clinic.com", "VET-006", True),
            ("Dr. Lisa Anderson", "Radiology", "0923-555-6666", "l.anderson@clinic.com", "VET-007", True),
            ("Dr. Nicol Alday", "Surgery", "0924-777-8888", "n.alday@clinic.com", "VET-008", True),
            ("Dr. Sarah Wilson", "Preventive Care", "0925-999-0000", "s.wilson@clinic.com", "VET-009", True),
        ]
        
        for vet in sample_vets:
            self.db.add_veterinarian(*vet)
        
        self.db.add_record("System: Sample veterinarians added")
    
    def add_sample_pets(self):
        """Add sample pets if database is empty."""
        sample_pets = [  #list - stores multiple items.
                       #used list to store multiple records retrieved from the database, such as list of pet.
            ("Buddy", "Dog", 3, "Juan Dela Cruz", 15.5, "0917-111-2233"),
            ("Mittens", "Cat", 2, "Maria Clara", 4.2, "0918-222-3344"),
            ("Rocky", "Dog", 5, "Pedro Penduko", 25.0, "0919-333-4455"),
            ("Snowball", "Rabbit", 1, "Ana Reyes", 1.8, "0920-444-5566"),
        ]
        
        for pet in sample_pets:
            self.db.add_pet(*pet)
        
        self.db.add_record("System: Sample pets added")
    
    def add_more_sample_pets(self):
        """Add more sample pets with real customer names for billing."""
        sample_pets = [
            ("Max", "Golden Retriever", 4, "Emma Johnson", 12.3, "0917-555-1122"),
            ("Bella", "Persian Cat", 3, "Liam Chen", 4.5, "0918-666-2233"),
            ("Charlie", "Beagle", 6, "Olivia Martinez", 22.0, "0919-777-3344"),
            ("Lucy", "Border Collie", 2, "Noah Wilson", 8.7, "0920-888-4455"),
            ("Cooper", "Labrador", 5, "Ava Davis", 18.2, "0921-999-5566"),
            ("Luna", "Siamese Cat", 1, "Ethan Brown", 3.8, "0922-111-6677"),
            ("Bailey", "Boxer", 7, "Sophia Garcia", 26.5, "0923-222-7788"),
            ("Daisy", "Poodle", 4, "Mason Lee", 14.8, "0924-333-8899"),
            ("Milo", "Maine Coon", 2, "Isabella Taylor", 4.1, "0925-444-9900"),
            ("Zoe", "French Bulldog", 3, "James Rodriguez", 11.2, "0926-555-0011"),
            ("Rocky", "German Shepherd", 5, "Charlotte White", 25.0, "0927-666-1122"),
            ("Coco", "Labrador Retriever", 3, "Benjamin Harris", 16.5, "0928-777-2233"),
            ("Leo", "Ragdoll Cat", 2, "Amelia Clark", 5.2, "0929-888-3344"),
            ("Chloe", "Siberian Husky", 4, "Lucas Lewis", 20.3, "0930-999-4455"),
            ("Oscar", "Bulldog", 6, "Mia Walker", 28.7, "0931-111-5566"),
            ("Molly", "Australian Shepherd", 3, "Henry Allen", 15.8, "0932-222-6677"),
            ("Simba", "Bengal Cat", 1, "Harper Young", 4.0, "0933-333-7788"),
            ("Rosie", "Corgi", 2, "Alexander King", 9.5, "0934-444-8899"),
            ("Jack", "Dachshund", 7, "Ella Scott", 7.8, "0935-555-9900"),
            ("Lily", "Pomeranian", 3, "Daniel Green", 3.2, "0936-666-0011"),
            ("Duke", "Rottweiler", 5, "Sofia Adams", 30.5, "0937-777-1122"),
            ("Misty", "Scottish Fold", 2, "Matthew Hall", 4.3, "0938-888-2233"),
            ("Cooper", "Doberman", 4, "Avery Nelson", 22.8, "0939-999-3344"),
            ("Ruby", "Cavalier Spaniel", 1, "Scarlett Baker", 6.5, "0940-111-4455"),
            ("Bear", "Saint Bernard", 3, "Joseph Carter", 45.2, "0941-222-5566"),
            ("Pepper", "Chihuahua", 2, "Grace Mitchell", 2.8, "0942-333-6677"),
            ("Gizmo", "Parrot", 4, "David Perez", 0.8, "0943-444-7788"),
            ("Finn", "Great Dane", 2, "Chloe Roberts", 35.6, "0944-555-8899"),
            ("Lola", "Yorkshire Terrier", 1, "Jackson Turner", 2.5, "0945-666-9900"),
            ("Winston", "Boston Terrier", 3, "Zoe Phillips", 8.9, "0946-777-0011"),
            ("Nala", "Birman Cat", 2, "Samuel Campbell", 4.7, "0947-888-1122"),
            ("Zeus", "Mastiff", 4, "Penelope Parker", 42.3, "0948-999-2233"),
            ("Cleo", "Sphynx Cat", 1, "John Evans", 3.9, "0949-111-3344"),
            ("Rex", "Pit Bull", 3, "Victoria Edwards", 28.5, "0950-222-4455"),
        ]
        
        for pet in sample_pets:
            self.db.add_pet(*pet)
        
        self.db.add_record("System: More sample pets added")
    
    def add_real_billing_data(self):
        """Add real billing data with proper customer names and services."""
        # First, ensure we have pets and vets
        pets = self.db.get_all_pets()
        vets = self.db.get_all_veterinarians()
        
        if len(pets) > 0 and len(vets) > 0:
            # Real customer names and services for billing
            billing_data = [
                # (owner_name, pet_name_with_breed, service, amount, status)
                ("Emma Johnson", "Max (Golden Retriever)", "Annual Checkup & Vaccination", 185.50, "Paid"),
                ("Liam Chen", "Bella (Persian Cat)", "Feline Dental Cleaning", 225.75, "Paid"),
                ("Olivia Martinez", "Charlie (Beagle)", "Emergency Surgery", 650.00, "Partial Payment"),
                ("Noah Wilson", "Lucy (Border Collie)", "Behavioral Consultation", 135.25, "Paid"),
                ("Ava Davis", "Cooper (Labrador)", "X-Ray & Diagnosis", 325.00, "Pending"),
                ("Ethan Brown", "Luna (Siamese Cat)", "Spaying Procedure", 450.50, "Paid"),
                ("Sophia Garcia", "Bailey (Boxer)", "Skin Allergy Treatment", 195.75, "Paid"),
                ("Mason Lee", "Daisy (Poodle)", "Ear Infection Treatment", 155.25, "Partial Payment"),
                ("Isabella Taylor", "Milo (Maine Coon)", "Senior Wellness Exam", 210.00, "Pending"),
                ("James Rodriguez", "Zoe (French Bulldog)", "Microchipping Service", 85.00, "Paid"),
                ("Charlotte White", "Rocky (German Shepherd)", "Ultrasound Examination", 375.50, "Pending"),
                ("Benjamin Harris", "Coco (Labrador Retriever)", "Deworming Treatment", 95.75, "Paid"),
                ("Amelia Clark", "Leo (Ragdoll Cat)", "Feline Vaccination Package", 145.25, "Paid"),
                ("Lucas Lewis", "Chloe (Siberian Husky)", "Orthopedic Consultation", 285.00, "Partial Payment"),
                ("Mia Walker", "Oscar (Bulldog)", "Eye Infection Treatment", 175.00, "Pending"),
                ("Henry Allen", "Molly (Australian Shepherd)", "Nutritional Consultation", 120.50, "Paid"),
                ("Harper Young", "Simba (Bengal Cat)", "Castration Surgery", 380.00, "Paid"),
                ("Alexander King", "Rosie (Corgi)", "Feline Leukemia Testing", 125.75, "Pending"),
                ("Ella Scott", "Jack (Dachshund)", "Cardiac Examination", 310.50, "Paid"),
                ("Daniel Green", "Lily (Pomeranian)", "Kennel Cough Vaccine", 90.25, "Partial Payment"),
                ("Sofia Adams", "Duke (Rottweiler)", "Hip Dysplasia Screening", 295.00, "Pending"),
                ("Matthew Hall", "Misty (Scottish Fold)", "Dental Extraction", 255.75, "Paid"),
                ("Avery Nelson", "Cooper (Doberman)", "Avian Wellness Check", 180.00, "Paid"),
                ("Scarlett Baker", "Ruby (Cavalier Spaniel)", "Joint Supplement Injection", 165.50, "Pending"),
                ("Joseph Carter", "Bear (Saint Bernard)", "Professional Teeth Cleaning", 195.00, "Paid"),
                ("Grace Mitchell", "Pepper (Chihuahua)", "Comprehensive Allergy Testing", 245.75, "Partial Payment"),
                ("David Perez", "Gizmo (Parrot)", "Urinary Tract Treatment", 210.25, "Pending"),
                ("Chloe Roberts", "Finn (Great Dane)", "Weight Management Program", 185.00, "Paid"),
                ("Jackson Turner", "Lola (Yorkshire Terrier)", "Dermatology Consultation", 165.50, "Paid"),
                ("Zoe Phillips", "Winston (Boston Terrier)", "Emergency Trauma Surgery", 750.00, "Pending"),
                ("Samuel Campbell", "Nala (Birman Cat)", "Blood Tests Panel", 275.25, "Paid"),
                ("Penelope Parker", "Zeus (Mastiff)", "Dental Surgery Complex", 525.00, "Partial Payment"),
                ("John Evans", "Cleo (Sphynx Cat)", "Grooming & Hygiene Service", 115.50, "Paid"),
                ("Victoria Edwards", "Rex (Pit Bull)", "Follow-up Examination", 95.75, "Pending"),
        ]
            
            # Create appointments and billing for each entry
            for i, (owner_name, pet_service, service_type, amount, status) in enumerate(billing_data[:34]):
                # Find pet by owner name
                pet_id = None
                for pet in pets:
                    if pet[4] == owner_name:  # owner_name is at index 4
                        pet_id = pet[0]
                        break
                
                if not pet_id:
                    # If pet not found, use the first pet
                    pet_id = pets[i % len(pets)][0]
                
                # Select a vet
                vet_id = vets[i % len(vets)][0] if i < len(vets) else None
                
                # Create appointment date (spread over time)
                date = (datetime.now() - timedelta(days=random.randint(1, 60))).strftime("%Y-%m-%d")
                time = f"{random.randint(9, 16)}:{random.choice(['00', '15', '30', '45'])}"
                
                # Create appointment
                appointment_id = self.db.add_appointment(
                    pet_id, vet_id, date, time, service_type, 
                    "Completed",
                    amount
                )
                
                # Set payment date if status is Paid
                payment_date = date if status == "Paid" else None
                
                # Create billing for the appointment
                self.db.add_billing(
                    appointment_id, pet_id, 
                    amount,
                    status,
                    f"{service_type} for {pet_service}",
                    payment_date
                )
            
            self.db.add_record("System: Real billing data added with proper customer names")
    
    def update_datetime(self):
        """Update the date and time display every second."""
        current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.date_label.config(text=current_time)
        self.root.after(1000, self.update_datetime)
    
    def clear_content(self):
        """Clear all widgets from the content frame."""
        for widget in self.content_frame.winfo_children():
            widget.destroy()
    
    def show_dashboard(self):
        """Display the dashboard section with statistics and overview."""
        self.clear_content()
        self.status_bar.config(text="Dashboard")
        
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))
        
        title = tk.Label(title_frame, text="📊 Dashboard Overview", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(anchor="w")
        
        # Get data from database
        pets = self.db.get_all_pets()
        vets = self.db.get_all_veterinarians()
        appointments = self.db.get_all_appointments()
        
        available_vets = sum(1 for vet in vets if vet[6] == 1)
        pending_appointments = sum(1 for app in appointments if app[6] == "Scheduled")
        billing = self.db.get_all_billing()
        total_revenue = sum(bill[3] for bill in billing if bill[4] == "Paid")
        
        # Statistics Cards
        stats_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        stats_frame.pack(fill="x", padx=30, pady=10)
        
        stats = [
            ("Total Pets", len(pets), "#3498db", "🐕"),
            ("Available Vets", available_vets, "#2ecc71", "👨‍⚕️"),
            ("Pending Appointments", pending_appointments, "#e74c3c", "📅"),
            ("Total Revenue", f"₱{total_revenue:,.2f}", "#9b59b6", "💰"),
        ]
        
        for i, (label, value, color, icon) in enumerate(stats):
            stat_card = tk.Frame(stats_frame, bg=self.colors['secondary'], relief="flat", bd=0)
            stat_card.grid(row=0, column=i, padx=10, pady=10, sticky="nsew")
            stats_frame.grid_columnconfigure(i, weight=1)
            
            top_frame = tk.Frame(stat_card, bg=self.colors['secondary'])
            top_frame.pack(pady=(15, 5))
            
            icon_label = tk.Label(top_frame, text=icon, font=("Arial", 20), 
                                 bg=self.colors['secondary'], fg=color)
            icon_label.pack(side="left", padx=(10, 5))
            
            value_label = tk.Label(top_frame, text=str(value), font=("Arial", 24, "bold"), 
                                 bg=self.colors['secondary'], fg="white")
            value_label.pack(side="left", padx=(5, 10))
            
            label_label = tk.Label(stat_card, text=label, font=("Arial", 11), 
                                 bg=self.colors['secondary'], fg="#cccccc")
            label_label.pack(pady=(0, 15))
        
        # Main Content Area
        main_content = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        main_content.pack(fill="both", expand=True, padx=30, pady=20)
        
        # Left column - Low Stock Items
        left_frame = tk.Frame(main_content, bg=self.colors['secondary'], relief="flat")
        left_frame.pack(side="left", fill="both", expand=True, padx=(0, 10))
        
        left_title = tk.Label(left_frame, text="⚠️ Low Stock Items", 
                             font=("Arial", 16, "bold"), fg="white", bg=self.colors['secondary'])
        left_title.pack(anchor="w", padx=20, pady=15)
        
        low_stock_listbox = tk.Listbox(left_frame, font=("Arial", 11), 
                                     height=8, bg=self.colors['dark'], fg="white",
                                     selectbackground=self.colors['accent'])
        low_stock_listbox.pack(fill="both", expand=True, padx=20, pady=(0, 20))
        
        inventory = self.db.get_all_inventory()
        low_stock_items = []
        for item in inventory:
            if item[3] <= item[7]:  # quantity <= min_stock
                low_stock_items.append(item)
        
        if low_stock_items:
            for item in low_stock_items[:10]:
                low_stock_listbox.insert("end", f"• {item[1]}: {item[3]} left (min: {item[7]})")
        else:
            low_stock_listbox.insert("end", "✅ All items are sufficiently stocked")
        
        # Right column - Recent Activity
        right_frame = tk.Frame(main_content, bg=self.colors['secondary'], relief="flat")
        right_frame.pack(side="right", fill="both", expand=True, padx=(10, 0))
        
        right_title = tk.Label(right_frame, text="📋 Recent Activity", 
                              font=("Arial", 16, "bold"), fg="white", bg=self.colors['secondary'])
        right_title.pack(anchor="w", padx=20, pady=15)
        
        activity_listbox = tk.Listbox(right_frame, font=("Arial", 10), 
                                     height=8, bg=self.colors['dark'], fg="white",
                                     selectbackground=self.colors['accent'])
        activity_listbox.pack(fill="both", expand=True, padx=20, pady=(0, 20))
        
        records = self.db.get_all_records()
        if records:
            recent_records = records[:8]
            for record in recent_records:
                activity_listbox.insert("end", f"[{record[1][11:16]}] {record[2][:50]}...")
        else:
            activity_listbox.insert("end", "No recent activity")
    
    def show_pets(self):
        """Display the pets management section with full CRUD operations."""
        self.clear_content()
        self.status_bar.config(text="Pets Management")
        
        # Title and Controls
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))
        
        title = tk.Label(title_frame, text="🐕 Pets Management", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(side="left")
        
        control_frame = tk.Frame(title_frame, bg=self.colors['content_bg'])
        control_frame.pack(side="right")
        
        add_btn = tk.Button(control_frame, text="➕ Add Pet", 
                           bg=self.colors['success'], fg="white",
                           font=("Arial", 11, "bold"), padx=20, pady=10,
                           command=self.add_pet_dialog)
        add_btn.pack(side="left", padx=5)
        
        refresh_btn = tk.Button(control_frame, text="🔄 Refresh", 
                               bg=self.colors['info'], fg="white",
                               font=("Arial", 11, "bold"), padx=20, pady=10,
                               command=self.load_pets_data)
        refresh_btn.pack(side="left", padx=5)
        
        # Pets Table
        table_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        table_frame.pack(fill="both", expand=True, padx=30, pady=10)
        
        # Create treeview
        columns = ("ID", "Name", "Species", "Age", "Owner", "Weight", "Contact")
        self.pets_tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        
        # Define column headings
        for col in columns:
            self.pets_tree.heading(col, text=col)
            self.pets_tree.column(col, anchor="center", width=120)
        
        # Add scrollbars
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.pets_tree.yview)
        v_scrollbar.pack(side="right", fill="y")
        self.pets_tree.configure(yscrollcommand=v_scrollbar.set)
        
        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal", command=self.pets_tree.xview)
        h_scrollbar.pack(side="bottom", fill="x")
        self.pets_tree.configure(xscrollcommand=h_scrollbar.set)
        
        self.pets_tree.pack(fill="both", expand=True)
        
        # Load pets data
        self.load_pets_data()
        
        # Action Buttons
        action_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        action_frame.pack(fill="x", padx=30, pady=10)
        
        view_btn = tk.Button(action_frame, text="👁️ View Details", 
                            bg=self.colors['secondary'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.view_pet_details)
        view_btn.pack(side="left", padx=5)
        
        edit_btn = tk.Button(action_frame, text="✏️ Edit Pet", 
                            bg=self.colors['warning'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.edit_pet_dialog)
        edit_btn.pack(side="left", padx=5)
        
        delete_btn = tk.Button(action_frame, text="🗑️ Delete Pet", 
                              bg=self.colors['accent'], fg="white",
                              font=("Arial", 11), padx=15, pady=8,
                              command=self.delete_pet)
        delete_btn.pack(side="left", padx=5)
    
    def load_pets_data(self):
        """Load pets data from database into treeview in proper ID order."""
        if hasattr(self, 'pets_tree'):
            for item in self.pets_tree.get_children():
                self.pets_tree.delete(item)
        
        pets = self.db.get_all_pets()
        if pets:
            for pet in pets:
                self.pets_tree.insert("", "end", values=pet)
        else:
            self.pets_tree.insert("", "end", values=("No pets found", "", "", "", "", "", ""))
    
    def add_pet_dialog(self):
        """Open dialog to add a new pet."""
        dialog = tk.Toplevel(self.root)
        dialog.title("Add New Pet")
        dialog.geometry("500x450")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text="🐕 Add New Pet", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        labels = ["Pet Name:", "Species:", "Age:", "Owner Name:", "Weight (kg):", "Contact:"]
        entries = []
        
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
            entry.grid(row=i, column=1, pady=10, padx=10)
            entries.append(entry)
        
        # Set default values
        entries[0].insert(0, "")
        entries[1].insert(0, "Dog")
        entries[2].insert(0, "1")
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_pet():
            """Save pet data to database."""
            name = entries[0].get()
            species = entries[1].get()
            age = entries[2].get()
            owner_name = entries[3].get()
            weight = entries[4].get()
            contact = entries[5].get()
            
            if name and species and age and owner_name:
                try:
                    age_int = int(age)
                    weight_float = float(weight) if weight else None
                    
                    # Add pet to database
                    pet_id = self.db.add_pet(name, species, age_int, owner_name, weight_float, contact)
                    
                    # Add activity record
                    self.db.add_record(f"Pet added: {name} (ID: {pet_id})")
                    
                    # Refresh pets display
                    self.load_pets_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Pet '{name}' added successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Name, Species, Age, and Owner Name are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Pet", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_pet)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def view_pet_details(self):
        """View details of selected pet."""
        if not hasattr(self, 'pets_tree'):
            messagebox.showerror("Error", "Pets table not loaded!")
            return
        
        selection = self.pets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a pet first!")
            return
        
        item = self.pets_tree.item(selection[0])
        values = item['values']
        pet_id = int(values[0])
        
        # Get detailed pet information
        pet = self.db.get_pet_by_id(pet_id)
        if not pet:
            messagebox.showerror("Error", "Pet not found!")
            return
        
        # Create details dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Pet Details - {pet[1]}")
        dialog.geometry("500x400")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        
        title = tk.Label(dialog, text=f"🐕 Pet Details: {pet[1]}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        details_frame = tk.Frame(dialog, bg="white")
        details_frame.pack(fill="both", expand=True, padx=40)
        
        details_text = f"""
        🆔 Pet ID: {pet[0]}
        🐾 Name: {pet[1]}
        🐕 Species: {pet[2]}
        📅 Age: {pet[3]} years
        👤 Owner: {pet[4]}
        ⚖️ Weight: {pet[5] if pet[5] else 'N/A'} kg
        📞 Contact: {pet[6] if pet[6] else 'N/A'}
        """
        
        details_label = tk.Label(details_frame, text=details_text, 
                                font=("Arial", 12), bg="white", justify="left")
        details_label.pack(anchor="w", pady=20)
        
        close_btn = tk.Button(dialog, text="Close", 
                             font=("Arial", 12, "bold"),
                             bg=self.colors['primary'], fg="white",
                             padx=30, pady=10, command=dialog.destroy)
        close_btn.pack(pady=20)
    
    def edit_pet_dialog(self):
        """Edit an existing pet."""
        if not hasattr(self, 'pets_tree'):
            messagebox.showerror("Error", "Pets table not loaded!")
            return
        
        selection = self.pets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a pet first!")
            return
        
        item = self.pets_tree.item(selection[0])
        values = item['values']
        pet_id = int(values[0])
        
        # Get current pet data
        pet = self.db.get_pet_by_id(pet_id)
        if not pet:
            messagebox.showerror("Error", "Pet not found!")
            return
        
        # Create edit dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Edit Pet - {pet[1]}")
        dialog.geometry("500x450")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text=f"✏️ Edit Pet: {pet[1]}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        labels = ["Pet Name:", "Species:", "Age:", "Owner Name:", "Weight (kg):", "Contact:"]
        entries = []
        
        # Current values
        current_values = [
            pet[1],  # name
            pet[2],  # species
            str(pet[3]),  # age
            pet[4],  # owner_name
            str(pet[5]) if pet[5] else "",  # weight
            pet[6] if pet[6] else ""  # contact
        ]
        
        for i, (label_text, current_value) in enumerate(zip(labels, current_values)):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
            entry.grid(row=i, column=1, pady=10, padx=10)
            entry.insert(0, current_value)
            entries.append(entry)
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_edited_pet():
            """Save edited pet data to database."""
            name = entries[0].get()
            species = entries[1].get()
            age = entries[2].get()
            owner_name = entries[3].get()
            weight = entries[4].get()
            contact = entries[5].get()
            
            if name and species and age and owner_name:
                try:
                    age_int = int(age)
                    weight_float = float(weight) if weight else None
                    
                    # Update pet in database
                    self.db.update_pet(pet_id, name, species, age_int, owner_name, weight_float, contact)
                    
                    # Add activity record
                    self.db.add_record(f"Pet updated: {name} (ID: {pet_id})")
                    
                    # Refresh pets display
                    self.load_pets_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Pet '{name}' updated successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Name, Species, Age, and Owner Name are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Changes", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_edited_pet)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def delete_pet(self):
        """Delete a pet with confirmation."""
        if not hasattr(self, 'pets_tree'):
            messagebox.showerror("Error", "Pets table not loaded!")
            return
        
        selection = self.pets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a pet first!")
            return
        
        item = self.pets_tree.item(selection[0])
        values = item['values']
        pet_id = int(values[0])
        pet_name = values[1]
        
        if messagebox.askyesno("Confirm Deletion", 
                              f"Are you sure you want to delete pet '{pet_name}' (ID: {pet_id})?\n\n⚠️ Warning: This will also delete related appointments and billing records!"):
            
            # Delete from database
            self.db.delete_pet(pet_id)
            
            # Add activity record
            self.db.add_record(f"Pet deleted: {pet_name} (ID: {pet_id})")
            
            # Refresh display
            self.load_pets_data()
            
            messagebox.showinfo("Success", f"Pet '{pet_name}' deleted successfully!")
    
    def show_veterinarians(self):
        """Display the veterinarians management section with full CRUD operations."""
        self.clear_content()
        self.status_bar.config(text="Veterinarians Management")
        
        # Title and Controls
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))
        
        title = tk.Label(title_frame, text="👨‍⚕️ Veterinarians Management", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(side="left")
        
        control_frame = tk.Frame(title_frame, bg=self.colors['content_bg'])
        control_frame.pack(side="right")
        
        add_btn = tk.Button(control_frame, text="➕ Add Vet", 
                           bg=self.colors['success'], fg="white",
                           font=("Arial", 11, "bold"), padx=20, pady=10,
                           command=self.add_vet_dialog)
        add_btn.pack(side="left", padx=5)
        
        refresh_btn = tk.Button(control_frame, text="🔄 Refresh", 
                               bg=self.colors['info'], fg="white",
                               font=("Arial", 11, "bold"), padx=20, pady=10,
                               command=self.load_vets_data)
        refresh_btn.pack(side="left", padx=5)
        
        # Veterinarians Table
        table_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        table_frame.pack(fill="both", expand=True, padx=30, pady=10)
        
        # Create treeview
        columns = ("ID", "Name", "Specialization", "Phone", "Email", "License", "Available")
        self.vets_tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        
        # Define column headings
        for col in columns:
            self.vets_tree.heading(col, text=col)
            self.vets_tree.column(col, anchor="center", width=120)
        
        # Add scrollbars
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.vets_tree.yview)
        v_scrollbar.pack(side="right", fill="y")
        self.vets_tree.configure(yscrollcommand=v_scrollbar.set)
        
        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal", command=self.vets_tree.xview)
        h_scrollbar.pack(side="bottom", fill="x")
        self.vets_tree.configure(xscrollcommand=h_scrollbar.set)
        
        self.vets_tree.pack(fill="both", expand=True)
        
        # Load veterinarians data
        self.load_vets_data()
        
        # Action Buttons
        action_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        action_frame.pack(fill="x", padx=30, pady=10)
        
        view_btn = tk.Button(action_frame, text="👁️ View Details", 
                            bg=self.colors['secondary'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.view_vet_details)
        view_btn.pack(side="left", padx=5)
        
        edit_btn = tk.Button(action_frame, text="✏️ Edit Vet", 
                            bg=self.colors['warning'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.edit_vet_dialog)
        edit_btn.pack(side="left", padx=5)
        
        delete_btn = tk.Button(action_frame, text="🗑️ Delete Vet", 
                              bg=self.colors['accent'], fg="white",
                              font=("Arial", 11), padx=15, pady=8,
                            command=self.delete_vet)
        delete_btn.pack(side="left", padx=5)
        
        toggle_btn = tk.Button(action_frame, text="🔄 Toggle Availability", 
                              bg=self.colors['info'], fg="white",
                              font=("Arial", 11), padx=15, pady=8,
                              command=self.toggle_vet_availability)
        toggle_btn.pack(side="left", padx=5)
    
    def load_vets_data(self):
        """Load veterinarians data from database into treeview in proper ID order."""
        if hasattr(self, 'vets_tree'):
            for item in self.vets_tree.get_children():
                self.vets_tree.delete(item)
        
        vets = self.db.get_all_veterinarians()
        if vets:
            for vet in vets:
                available = "✅ Yes" if vet[6] == 1 else "❌ No"
                self.vets_tree.insert("", "end", values=(
                    vet[0], vet[1], vet[2], vet[3], vet[4], vet[5], available
                ))
        else:
            self.vets_tree.insert("", "end", values=("No veterinarians found", "", "", "", "", "", ""))
    
    def add_vet_dialog(self):
        """Open dialog to add a new veterinarian."""
        dialog = tk.Toplevel(self.root)
        dialog.title("Add New Veterinarian")
        dialog.geometry("500x500")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text="👨‍⚕️ Add New Veterinarian", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        labels = ["Name:", "Specialization:", "Phone:", "Email:", "License Number:", "Available:"]
        entries = []
        
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            if label_text == "Available:":
                var = tk.BooleanVar(value=True)
                checkbtn = tk.Checkbutton(form_frame, variable=var, bg="white")
                checkbtn.grid(row=i, column=1, pady=10, padx=10, sticky="w")
                entries.append(var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i, column=1, pady=10, padx=10)
                entries.append(entry)
        
        # Set default values
        entries[0].insert(0, "Dr. ")
        entries[1].insert(0, "General Practice")
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_vet():
            """Save veterinarian data to database."""
            name = entries[0].get()
            specialization = entries[1].get()
            phone = entries[2].get()
            email = entries[3].get()
            license_num = entries[4].get()
            available = entries[5].get() if isinstance(entries[5], tk.BooleanVar) else True
            
            if name and specialization:
                # Add veterinarian to database
                self.db.add_veterinarian(name, specialization, phone, email, license_num, available)
                
                # Add activity record
                self.db.add_record(f"Veterinarian added: {name}")
                
                # Refresh veterinarians display
                self.load_vets_data()
                
                dialog.destroy()
                messagebox.showinfo("Success", f"Veterinarian '{name}' added successfully!")
            else:
                messagebox.showerror("Error", "Name and Specialization are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Veterinarian", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_vet)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def view_vet_details(self):
        """View details of selected veterinarian."""
        if not hasattr(self, 'vets_tree'):
            messagebox.showerror("Error", "Veterinarians table not loaded!")
            return
        
        selection = self.vets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a veterinarian first!")
            return
        
        item = self.vets_tree.item(selection[0])
        values = item['values']
        vet_id = int(values[0])
        
        # Get detailed veterinarian information
        vet = self.db.get_vet_by_id(vet_id)
        if not vet:
            messagebox.showerror("Error", "Veterinarian not found!")
            return
        
        # Create details dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Veterinarian Details - {vet[1]}")
        dialog.geometry("500x400")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        
        title = tk.Label(dialog, text=f"👨‍⚕️ Veterinarian Details: {vet[1]}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        details_frame = tk.Frame(dialog, bg="white")
        details_frame.pack(fill="both", expand=True, padx=40)
        
        available = "✅ Available" if vet[6] == 1 else "❌ Not Available"
        details_text = f"""
        🆔 Vet ID: {vet[0]}
        👤 Name: {vet[1]}
        🎯 Specialization: {vet[2]}
        📞 Phone: {vet[3] if vet[3] else 'N/A'}
        📧 Email: {vet[4] if vet[4] else 'N/A'}
        📄 License: {vet[5] if vet[5] else 'N/A'}
        🟢 Status: {available}
        """
        
        details_label = tk.Label(details_frame, text=details_text, 
                                font=("Arial", 12), bg="white", justify="left")
        details_label.pack(anchor="w", pady=20)
        
        close_btn = tk.Button(dialog, text="Close", 
                             font=("Arial", 12, "bold"),
                             bg=self.colors['primary'], fg="white",
                             padx=30, pady=10, command=dialog.destroy)
        close_btn.pack(pady=20)
    
    def edit_vet_dialog(self):
        """Edit an existing veterinarian."""
        if not hasattr(self, 'vets_tree'):
            messagebox.showerror("Error", "Veterinarians table not loaded!")
            return
        
        selection = self.vets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a veterinarian first!")
            return
        
        item = self.vets_tree.item(selection[0])
        values = item['values']
        vet_id = int(values[0])
        
        # Get current veterinarian data
        vet = self.db.get_vet_by_id(vet_id)
        if not vet:
            messagebox.showerror("Error", "Veterinarian not found!")
            return
        
        # Create edit dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Edit Veterinarian - {vet[1]}")
        dialog.geometry("500x500")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text=f"✏️ Edit Veterinarian: {vet[1]}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        labels = ["Name:", "Specialization:", "Phone:", "Email:", "License Number:", "Available:"]
        entries = []
        
        # Current values
        current_values = [
            vet[1],  # name
            vet[2],  # specialization
            vet[3] if vet[3] else "",  # phone
            vet[4] if vet[4] else "",  # email
            vet[5] if vet[5] else "",  # license
            bool(vet[6])  # available
        ]
        
        for i, (label_text, current_value) in enumerate(zip(labels, current_values)):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            if label_text == "Available:":
                var = tk.BooleanVar(value=current_value)
                checkbtn = tk.Checkbutton(form_frame, variable=var, bg="white")
                checkbtn.grid(row=i, column=1, pady=10, padx=10, sticky="w")
                entries.append(var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i, column=1, pady=10, padx=10)
                entry.insert(0, current_value)
                entries.append(entry)
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_edited_vet():
            """Save edited veterinarian data to database."""
            name = entries[0].get()
            specialization = entries[1].get()
            phone = entries[2].get()
            email = entries[3].get()
            license_num = entries[4].get()
            available = entries[5].get() if isinstance(entries[5], tk.BooleanVar) else True
            
            if name and specialization:
                # Update veterinarian in database
                self.db.update_veterinarian(vet_id, name, specialization, phone, email, license_num, available)
                
                # Add activity record
                self.db.add_record(f"Veterinarian updated: {name}")
                
                # Refresh veterinarians display
                self.load_vets_data()
                
                dialog.destroy()
                messagebox.showinfo("Success", f"Veterinarian '{name}' updated successfully!")
            else:
                messagebox.showerror("Error", "Name and Specialization are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Changes", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_edited_vet)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def delete_vet(self):
        """Delete a veterinarian with confirmation."""
        if not hasattr(self, 'vets_tree'):
            messagebox.showerror("Error", "Veterinarians table not loaded!")
            return
        
        selection = self.vets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a veterinarian first!")
            return
        
        item = self.vets_tree.item(selection[0])
        values = item['values']
        vet_id = int(values[0])
        vet_name = values[1]
        
        if messagebox.askyesno("Confirm Deletion", 
                              f"Are you sure you want to delete veterinarian '{vet_name}' (ID: {vet_id})?\n\n⚠️ Warning: This will affect related appointments!"):
            
            # Delete from database
            self.db.delete_veterinarian(vet_id)
            
            # Add activity record
            self.db.add_record(f"Veterinarian deleted: {vet_name} (ID: {vet_id})")
            
            # Refresh display
            self.load_vets_data()
            
            messagebox.showinfo("Success", f"Veterinarian '{vet_name}' deleted successfully!")
    
    def toggle_vet_availability(self):
        """Toggle veterinarian availability status."""
        if not hasattr(self, 'vets_tree'):
            messagebox.showerror("Error", "Veterinarians table not loaded!")
            return
        
        selection = self.vets_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a veterinarian first!")
            return
        
        item = self.vets_tree.item(selection[0])
        values = item['values']
        vet_id = int(values[0])
        vet_name = values[1]
        current_status = values[6]
        
        # Get current veterinarian data
        vet = self.db.get_vet_by_id(vet_id)
        if not vet:
            messagebox.showerror("Error", "Veterinarian not found!")
            return
        
        new_status = not bool(vet[6])  # Toggle availability
        
        # Update veterinarian in database
        self.db.update_veterinarian(
            vet_id, vet[1], vet[2], vet[3], vet[4], vet[5], new_status
        )
        
        # Add activity record
        status_text = "Available" if new_status else "Not Available"
        self.db.add_record(f"Veterinarian availability toggled: {vet_name} is now {status_text}")
        
        # Refresh veterinarians display
        self.load_vets_data()
        
        messagebox.showinfo("Success", f"Veterinarian '{vet_name}' is now {'Available' if new_status else 'Not Available'}!")
    
    def show_inventory(self):
        """Display the inventory management section with full CRUD operations."""
        self.clear_content()
        self.status_bar.config(text="Inventory Management")
        
        # Title and Controls
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))
        
        title = tk.Label(title_frame, text="📦 Inventory Management", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(side="left")
        
        control_frame = tk.Frame(title_frame, bg=self.colors['content_bg'])
        control_frame.pack(side="right")
        
        add_btn = tk.Button(control_frame, text="➕ Add Item", 
                           bg=self.colors['success'], fg="white",
                           font=("Arial", 11, "bold"), padx=20, pady=10,
                           command=self.add_inventory_dialog)
        add_btn.pack(side="left", padx=5)
        
        refresh_btn = tk.Button(control_frame, text="🔄 Refresh", 
                               bg=self.colors['info'], fg="white",
                               font=("Arial", 11, "bold"), padx=20, pady=10,
                               command=self.load_inventory_data)
        refresh_btn.pack(side="left", padx=5)
        
        # Inventory Table
        table_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        table_frame.pack(fill="both", expand=True, padx=30, pady=10)
        
        # Create treeview
        columns = ("ID", "Name", "Type", "Quantity", "Price", "Supplier", "Expiration", "Min Stock")
        self.inventory_tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        
        # Define column headings
        for col in columns:
            self.inventory_tree.heading(col, text=col)
            self.inventory_tree.column(col, anchor="center", width=100)
        
        # Add scrollbars
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.inventory_tree.yview)
        v_scrollbar.pack(side="right", fill="y")
        self.inventory_tree.configure(yscrollcommand=v_scrollbar.set)
        
        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal", command=self.inventory_tree.xview)
        h_scrollbar.pack(side="bottom", fill="x")
        self.inventory_tree.configure(xscrollcommand=h_scrollbar.set)
        
        self.inventory_tree.pack(fill="both", expand=True)
        
        # Load inventory data
        self.load_inventory_data()
        
        # Action Buttons
        action_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        action_frame.pack(fill="x", padx=30, pady=10)
        
        view_btn = tk.Button(action_frame, text="👁️ View Details", 
                            bg=self.colors['secondary'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.view_inventory_details)
        view_btn.pack(side="left", padx=5)
        
        edit_btn = tk.Button(action_frame, text="✏️ Edit Item", 
                            bg=self.colors['warning'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.edit_inventory_dialog)
        edit_btn.pack(side="left", padx=5)
        
        delete_btn = tk.Button(action_frame, text="🗑️ Delete Item", 
                              bg=self.colors['accent'], fg="white",
                            font=("Arial", 11), padx=15, pady=8,
                            command=self.delete_inventory_item)
        delete_btn.pack(side="left", padx=5)
        
        restock_btn = tk.Button(action_frame, text="📦 Restock", 
                               bg=self.colors['info'], fg="white",
                               font=("Arial", 11), padx=15, pady=8,
                               command=self.restock_inventory)
        restock_btn.pack(side="left", padx=5)
    
    def load_inventory_data(self):
        """Load inventory data from database into treeview in proper ID order."""
        if hasattr(self, 'inventory_tree'):
            for item in self.inventory_tree.get_children():
                self.inventory_tree.delete(item)
        
        inventory = self.db.get_all_inventory()
        if inventory:
            for item in inventory:
                # Format price with peso sign
                price = f"₱{item[4]:,.2f}"
                expiration = item[6] if item[6] else "N/A"
                self.inventory_tree.insert("", "end", values=(
                    item[0], item[1], item[2], item[3], price, 
                    item[5] if item[5] else "N/A", expiration, item[7]
                ))
        else:
            self.inventory_tree.insert("", "end", values=("No inventory items", "", "", "", "", "", "", ""))
    
    def add_inventory_dialog(self):
        """Open dialog to add a new inventory item."""
        dialog = tk.Toplevel(self.root)
        dialog.title("Add New Inventory Item")
        dialog.geometry("500x550")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text="📦 Add New Inventory Item", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        labels = ["Item Name:", "Type:", "Quantity:", "Price:", "Supplier:", "Expiration Date (YYYY-MM-DD):", "Minimum Stock:"]
        entries = []
        
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
            entry.grid(row=i, column=1, pady=10, padx=10)
            entries.append(entry)
        
        # Set default values
        entries[1].insert(0, "Medication")
        entries[2].insert(0, "10")
        entries[3].insert(0, "0.00")
        entries[6].insert(0, "5")
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_inventory():
            """Save inventory item data to database."""
            name = entries[0].get()
            item_type = entries[1].get()
            quantity = entries[2].get()
            price = entries[3].get()
            supplier = entries[4].get()
            expiration = entries[5].get()
            min_stock = entries[6].get()
            
            if name and item_type and quantity and price:
                try:
                    quantity_int = int(quantity)
                    price_float = float(price)
                    min_stock_int = int(min_stock) if min_stock else 5
                    
                    # Validate expiration date format if provided
                    if expiration:
                        try:
                            datetime.strptime(expiration, "%Y-%m-%d")
                        except ValueError:
                            messagebox.showerror("Error", "Invalid date format! Use YYYY-MM-DD.")
                            return
                    
                    # Add inventory item to database
                    self.db.add_inventory_item(name, item_type, quantity_int, price_float, 
                                              supplier if supplier else None, 
                                              expiration if expiration else None, 
                                              min_stock_int)
                    
                    # Add activity record
                    self.db.add_record(f"Inventory added: {name} (Qty: {quantity_int})")
                    
                    # Refresh inventory display
                    self.load_inventory_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Inventory item '{name}' added successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Name, Type, Quantity, and Price are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Item", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_inventory)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def view_inventory_details(self):
        """View details of selected inventory item."""
        if not hasattr(self, 'inventory_tree'):
            messagebox.showerror("Error", "Inventory table not loaded!")
            return
        
        selection = self.inventory_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an inventory item first!")
            return
        
        item = self.inventory_tree.item(selection[0])
        values = item['values']
        item_id = int(values[0])
        
        # Get detailed inventory information
        inventory_item = self.db.get_inventory_item_by_id(item_id)
        if not inventory_item:
            messagebox.showerror("Error", "Inventory item not found!")
            return
        
        # Create details dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Inventory Details - {inventory_item[1]}")
        dialog.geometry("500x450")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        
        title = tk.Label(dialog, text=f"📦 Inventory Details: {inventory_item[1]}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        details_frame = tk.Frame(dialog, bg="white")
        details_frame.pack(fill="both", expand=True, padx=40)
        
        # Check stock status
        quantity = inventory_item[3]
        min_stock = inventory_item[7]
        stock_status = "✅ Sufficient" if quantity > min_stock else "⚠️ Low Stock" if quantity > 0 else "❌ Out of Stock"
        
        details_text = f"""
        🆔 Item ID: {inventory_item[0]}
        📦 Name: {inventory_item[1]}
        🏷️ Type: {inventory_item[2]}
        📊 Quantity: {quantity} units
        💰 Price: ₱{inventory_item[4]:,.2f}
        🏢 Supplier: {inventory_item[5] if inventory_item[5] else 'N/A'}
        📅 Expiration: {inventory_item[6] if inventory_item[6] else 'N/A'}
        📈 Min Stock: {min_stock} units
        🔍 Status: {stock_status}
        """
        
        details_label = tk.Label(details_frame, text=details_text, 
                                font=("Arial", 12), bg="white", justify="left")
        details_label.pack(anchor="w", pady=20)
        
        close_btn = tk.Button(dialog, text="Close", 
                             font=("Arial", 12, "bold"),
                             bg=self.colors['primary'], fg="white",
                             padx=30, pady=10, command=dialog.destroy)
        close_btn.pack(pady=20)
    
    def edit_inventory_dialog(self):
        """Edit an existing inventory item."""
        if not hasattr(self, 'inventory_tree'):
            messagebox.showerror("Error", "Inventory table not loaded!")
            return
        
        selection = self.inventory_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an inventory item first!")
            return
        
        item = self.inventory_tree.item(selection[0])
        values = item['values']
        item_id = int(values[0])
        
        # Get current inventory data
        inventory_item = self.db.get_inventory_item_by_id(item_id)
        if not inventory_item:
            messagebox.showerror("Error", "Inventory item not found!")
            return
        
        # Create edit dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Edit Inventory Item - {inventory_item[1]}")
        dialog.geometry("500x550")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text=f"✏️ Edit Inventory Item: {inventory_item[1]}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        labels = ["Item Name:", "Type:", "Quantity:", "Price:", "Supplier:", "Expiration Date (YYYY-MM-DD):", "Minimum Stock:"]
        entries = []
        
        # Current values
        current_values = [
            inventory_item[1],  # name
            inventory_item[2],  # type
            str(inventory_item[3]),  # quantity
            str(inventory_item[4]),  # price
            inventory_item[5] if inventory_item[5] else "",  # supplier
            inventory_item[6] if inventory_item[6] else "",  # expiration
            str(inventory_item[7])  # min_stock
        ]
        
        for i, (label_text, current_value) in enumerate(zip(labels, current_values)):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
            entry.grid(row=i, column=1, pady=10, padx=10)
            entry.insert(0, current_value)
            entries.append(entry)
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_edited_inventory():
            """Save edited inventory item data to database."""
            name = entries[0].get()
            item_type = entries[1].get()
            quantity = entries[2].get()
            price = entries[3].get()
            supplier = entries[4].get()
            expiration = entries[5].get()
            min_stock = entries[6].get()
            
            if name and item_type and quantity and price:
                try:
                    quantity_int = int(quantity)
                    price_float = float(price)
                    min_stock_int = int(min_stock) if min_stock else 5
                    
                    # Validate expiration date format if provided
                    if expiration:
                        try:
                            datetime.strptime(expiration, "%Y-%m-%d")
                        except ValueError:
                            messagebox.showerror("Error", "Invalid date format! Use YYYY-MM-DD.")
                            return
                    
                    # Update inventory item in database
                    self.db.update_inventory_item(item_id, name, item_type, quantity_int, price_float, 
                                                 supplier if supplier else None, 
                                                 expiration if expiration else None, 
                                                 min_stock_int)
                    
                    # Add activity record
                    self.db.add_record(f"Inventory updated: {name} (ID: {item_id})")
                    
                    # Refresh inventory display
                    self.load_inventory_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Inventory item '{name}' updated successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Name, Type, Quantity, and Price are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Changes", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_edited_inventory)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def delete_inventory_item(self):
        """Delete an inventory item with confirmation."""
        if not hasattr(self, 'inventory_tree'):
            messagebox.showerror("Error", "Inventory table not loaded!")
            return
        
        selection = self.inventory_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an inventory item first!")
            return
        
        item = self.inventory_tree.item(selection[0])
        values = item['values']
        item_id = int(values[0])
        item_name = values[1]
        
        if messagebox.askyesno("Confirm Deletion", 
                              f"Are you sure you want to delete inventory item '{item_name}' (ID: {item_id})?"):
            
            # Delete from database
            self.db.delete_inventory_item(item_id)
            
            # Add activity record
            self.db.add_record(f"Inventory deleted: {item_name} (ID: {item_id})")
            
            # Refresh display
            self.load_inventory_data()
            
            messagebox.showinfo("Success", f"Inventory item '{item_name}' deleted successfully!")
    
    def restock_inventory(self):
        """Restock an inventory item."""
        if not hasattr(self, 'inventory_tree'):
            messagebox.showerror("Error", "Inventory table not loaded!")
            return
        
        selection = self.inventory_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an inventory item first!")
            return
        
        item = self.inventory_tree.item(selection[0])
        values = item['values']
        item_id = int(values[0])
        item_name = values[1]
        
        # Get current inventory data
        inventory_item = self.db.get_inventory_item_by_id(item_id)
        if not inventory_item:
            messagebox.showerror("Error", "Inventory item not found!")
            return
        
        # Create restock dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Restock Inventory - {item_name}")
        dialog.geometry("400x250")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text=f"📦 Restock: {item_name}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        lbl = tk.Label(form_frame, text=f"Current Quantity: {inventory_item[3]} units", 
                      font=("Arial", 11), bg="white")
        lbl.pack(pady=10)
        
        lbl2 = tk.Label(form_frame, text="Add Quantity:", font=("Arial", 11), bg="white")
        lbl2.pack(pady=10)
        
        quantity_entry = tk.Entry(form_frame, font=("Arial", 11), width=20)
        quantity_entry.pack(pady=10)
        quantity_entry.insert(0, "10")
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=20)
        
        def perform_restock():
            """Perform the restock operation."""
            try:
                add_quantity = int(quantity_entry.get())
                if add_quantity <= 0:
                    messagebox.showerror("Error", "Quantity must be greater than 0!")
                    return
                
                new_quantity = inventory_item[3] + add_quantity
                
                # Update inventory item in database
                self.db.update_inventory_item(
                    item_id, inventory_item[1], inventory_item[2], new_quantity,
                    inventory_item[4], inventory_item[5], inventory_item[6], inventory_item[7]
                )
                
                # Add activity record
                self.db.add_record(f"Inventory restocked: {item_name} (+{add_quantity}, Total: {new_quantity})")
                
                # Refresh inventory display
                self.load_inventory_data()
                
                dialog.destroy()
                messagebox.showinfo("Success", f"Restocked '{item_name}' with {add_quantity} units. New total: {new_quantity} units!")
            except ValueError:
                messagebox.showerror("Error", "Please enter a valid number!")
        
        restock_btn = tk.Button(button_frame, text="📦 Restock", 
                               font=("Arial", 12, "bold"),
                               bg=self.colors['success'], fg="white",
                               padx=30, pady=10, command=perform_restock)
        restock_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=10, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def show_records(self):
        """Display the activity records section."""
        self.clear_content()
        self.status_bar.config(text="Activity Records")
        
        # Title and Controls
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))
        
        title = tk.Label(title_frame, text="📋 Activity Records", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(side="left")
        
        control_frame = tk.Frame(title_frame, bg=self.colors['content_bg'])
        control_frame.pack(side="right")
        
        refresh_btn = tk.Button(control_frame, text="🔄 Refresh", 
                               bg=self.colors['info'], fg="white",
                               font=("Arial", 11, "bold"), padx=20, pady=10,
                               command=self.load_records_data)
        refresh_btn.pack(side="left", padx=5)
        
        clear_btn = tk.Button(control_frame, text="🗑️ Clear All", 
                             bg=self.colors['accent'], fg="white",
                             font=("Arial", 11, "bold"), padx=20, pady=10,
                             command=self.clear_all_records)
        clear_btn.pack(side="left", padx=5)
        
        # Records Table
        table_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        table_frame.pack(fill="both", expand=True, padx=30, pady=10)
        
        # Create treeview
        columns = ("ID", "Timestamp", "Activity Record")
        self.records_tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=20)
        
        # Define column headings
        self.records_tree.heading("ID", text="ID")
        self.records_tree.heading("Timestamp", text="Timestamp")
        self.records_tree.heading("Activity Record", text="Activity Record")
        
        # Set column widths
        self.records_tree.column("ID", width=50, anchor="center")
        self.records_tree.column("Timestamp", width=150, anchor="center")
        self.records_tree.column("Activity Record", width=600, anchor="w")
        
        # Add scrollbars
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.records_tree.yview)
        v_scrollbar.pack(side="right", fill="y")
        self.records_tree.configure(yscrollcommand=v_scrollbar.set)
        
        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal", command=self.records_tree.xview)
        h_scrollbar.pack(side="bottom", fill="x")
        self.records_tree.configure(xscrollcommand=h_scrollbar.set)
        
        self.records_tree.pack(fill="both", expand=True)
        
        # Load records data
        self.load_records_data()
        
        # Action Buttons
        action_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        action_frame.pack(fill="x", padx=30, pady=10)
        
        delete_btn = tk.Button(action_frame, text="🗑️ Delete Selected", 
                              bg=self.colors['accent'], fg="white",
                              font=("Arial", 11), padx=15, pady=8,
                              command=self.delete_selected_record)
        delete_btn.pack(side="left", padx=5)
    
    def load_records_data(self):
        """Load activity records data from database into treeview in proper order."""
        if hasattr(self, 'records_tree'):
            for item in self.records_tree.get_children():
                self.records_tree.delete(item)
        
        records = self.db.get_all_records()
        if records:
            for record in records:
                self.records_tree.insert("", "end", values=record)
        else:
            self.records_tree.insert("", "end", values=("No records found", "", ""))
    
    def clear_all_records(self):
        """Clear all activity records with confirmation."""
        if messagebox.askyesno("Confirm Clear All", 
                              "Are you sure you want to delete ALL activity records?\n\n⚠️ This action cannot be undone!"):
            
            # Clear all records from database
            conn = self.db.connect()
            cursor = conn.cursor()
            cursor.execute("DELETE FROM records")
            conn.commit()
            conn.close()
            
            # Add a record about clearing (this will be the only record)
            self.db.add_record("System: All activity records cleared")
            
            # Refresh records display
            self.load_records_data()
            
            messagebox.showinfo("Success", "All activity records cleared!")
    
    def delete_selected_record(self):
        """Delete selected activity record."""
        if not hasattr(self, 'records_tree'):
            messagebox.showerror("Error", "Records table not loaded!")
            return
        
        selection = self.records_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a record first!")
            return
        
        item = self.records_tree.item(selection[0])
        values = item['values']
        record_id = int(values[0])
        
        if messagebox.askyesno("Confirm Deletion", 
                              f"Are you sure you want to delete this record (ID: {record_id})?"):
            
            # Delete from database
            self.db.delete_record(record_id)
            
            # Refresh records display
            self.load_records_data()
            
            messagebox.showinfo("Success", f"Record #{record_id} deleted successfully!")
    
    # ============= APPOINTMENTS SECTION =============
    def show_appointments(self):
        """Display the appointments section with list and controls."""
        self.clear_content()
        self.status_bar.config(text="Appointment System")
        
        # ============= TITLE AND CONTROLS =============
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))
        
        title = tk.Label(title_frame, text="📅 Appointment System", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(side="left")
        
        # Control buttons frame
        control_frame = tk.Frame(title_frame, bg=self.colors['content_bg'])
        control_frame.pack(side="right")
        
        add_btn = tk.Button(control_frame, text="➕ New Appointment", 
                           bg=self.colors['success'], fg="white",
                           font=("Arial", 11, "bold"),
                           padx=20, pady=10,
                           command=self.show_add_appointment_form)
        add_btn.pack(side="left", padx=5)
        
        # ============= APPOINTMENTS LIST =============
        list_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        list_frame.pack(fill="both", expand=True, padx=30, pady=10)
        
        # Create treeview container
        tree_frame = tk.Frame(list_frame, bg=self.colors['content_bg'])
        tree_frame.pack(fill="both", expand=True)
        
        # Add scrollbars
        v_scrollbar = ttk.Scrollbar(tree_frame)
        v_scrollbar.pack(side="right", fill="y")
        
        h_scrollbar = ttk.Scrollbar(tree_frame, orient="horizontal")
        h_scrollbar.pack(side="bottom", fill="x")
        
        # Create treeview for appointments
        columns = ("ID", "Pet", "Date", "Time", "Reason", "Status", "Amount")
        self.appointments_tree = ttk.Treeview(tree_frame, columns=columns, show="headings", height=15,
                                            yscrollcommand=v_scrollbar.set,
                                            xscrollcommand=h_scrollbar.set)
        
        # Configure scrollbars
        v_scrollbar.config(command=self.appointments_tree.yview)
        h_scrollbar.config(command=self.appointments_tree.xview)
        
        # Define column headings and widths
        column_widths = [50, 120, 100, 80, 150, 100, 80]
        for col, width in zip(columns, column_widths):
            self.appointments_tree.heading(col, text=col)
            self.appointments_tree.column(col, width=width, anchor="center")
        
        self.appointments_tree.pack(fill="both", expand=True)
        
        # Load appointments data from database
        self.load_appointments_data()
        
        # ============= ACTION BUTTONS =============
        action_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        action_frame.pack(fill="x", padx=30, pady=10)

        # Action buttons for appointment management
        add_btn2 = tk.Button(action_frame, text="➕ Add", bg=self.colors['success'], fg="white",
                            font=("Arial", 10, "bold"), padx=15, pady=8, command=self.show_add_appointment_form)
        add_btn2.pack(side="left", padx=5)

        view_btn = tk.Button(action_frame, text="👁️ View Details", bg=self.colors['secondary'], fg="white",
                            font=("Arial", 10), padx=15, pady=8, command=self.view_appointment_details)
        view_btn.pack(side="left", padx=5)

        edit_btn = tk.Button(action_frame, text="📝 Edit", bg=self.colors['secondary'], fg="white",
                            font=("Arial", 10), padx=15, pady=8, command=self.edit_appointment)
        edit_btn.pack(side="left", padx=5)

        delete_btn = tk.Button(action_frame, text="🗑️ Delete", bg=self.colors['accent'], fg="white",
                              font=("Arial", 10), padx=15, pady=8, command=self.delete_appointment)
        delete_btn.pack(side="left", padx=5)

        cancel_btn = tk.Button(action_frame, text="❌ Cancel", bg=self.colors['secondary'], fg="white",
                              font=("Arial", 10), padx=15, pady=8, command=self.cancel_appointment)
        cancel_btn.pack(side="left", padx=5)

        complete_btn = tk.Button(action_frame, text="✅ Complete", bg=self.colors['secondary'], fg="white",
                                 font=("Arial", 10), padx=15, pady=8, command=self.complete_appointment)
        complete_btn.pack(side="left", padx=5)

        bill_btn = tk.Button(action_frame, text="💰 Add Bill", bg=self.colors['secondary'], fg="white",
                            font=("Arial", 10), padx=15, pady=8, command=self.add_appointment_bill)
        bill_btn.pack(side="left", padx=5)
    
    def load_appointments_data(self):
        """Load appointments from database into treeview in proper ID order."""
        # Clear existing data from treeview
        if hasattr(self, 'appointments_tree'):
            for item in self.appointments_tree.get_children():
                self.appointments_tree.delete(item)
        
        # Load appointments from database with pet names
        conn = self.db.connect()
        cursor = conn.cursor()
        
        cursor.execute('''
        SELECT a.id, p.name, a.date, a.time, a.reason, a.status, a.billing_amount
        FROM appointments a
        LEFT JOIN pets p ON a.pet_id = p.id
        ORDER BY a.id
        ''')
        
        appointments = cursor.fetchall()
        conn.close()
        
        # Insert appointments into treeview
        if appointments:
            for app in appointments:
                self.appointments_tree.insert("", "end", values=(
                    app[0],  # ID
                    app[1] if app[1] else "Unknown",  # Pet name
                    app[2],  # Date
                    app[3],  # Time
                    app[4],  # Reason
                    app[5],  # Status
                    f"{app[6]:,.2f}" if app[6] else "0.00"  # Amount without currency sign
                ))
        else:
            self.appointments_tree.insert("", "end", values=("No appointments", "", "", "", "", "", ""))
    
    def show_add_appointment_form(self):
        """Show form to add new appointment."""
        dialog = tk.Toplevel(self.root)
        dialog.title("Add New Appointment")
        dialog.geometry("500x550")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text="📅 Add New Appointment", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        # Get pets and vets for dropdowns
        pets = self.db.get_pets_for_dropdown()
        vets = self.db.get_vets_for_dropdown()
        
        labels = ["Pet:", "Veterinarian:", "Date (YYYY-MM-DD):", "Time (HH:MM):", "Reason:", "Status:", "Billing Amount:"]
        entries = []
        
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            if label_text == "Pet:":
                # Pet dropdown
                pet_var = tk.StringVar()
                pet_dropdown = ttk.Combobox(form_frame, textvariable=pet_var, width=32)
                pet_dropdown['values'] = [f"{pet[0]}: {pet[1]}" for pet in pets]
                if pets:
                    pet_dropdown.current(0)
                pet_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(pet_var)
            elif label_text == "Veterinarian:":
                # Vet dropdown with specialization
                vet_var = tk.StringVar()
                vet_dropdown = ttk.Combobox(form_frame, textvariable=vet_var, width=32)
                
                # Create display text with specialization
                vet_options = []
                for vet in vets:
                    if len(vet) >= 3:  # Check if specialization exists
                        vet_options.append(f"{vet[1]} ({vet[2]}) - ID: {vet[0]}")
                    else:
                        vet_options.append(f"{vet[1]} - ID: {vet[0]}")
                
                vet_dropdown['values'] = vet_options
                if vets:
                    vet_dropdown.current(0)
                vet_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(vet_var)
            elif label_text == "Status:":
                # Status dropdown
                status_var = tk.StringVar(value="Scheduled")
                status_dropdown = ttk.Combobox(form_frame, textvariable=status_var, width=32)
                status_dropdown['values'] = ["Scheduled", "Confirmed", "In Progress", "Completed", "Cancelled"]
                status_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(status_var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i, column=1, pady=10, padx=10)
                entries.append(entry)
        
        # Set default values
        entries[2].insert(0, datetime.now().strftime("%Y-%m-%d"))
        entries[3].insert(0, "09:00")
        entries[5].set("Scheduled")
        entries[6].insert(0, "0.00")
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_appointment():
            """Save appointment data to database."""
            pet_str = entries[0].get()
            vet_str = entries[1].get()
            date = entries[2].get()
            time = entries[3].get()
            reason = entries[4].get()
            status = entries[5].get()
            amount = entries[6].get()
            
            if pet_str and date and time and reason:
                try:
                    # Extract IDs from dropdown strings
                    pet_id = int(pet_str.split(":")[0]) if ":" in pet_str else int(pet_str)
                    
                    # Extract vet ID from the formatted string
                    vet_id = None
                    if vet_str:
                        # Format: "Dr. Name (Specialization) - ID: 1"
                        if " - ID: " in vet_str:
                            vet_id = int(vet_str.split(" - ID: ")[1])
                        else:
                            # Fallback for old format
                            vet_id = int(vet_str.split(":")[0]) if ":" in vet_str else int(vet_str)
                    
                    # Validate date and time formats
                    datetime.strptime(date, "%Y-%m-%d")
                    datetime.strptime(time, "%H:%M")
                    
                    amount_float = float(amount) if amount else 0.0
                    
                    # Add appointment to database
                    appointment_id = self.db.add_appointment(pet_id, vet_id, date, time, reason, status, amount_float)
                    
                    # Add activity record
                    self.db.add_record(f"Appointment added: ID {appointment_id} for {date} {time}")
                    
                    # Refresh appointments display
                    self.load_appointments_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Appointment #{appointment_id} added successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
                except Exception as e:
                    messagebox.showerror("Error", f"Error: {str(e)}")
            else:
                messagebox.showerror("Error", "Pet, Date, Time, and Reason are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Appointment", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_appointment)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def view_appointment_details(self):
        """View details of selected appointment."""
        if not hasattr(self, 'appointments_tree'):
            messagebox.showerror("Error", "Appointments table not loaded!")
            return
        
        selection = self.appointments_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an appointment first!")
            return
        
        item = self.appointments_tree.item(selection[0])
        values = item['values']
        appointment_id = int(values[0])
        
        # Get detailed appointment information
        appointment = self.db.get_appointment_details(appointment_id)
        if not appointment:
            messagebox.showerror("Error", "Appointment not found!")
            return
        
        # Create details dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Appointment Details - #{appointment_id}")
        dialog.geometry("500x450")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        
        title = tk.Label(dialog, text=f"📅 Appointment Details: #{appointment_id}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        details_frame = tk.Frame(dialog, bg="white")
        details_frame.pack(fill="both", expand=True, padx=40)
        
        details_text = f"""
        🆔 Appointment ID: {appointment[0]}
        🐕 Pet: {appointment[8] if appointment[8] else 'Unknown'}
        👤 Owner: {appointment[9] if appointment[9] else 'Unknown'}
        👨‍⚕️ Veterinarian: {appointment[10] if appointment[10] else 'Not assigned'}
        🎯 Specialization: {appointment[11] if appointment[11] else 'Not specified'}
        📅 Date: {appointment[3]}
        ⏰ Time: {appointment[4]}
        📝 Reason: {appointment[5]}
        🟢 Status: {appointment[6]}
        💰 Billing Amount: ₱{appointment[7]:,.2f}
        """
        
        details_label = tk.Label(details_frame, text=details_text, 
                                font=("Arial", 12), bg="white", justify="left")
        details_label.pack(anchor="w", pady=20)
        
        close_btn = tk.Button(dialog, text="Close", 
                             font=("Arial", 12, "bold"),
                             bg=self.colors['primary'], fg="white",
                             padx=30, pady=10, command=dialog.destroy)
        close_btn.pack(pady=20)
    
    def edit_appointment(self):
        """Edit selected appointment."""
        if not hasattr(self, 'appointments_tree'):
            messagebox.showerror("Error", "Appointments table not loaded!")
            return
        
        selection = self.appointments_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an appointment first!")
            return
        
        item = self.appointments_tree.item(selection[0])
        values = item['values']
        appointment_id = int(values[0])
        
        # Get current appointment data
        appointment = self.db.get_appointment_details(appointment_id)
        if not appointment:
            messagebox.showerror("Error", "Appointment not found!")
            return
        
        # Create edit dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Edit Appointment - #{appointment_id}")
        dialog.geometry("500x550")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text=f"✏️ Edit Appointment: #{appointment_id}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        # Get pets and vets for dropdowns
        pets = self.db.get_pets_for_dropdown()
        vets = self.db.get_vets_for_dropdown()
        
        labels = ["Pet:", "Veterinarian:", "Date (YYYY-MM-DD):", "Time (HH:MM):", "Reason:", "Status:", "Billing Amount:"]
        entries = []
        
        # Current values
        current_pet = f"{appointment[1]}"  # pet_id
        current_vet = f"{appointment[2]}" if appointment[2] else ""  # vet_id
        current_values = [
            current_pet,
            current_vet,
            appointment[3],  # date
            appointment[4],  # time
            appointment[5],  # reason
            appointment[6],  # status
            str(appointment[7])  # billing_amount
        ]
        
        for i, (label_text, current_value) in enumerate(zip(labels, current_values)):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            if label_text == "Pet:":
                # Pet dropdown
                pet_var = tk.StringVar(value=current_value)
                pet_dropdown = ttk.Combobox(form_frame, textvariable=pet_var, width=32)
                pet_dropdown['values'] = [f"{pet[0]}: {pet[1]}" for pet in pets]
                pet_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(pet_var)
            elif label_text == "Veterinarian:":
                # Vet dropdown with specialization
                vet_var = tk.StringVar(value=current_value)
                vet_dropdown = ttk.Combobox(form_frame, textvariable=vet_var, width=32)
                
                # Create display text with specialization
                vet_options = []
                for vet in vets:
                    if len(vet) >= 3:  # Check if specialization exists
                        vet_options.append(f"{vet[1]} ({vet[2]}) - ID: {vet[0]}")
                    else:
                        vet_options.append(f"{vet[1]} - ID: {vet[0]}")
                
                vet_dropdown['values'] = vet_options
                
                # Set current value if it exists
                if current_value and current_value != "None" and current_value != "":
                    # Find matching vet in options
                    for option in vet_options:
                        if f"ID: {current_value}" in option:
                            vet_var.set(option)
                            break
                
                vet_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(vet_var)
            elif label_text == "Status:":
                # Status dropdown
                status_var = tk.StringVar(value=current_value)
                status_dropdown = ttk.Combobox(form_frame, textvariable=status_var, width=32)
                status_dropdown['values'] = ["Scheduled", "Confirmed", "In Progress", "Completed", "Cancelled"]
                status_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(status_var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i, column=1, pady=10, padx=10)
                entry.insert(0, current_value)
                entries.append(entry)
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_edited_appointment():
            """Save edited appointment data to database."""
            pet_str = entries[0].get()
            vet_str = entries[1].get()
            date = entries[2].get()
            time = entries[3].get()
            reason = entries[4].get()
            status = entries[5].get()
            amount = entries[6].get()
            
            if pet_str and date and time and reason:
                try:
                    # Extract IDs from dropdown strings
                    pet_id = int(pet_str.split(":")[0]) if ":" in pet_str else int(pet_str)
                    
                    # Extract vet ID from the formatted string
                    vet_id = None
                    if vet_str:
                        # Format: "Dr. Name (Specialization) - ID: 1"
                        if " - ID: " in vet_str:
                            vet_id = int(vet_str.split(" - ID: ")[1])
                        else:
                            # Fallback for old format or if vet_str is just an ID
                            try:
                                vet_id = int(vet_str.split(":")[0]) if ":" in vet_str else int(vet_str)
                            except ValueError:
                                vet_id = None
                    
                    # Validate date and time formats
                    datetime.strptime(date, "%Y-%m-%d")
                    datetime.strptime(time, "%H:%M")
                    
                    amount_float = float(amount) if amount else 0.0
                    
                    # Update appointment in database
                    self.db.update_appointment(appointment_id, pet_id, vet_id, date, time, reason, status, amount_float)
                    
                    # Add activity record
                    self.db.add_record(f"Appointment updated: ID {appointment_id}")
                    
                    # Refresh appointments display
                    self.load_appointments_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Appointment #{appointment_id} updated successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
                except Exception as e:
                    messagebox.showerror("Error", f"Error: {str(e)}")
            else:
                messagebox.showerror("Error", "Pet, Date, Time, and Reason are required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Changes", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_edited_appointment)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def delete_appointment(self):
        """Delete selected appointment with confirmation."""
        if not hasattr(self, 'appointments_tree'):
            messagebox.showerror("Error", "Appointments table not loaded!")
            return
        
        selection = self.appointments_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an appointment first!")
            return
        
        item = self.appointments_tree.item(selection[0])
        values = item['values']
        appointment_id = int(values[0])
        
        if messagebox.askyesno("Confirm Deletion", 
                              f"Are you sure you want to delete appointment #{appointment_id}?\n\n⚠️ This will also delete related billing records!"):
            
            # Delete from database
            self.db.delete_appointment(appointment_id)
            
            # Add activity record
            self.db.add_record(f"Appointment deleted: ID {appointment_id}")
            
            # Refresh appointments display
            self.load_appointments_data()
            
            messagebox.showinfo("Success", f"Appointment #{appointment_id} deleted successfully!")
    
    def cancel_appointment(self):
        """Cancel selected appointment."""
        if not hasattr(self, 'appointments_tree'):
            messagebox.showerror("Error", "Appointments table not loaded!")
            return
        
        selection = self.appointments_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an appointment first!")
            return
        
        item = self.appointments_tree.item(selection[0])
        values = item['values']
        appointment_id = int(values[0])
        
        if messagebox.askyesno("Confirm Cancellation", 
                              f"Are you sure you want to cancel appointment #{appointment_id}?"):
            
            # Update appointment status to Cancelled
            conn = self.db.connect()
            cursor = conn.cursor()
            cursor.execute("UPDATE appointments SET status = 'Cancelled' WHERE id = ?", (appointment_id,))
            conn.commit()
            conn.close()
            
            # Add activity record
            self.db.add_record(f"Appointment cancelled: ID {appointment_id}")
            
            # Refresh appointments display
            self.load_appointments_data()
            
            messagebox.showinfo("Success", f"Appointment #{appointment_id} cancelled successfully!")
    
    def complete_appointment(self):
        """Mark selected appointment as complete."""
        if not hasattr(self, 'appointments_tree'):
            messagebox.showerror("Error", "Appointments table not loaded!")
            return
        
        selection = self.appointments_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an appointment first!")
            return
        
        item = self.appointments_tree.item(selection[0])
        values = item['values']
        appointment_id = int(values[0])
        
        if messagebox.askyesno("Confirm Completion", 
                              f"Are you sure you want to mark appointment #{appointment_id} as complete?"):
            
            # Update appointment status to Completed
            conn = self.db.connect()
            cursor = conn.cursor()
            cursor.execute("UPDATE appointments SET status = 'Completed' WHERE id = ?", (appointment_id,))
            conn.commit()
            conn.close()
            
            # Add activity record
            self.db.add_record(f"Appointment completed: ID {appointment_id}")
            
            # Refresh appointments display
            self.load_appointments_data()
            
            messagebox.showinfo("Success", f"Appointment #{appointment_id} marked as complete!")
    
    def add_appointment_bill(self):
        """Add bill for selected appointment."""
        if not hasattr(self, 'appointments_tree'):
            messagebox.showerror("Error", "Appointments table not loaded!")
            return
        
        selection = self.appointments_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select an appointment first!")
            return
        
        item = self.appointments_tree.item(selection[0])
        values = item['values']
        appointment_id = int(values[0])
        
        # Get appointment details
        appointment = self.db.get_appointment_details(appointment_id)
        if not appointment:
            messagebox.showerror("Error", "Appointment not found!")
            return
        
        # Create billing dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Add Bill for Appointment #{appointment_id}")
        dialog.geometry("500x450")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()
        
        title = tk.Label(dialog, text=f"💰 Add Bill for Appointment #{appointment_id}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        # Display appointment info
        info_label = tk.Label(form_frame, 
                             text=f"Pet: {appointment[8] if appointment[8] else 'Unknown'}\nDate: {appointment[3]} {appointment[4]}\nReason: {appointment[5]}",
                             font=("Arial", 11), bg="white", justify="left")
        info_label.grid(row=0, column=0, columnspan=2, pady=10, sticky="w")
        
        labels = ["Total Amount:", "Status:", "Payment Date (YYYY-MM-DD):", "Items/Services:"]
        entries = []
        
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i+1, column=0, sticky="w", pady=10)
            
            if label_text == "Status:":
                # Status dropdown
                status_var = tk.StringVar(value="Pending")
                status_dropdown = ttk.Combobox(form_frame, textvariable=status_var, width=32)
                status_dropdown['values'] = ["Pending", "Paid", "Partial", "Cancelled"]
                status_dropdown.grid(row=i+1, column=1, pady=10, padx=10)
                entries.append(status_var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i+1, column=1, pady=10, padx=10)
                entries.append(entry)
        
        # Set default values
        entries[0].insert(0, str(appointment[7]))  # billing_amount from appointment
        entries[2].insert(0, datetime.now().strftime("%Y-%m-%d"))
        
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_bill():
            """Save billing data to database."""
            total_amount = entries[0].get()
            status = entries[1].get()
            payment_date = entries[2].get()
            items = entries[3].get()
            
            if total_amount:
                try:
                    total_amount_float = float(total_amount)
                    
                    # Validate payment date format if provided
                    if payment_date:
                        try:
                            datetime.strptime(payment_date, "%Y-%m-%d")
                        except ValueError:
                            messagebox.showerror("Error", "Invalid date format! Use YYYY-MM-DD.")
                            return
                    
                    # Add billing to database
                    self.db.add_billing(appointment_id, appointment[1], total_amount_float, status, items, 
                                       payment_date if payment_date else None)
                    
                    # Update appointment billing amount
                    conn = self.db.connect()
                    cursor = conn.cursor()
                    cursor.execute("UPDATE appointments SET billing_amount = ? WHERE id = ?", 
                                 (total_amount_float, appointment_id))
                    conn.commit()
                    conn.close()
                    
                    # Add activity record
                    self.db.add_record(f"Bill added for appointment #{appointment_id}: ₱{total_amount_float:,.2f}")
                    
                    # Refresh appointments display
                    self.load_appointments_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", f"Bill added for appointment #{appointment_id}!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Total Amount is required!")
        
        save_btn = tk.Button(button_frame, text="💾 Save Bill", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12, command=save_bill)
        save_btn.pack(side="left", padx=15)
        
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12, command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def show_billing(self):
        """Display the billing section with records and controls."""
        self.clear_content()
        self.status_bar.config(text="Billing")

        # ============= TITLE =============
        title_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        title_frame.pack(fill="x", padx=30, pady=(20, 10))

        title = tk.Label(title_frame, text="💰 Billing Section", 
                        font=("Arial", 24, "bold"), fg="white", bg=self.colors['content_bg'])
        title.pack(anchor="w")

        # ============= BILLING TABLE =============
        table_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        table_frame.pack(fill="both", expand=True, padx=30, pady=20)

        # Create treeview with scrollbars
        columns = ("ID", "Customer", "Service", "Amount", "Status", "Date", "Items")
        self.billing_tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)

        # Define column headings
        for col in columns:
            self.billing_tree.heading(col, text=col)
            self.billing_tree.column(col, anchor="center", width=100)

        # Add scrollbars
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.billing_tree.yview)
        v_scrollbar.pack(side="right", fill="y")
        self.billing_tree.configure(yscrollcommand=v_scrollbar.set)

        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal", command=self.billing_tree.xview)
        h_scrollbar.pack(side="bottom", fill="x")
        self.billing_tree.configure(xscrollcommand=h_scrollbar.set)

        self.billing_tree.pack(fill="both", expand=True)

        # Load billing data from database
        self.load_billing_data()

        # ============= ACTION BUTTONS =============
        action_frame = tk.Frame(self.content_frame, bg=self.colors['content_bg'])
        action_frame.pack(fill="x", padx=30, pady=10)

        # Billing management buttons
        add_btn = tk.Button(action_frame, text="➕ Add Billing", 
                   bg=self.colors['success'], fg="white",
                   font=("Arial", 11, "bold"),
                   padx=20, pady=10,
                   command=self.add_billing)
        add_btn.pack(side="left", padx=5)

        edit_btn = tk.Button(action_frame, text="✏️ Edit Billing", 
                    bg=self.colors['secondary'], fg="white",
                    font=("Arial", 11, "bold"),
                    padx=20, pady=10,
                    command=self.edit_billing)
        edit_btn.pack(side="left", padx=5)

        delete_btn = tk.Button(action_frame, text="🗑️ Delete Billing", 
                      bg=self.colors['accent'], fg="white",
                      font=("Arial", 11, "bold"),
                      padx=20, pady=10,
                      command=self.delete_billing)
        delete_btn.pack(side="left", padx=5)

        refresh_btn = tk.Button(action_frame, text="🔄 Refresh", 
                       bg=self.colors['info'], fg="white",
                       font=("Arial", 11, "bold"),
                       padx=20, pady=10,
                       command=self.load_billing_data)
        refresh_btn.pack(side="left", padx=5)
    
    def load_billing_data(self):
        """Load billing data from database into treeview in proper ID order."""
        # Clear existing data from treeview
        if hasattr(self, 'billing_tree'):
            for item in self.billing_tree.get_children():
                self.billing_tree.delete(item)
        
        # Load billing data with customer information
        billing_data = self.db.get_billing_with_customer_info()
        
        # Insert billing records into treeview
        if billing_data:
            for bill in billing_data:
                # Get customer name from database
                customer_name = bill[7] if bill[7] else "Unknown Customer"
                service = bill[8] if bill[8] else "General Service"
                items = bill[6] if bill[6] else f"Appointment: {service}"
                
                # Format payment date
                payment_date = bill[5]
                if payment_date and payment_date != "Not paid":
                    try:
                        # Try to format the date
                        payment_date = datetime.strptime(payment_date, "%Y-%m-%d").strftime("%Y-%m-%d")
                    except:
                        payment_date = "Not paid"
                else:
                    payment_date = "Not paid"
                
                # Format amount
                amount = f"{bill[3]:,.2f}"
                
                self.billing_tree.insert("", "end", values=(
                    bill[0],  # ID
                    customer_name,  # Customer
                    service,  # Service
                    amount,  # Amount
                    bill[4],  # Status
                    payment_date,  # Date
                    items[:50] + "..." if len(items) > 50 else items  # Truncate long items
                ))
        else:
            self.billing_tree.insert("", "end", values=("No billing records", "", "", "", "", "", ""))
    
    def add_billing(self):
        """Open dialog to add new billing entry."""
        dialog = tk.Toplevel(self.root)
        dialog.title("Add New Billing")
        dialog.geometry("500x500")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()  # Make dialog modal
        
        # Dialog title
        title = tk.Label(dialog, text="💰 New Billing Entry", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)
        
        # Form fields
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)
        
        # Get pets for dropdown
        pets = self.db.get_pets_with_owners()
        
        labels = ["Select Pet (Customer):", "Appointment ID:", "Total Amount:", "Status:", "Payment Date (YYYY-MM-DD):", "Items:"]
        entries = []  # Store entry widgets
        
        # Create form fields dynamically
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            if label_text == "Select Pet (Customer):":
                # Pet dropdown with customer name
                pet_var = tk.StringVar()
                pet_dropdown = ttk.Combobox(form_frame, textvariable=pet_var, width=32)
                
                # Create display text with pet name and owner
                pet_options = []
                for pet in pets:
                    pet_options.append(f"ID: {pet[0]} - {pet[1]} (Owner: {pet[2]})")
                
                pet_dropdown['values'] = pet_options
                if pets:
                    pet_dropdown.current(0)
                pet_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(pet_var)
            elif label_text == "Status:":
                # Status dropdown
                status_var = tk.StringVar(value="Pending")
                status_dropdown = ttk.Combobox(form_frame, textvariable=status_var, width=32)
                status_dropdown['values'] = ["Pending", "Paid", "Partial", "Cancelled"]
                status_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(status_var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i, column=1, pady=10, padx=10)
                entries.append(entry)
        
        # Set default values for form fields
        entries[3].set("Pending")
        entries[4].insert(0, datetime.now().strftime("%Y-%m-%d"))
        
        # Action buttons
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)
        
        def save_billing():
            """Save billing data to database."""
            # Get values from form fields
            pet_str = entries[0].get()
            app_id = entries[1].get()
            total_amount = entries[2].get()
            status = entries[3].get()
            payment_date = entries[4].get()
            items = entries[5].get()
            
            # Extract pet ID from dropdown string
            if not pet_str:
                messagebox.showerror("Error", "Please select a pet (customer)!")
                return
            
            try:
                # Extract pet ID from the string: "ID: X - PetName (Owner: OwnerName)"
                pet_id = int(pet_str.split("ID: ")[1].split(" - ")[0])
            except (IndexError, ValueError):
                messagebox.showerror("Error", "Invalid pet selection!")
                return
            
            # Validate required fields
            if app_id and total_amount:
                try:
                    total_amount_float = float(total_amount)
                    
                    # Validate payment date format if provided
                    if payment_date:
                        try:
                            datetime.strptime(payment_date, "%Y-%m-%d")
                        except ValueError:
                            messagebox.showerror("Error", "Invalid date format! Use YYYY-MM-DD.")
                            return
                    
                    # Add billing to database
                    self.db.add_billing(app_id, pet_id, total_amount_float, status, items, payment_date if payment_date else None)
                    
                    # Add activity record
                    self.db.add_record(f"Billing added: Pet ID {pet_id}, Amount {total_amount_float:,.2f}")
                    
                    # Refresh billing display
                    self.load_billing_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", "Billing added successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Appointment ID and Total Amount are required!")
        
        # Save button
        save_btn = tk.Button(button_frame, text="💾 Save Billing", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12,
                            command=save_billing)
        save_btn.pack(side="left", padx=15)
        
        # Cancel button
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12,
                              command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def edit_billing(self):
        """Edit an existing billing entry."""
        if not hasattr(self, 'billing_tree'):
            messagebox.showerror("Error", "Billing table not loaded!")
            return

        # Get selected item from treeview
        selection = self.billing_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a billing entry first!")
            return

        # Get billing ID from selected item
        item = self.billing_tree.item(selection[0])
        values = item['values']
        bill_id = int(values[0])

        # Retrieve billing data from database
        conn = self.db.connect()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM billing WHERE id = ?", (bill_id,))
        bill_data = cursor.fetchone()
        conn.close()

        if not bill_data:
            messagebox.showerror("Error", "Billing entry not found!")
            return
        
        # Get pet information for dropdown
        pets = self.db.get_pets_with_owners()
        
        # Get current pet info
        current_pet_id = bill_data[2]
        current_pet = None
        for pet in pets:
            if pet[0] == current_pet_id:
                current_pet = f"ID: {pet[0]} - {pet[1]} (Owner: {pet[2]})"
                break

        # Create edit dialog
        dialog = tk.Toplevel(self.root)
        dialog.title(f"Edit Billing Entry: {bill_id}")
        dialog.geometry("500x500")
        dialog.configure(bg="white")
        dialog.transient(self.root)
        dialog.grab_set()

        # Dialog title
        title = tk.Label(dialog, text=f"✏️ Edit Billing Entry: {bill_id}", 
                        font=("Arial", 16, "bold"), bg="white")
        title.pack(pady=20)

        # Form fields
        form_frame = tk.Frame(dialog, bg="white")
        form_frame.pack(fill="both", expand=True, padx=40)

        labels = ["Select Pet (Customer):", "Appointment ID:", "Total Amount:", "Status:", "Payment Date (YYYY-MM-DD):", "Items:"]
        entries = []
        
        # Create form fields
        for i, label_text in enumerate(labels):
            lbl = tk.Label(form_frame, text=label_text, font=("Arial", 11), bg="white")
            lbl.grid(row=i, column=0, sticky="w", pady=10)
            
            if label_text == "Select Pet (Customer):":
                # Pet dropdown with customer name
                pet_var = tk.StringVar()
                pet_dropdown = ttk.Combobox(form_frame, textvariable=pet_var, width=32)
                
                # Create display text with pet name and owner
                pet_options = []
                for pet in pets:
                    pet_options.append(f"ID: {pet[0]} - {pet[1]} (Owner: {pet[2]})")
                
                pet_dropdown['values'] = pet_options
                
                # Set current value if it exists
                if current_pet:
                    pet_var.set(current_pet)
                elif pets:
                    pet_dropdown.current(0)
                
                pet_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(pet_var)
            elif label_text == "Status:":
                # Status dropdown
                status_var = tk.StringVar(value=bill_data[4])
                status_dropdown = ttk.Combobox(form_frame, textvariable=status_var, width=32)
                status_dropdown['values'] = ["Pending", "Paid", "Partial", "Cancelled"]
                status_dropdown.grid(row=i, column=1, pady=10, padx=10)
                entries.append(status_var)
            else:
                entry = tk.Entry(form_frame, font=("Arial", 11), width=35)
                entry.grid(row=i, column=1, pady=10, padx=10)
                entries.append(entry)
        
        # Set current values for other fields
        entries[1].insert(0, str(bill_data[1]))  # appointment_id
        entries[2].insert(0, f"{bill_data[3]:.2f}")  # total_amount
        entries[4].insert(0, bill_data[5] if bill_data[5] else "")  # payment_date
        entries[5].insert(0, bill_data[6] if bill_data[6] else "")  # items

        # Action buttons
        button_frame = tk.Frame(dialog, bg="white")
        button_frame.pack(pady=25)

        def save_edited_billing():
            """Save edited billing data to database."""
            # Get values from form fields
            pet_str = entries[0].get()
            app_id = entries[1].get()
            total_amount = entries[2].get()
            status = entries[3].get()
            payment_date = entries[4].get()
            items = entries[5].get()

            # Extract pet ID from dropdown string
            if not pet_str:
                messagebox.showerror("Error", "Please select a pet (customer)!")
                return
            
            try:
                # Extract pet ID from the string: "ID: X - PetName (Owner: OwnerName)"
                pet_id = int(pet_str.split("ID: ")[1].split(" - ")[0])
            except (IndexError, ValueError):
                messagebox.showerror("Error", "Invalid pet selection!")
                return

            # Validate required fields
            if app_id and total_amount:
                try:
                    total_amount_float = float(total_amount)
                    
                    # Validate payment date format if provided
                    if payment_date:
                        try:
                            datetime.strptime(payment_date, "%Y-%m-%d")
                        except ValueError:
                            messagebox.showerror("Error", "Invalid date format! Use YYYY-MM-DD.")
                            return
                    
                    # Update billing in database
                    self.db.update_billing(bill_id, app_id, pet_id, total_amount_float, status, items, 
                                          payment_date if payment_date else None)
                    
                    # Add activity record
                    self.db.add_record(f"Billing updated: ID {bill_id}")
                    
                    # Refresh billing display
                    self.load_billing_data()
                    
                    dialog.destroy()
                    messagebox.showinfo("Success", "Billing entry updated successfully!")
                except ValueError as e:
                    messagebox.showerror("Error", f"Invalid input: {str(e)}")
            else:
                messagebox.showerror("Error", "Appointment ID and Total Amount are required!")

        # Save button
        save_btn = tk.Button(button_frame, text="💾 Save Changes", 
                            font=("Arial", 12, "bold"),
                            bg=self.colors['success'], fg="white",
                            padx=30, pady=12,
                            command=save_edited_billing)
        save_btn.pack(side="left", padx=15)

        # Cancel button
        cancel_btn = tk.Button(button_frame, text="❌ Cancel", 
                              font=("Arial", 12, "bold"),
                              bg=self.colors['accent'], fg="white",
                              padx=30, pady=12,
                              command=dialog.destroy)
        cancel_btn.pack(side="left", padx=15)
    
    def delete_billing(self):
        """Delete a billing entry with confirmation."""
        if not hasattr(self, 'billing_tree'):
            messagebox.showerror("Error", "Billing table not loaded!")
            return
        
        # Get selected item from treeview
        selection = self.billing_tree.selection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a billing entry first!")
            return
        
        # Get billing ID from selected item
        item = self.billing_tree.item(selection[0])
        values = item['values']
        bill_id = int(values[0])
        
        # Confirm deletion
        if messagebox.askyesno("Confirm Deletion", 
                              f"Are you sure you want to delete billing entry #{bill_id}?"):
            # Delete from database
            self.db.delete_billing(bill_id)
            
            # Add activity record
            self.db.add_record(f"Billing deleted: ID {bill_id}")
            
            # Remove from treeview
            self.billing_tree.delete(selection[0])
            
            messagebox.showinfo("Success", "Billing entry deleted successfully!")
    
    def exit_system(self):
        """Exit the application with confirmation."""
        if messagebox.askyesno("Confirm Exit", "Are you sure you want to exit the system?"):
            self.db.add_record("System: Application closed")
            self.root.destroy()


# ============= APPLICATION ENTRY POINT =============
if __name__ == "__main__":
    # Create main window
    root = tk.Tk()
    
    # Create and run application
    app = VetClinicGUI(root)
    
    # Start the main event loop
    root.mainloop()
