import mysql.connector as sql
import random
import time
import sys


mycon = sql.connect(
        host = '127.0.0.1',
        user = "root",
        password = "abimbola",
        database = 'mybankproject' 
)

# mycursor = mycon.cursor() 
mycon.autocommit = True

# mycursor.execute('CREATE DATABASE mybankproject')
# mycursor.execute("CREATE TABLE customer_user (customer_ID INT(4) PRIMARY KEY AUTO_INCREMENT, full_name VARCHAR(40), phone_no VARCHAR(12), username VARCHAR(10) UNIQUE, pin INT(4), account_no VARCHAR(12) UNIQUE, account_balance FLOAT(20), date_created DATETIME DEFAULT CURRENT_TIMESTAMP)")
# print('Table created')
# mycursor.execute("CREATE TABLE Transaction_table (transaction_id INT(4) PRIMARY KEY AUTO_INCREMENT, date_time DATETIME(6), amount FLOAT(20), transaction_type VARCHAR(20), beneficiary_acc VARCHAR(12), sender_acc VARCHAR(12), reciever_account VARCHAR(12), account_no VARCHAR(12))")
# print('Table created')

# mycursor.execute('SHOW TABLES')
# for table in mycursor:
#     print(table) 


class Bank:
    def __init__(self):
        self.name = "Lolla Bank "
        self.account_No = str(random.randint(1000000000,1099999999))
        self.account_balance = float(0)
        self.mycursor = mycon.cursor()
        self.dashboard()
    
        
        

    def dashboard(self):
        print(f''' 
            Welcome to {self.name} Bank
            1. Register
            2. Sign in
            3. Exit
        ''')
        user = input('option: ')
        if user == '1':
            self.register()
        elif user == '2':
            self.signin()
        elif user == '3':
            print('Thank you for banking with us . Goodbye👍')
        else:
            print('Invalid input')
            pass
            self.dashboard()

    def register(self):
        print('''
        Signup\n
        ''')

        full_name = input("Full Name: ")
        phone_no = input("Phone Number: ")
        username =input("Enter Prefered Username: ")
        pin =input("Enter your four(4)digit pin: ")

        while len(pin)!= 4 or not pin.isdigit():
            print("Invalid pin. please enter a 4-digit number.")
            pin = input("Enter your four(4) digit pin: ")

        try:
            query = "INSERT INTO customer_user(full_name, phone_no, username, pin, account_no, account_balance) VALUES (%s, %s, %s, %s, %s, %s)"
            values = (full_name, phone_no, username, pin, self.account_No, self.account_balance )
            self.mycursor.execute(query, values)
            mycon.commit()
        except :
            print("Username Taken, enter another")
        else:
            print("Please wait...")
            time.sleep(2)
            print(f'''
            Congratulations your account as been created successfully.
            Your account number is {self.account_No} 
            
            You can proceed...
             ''')
    def signin(self):
        
        print('''
         Sign In\n
         ''')

        username = input("Enter your username: ")
        pin = input("Enter your 4-digit PIN: ")

        while len(pin) != 4 or not pin.isdigit():
            print("Invalid PIN. Please enter a 4-digit number.")
            pin = input("Enter your 4-digit PIN: ")

        try:
            query = "SELECT full_name, account_no, account_balance FROM customer_user WHERE username = %s AND pin = %s"
            values = (username, pin)
            self.mycursor.execute(query, values)
            user_data = self.mycursor.fetchone()
            # print(user_data)

            if user_data:
                full_name, account_no, account_balance = user_data
                time.sleep(2)
                print('Loading......')
                print("Sign in successfully!")
                print(f'Welcome {full_name} with account number {account_no}, Your account balance is ${account_balance}')
                self.dashboard2()
            else:
             print("Invalid username or PIN. Please try again.")
        except:
            print("An error occurred")
            self.dashboard()


    def dashboard2(self):
        print(f'''
            Hello user,{self.name} is here to help you!
            1. deposit
            2. withdraw
            3. transfer
            4. check balance
            5. airtime
            6. logout
            7. check history
            8. Exit
          ''')
        try:
            choice = int(input("Enter your choice: "))
            if choice == 1:
                self.deposit()
            elif choice == 2:
             self.withdraw()
            elif choice == 3:
             self.transfer()
            elif choice == 4:
             self.account_bal()
            elif choice == 5:
             self.airtime()
            elif choice == 6:
                self.dashboard()
            elif choice == 7:
             self.history()
            elif choice == 8:
              self.exit()
            else:
             print('Invalid input')
            self.dashboard2()
        except ValueError:
           print("Invalid input. please enter a valid number.")
        self.dashboard2()

    def deposit(self):
        try:
            amount = float(input("Enter the amount to deposit: "))
            username = input("Enter your username: ")
        
            if amount <= 0:
                print("Invalid amount. Please enter a positive number.")
                self.deposit()
            else:
                query = "UPDATE customer_user SET account_balance = account_balance + %s WHERE username = %s"
                values = (amount, username )
                self.mycursor.execute(query, values)
                mycon.commit()  
                self.account_balance += amount
            
                time.sleep(2)
                print(f"Deposit successful. Your new balance is: {self.account_balance}")
        except ValueError:
            print("Invalid input. Please enter a valid number.")
            self.dashboard2()


    def withdraw(self):
        try:
            amount = float(input("Enter the amount to withdraw: "))
            username = input("Enter your username: ")
        
            if amount <= 0:
                print("Invalid amount. Please enter a positive number.")
                self.withdraw()
            elif amount > self.account_balance:
                print(f"Insufficient funds. Your current balance is: {self.account_balance}.")
                self.withdraw()
            else:
           
                query = "UPDATE customer_user SET account_balance = account_balance - %s WHERE username = %s"
                values = (amount, username)
                self.mycursor.execute(query, values)
                mycon.commit() 
                self.account_balance -= amount
            
                time.sleep(2)
                print(f"Withdrawal successful. Your new balance is: {self.account_balance}")
        except ValueError:
            print("Invalid input. Please enter a valid number.")
            self.dashboard2()

    def transfer(self):
       
        try:
            amount = float(input("Enter the amount to transfer: "))
            recipient_username = input("Enter the recipient's username: ")
            username = input("Enter your username: ")
        

            if amount <= 0:
                print("Invalid amount. Please enter a positive number.")
                self.transfer()
            elif amount > self.account_balance:
                print("Insufficient funds. Please try again.")
                self.transfer()
            else:
                query = "SELECT * FROM customer_user WHERE username = %s"
                values = (recipient_username,)
                self.mycursor.execute(query, values)
                recipient_data = self.mycursor.fetchone()

            if recipient_data:
                query = "UPDATE customer_user SET account_balance = account_balance - %s WHERE username = %s"
                values = (amount, username)
                self.mycursor.execute(query, values)

                query = "UPDATE customer_user SET account_balance = account_balance + %s WHERE username = %s"
                values = (amount, recipient_username)
                self.mycursor.execute(query, values)
                mycon.commit()

                self.account_balance -= amount

                time.sleep(2)
                print(f"Transfer successful. Your new balance is: {self.account_balance}")
            else:
                print("Recipient not found. Please try again.")
                self.transfer()
        except ValueError:

            print("Invalid input. Please enter a valid number.")
            self.dashboard2()

    def account_bal(self):
        username = input("Enter your username: ")
        pin = input("Enter your 4-digit PIN: ")

        while len(pin) != 4 or not pin.isdigit():
            print("Invalid PIN. Please enter a 4-digit number.")
            pin = input("Enter your 4-digit PIN: ")

        try:
            query = "SELECT account_balance FROM customer_user WHERE username = %s AND pin = %s"
            values = (username, pin)
            self.mycursor.execute(query, values)
            result = self.mycursor.fetchone()
            if result:
                balance_amount = result[0]
                print(f"Your account balance is {balance_amount}")
            else:
             print("Invalid username or PIN. Please try again.")
        except ValueError:
         print("Invalid input. Please enter a valid number.")
        self.dashboard2()


    def airtime(self):
        try:
            username = input("Enter your username: ")
            pin = input("Enter your 4-digit PIN: ")

        
            while len(pin) != 4 or not pin.isdigit():
                print("Invalid PIN. Please enter a 4-digit number.")
                pin = input("Enter your 4-digit PIN: ")

        
            query = "SELECT account_balance FROM customer_user WHERE username = %s AND pin = %s"
            values = (username, pin)
            self.mycursor.execute(query, values)
            result = self.mycursor.fetchone()

            if not result:
                print("Invalid username or PIN. Please try again.")
                return self.dashboard2()

            account_balance = result[0]

        
            try:
                amount = float(input("Enter amount for airtime: "))
                if amount <= 0:
                    print("Invalid amount. Please enter a positive number.")
                    return self.airtime()
            except ValueError:
                print("Invalid input. Please enter a numeric value.")
                return self.airtime()

            phone_number = input("Enter recipient's phone number: ").strip()

        
            if len(phone_number) != 11 or not phone_number.isdigit():
                ("Invalid phone number. Please enter a valid 11-digit phone number.")
                return self.airtime()

        
            if amount > account_balance:
                print(f"Insufficient funds. Your current balance is: ${account_balance}")
                return self.dashboard2()

    
            query = "UPDATE customer_user SET account_balance = account_balance - %s WHERE username = %s"
            values = (amount, username)
            self.mycursor.execute(query, values)
            mycon.commit()

    
            account_balance -= amount

            print(f"Airtime purchase successful! {phone_number} has been recharged with ${amount}.")
            print(f"Your new balance is: ${account_balance}")
            self.dashboard2()

        except:
            print("An error occurred ")
            self.dashboard2()

    def history(self):
        print("Transaction History\n")
        username = input("Enter your username: ")
        pin = input("Enter your 4-digit PIN: ")

    
        while len(pin) != 4 or not pin.isdigit():
            print("Invalid PIN. Please enter a 4-digit number.")
            pin = input("Enter your 4-digit PIN: ")

        try:
        
            query = "SELECT * FROM customer_user WHERE username = %s AND pin = %s"
            values = (username, pin)
            self.mycursor.execute(query, values)
            user = self.mycursor.fetchone()

            if not user:
                print("Invalid username or PIN. Please try again.")
                return self.dashboard2()

            query = "SELECT date_time, amount, transaction_type, reciever_account FROM Transaction_table WHERE account_no = %s "
            self.mycursor.execute(query, (username,))
            history_records = self.mycursor.fetchall()

            if not history_records:
              print("No transaction history found.")
            else:
                print("Transaction History:")
                print("--------------------")
                for record in history_records:
                    print(f"date_time: {record[0]}")
                    print(f"amount: {record[1]}")
                    print(f"transaction_type: {record[2]}")
                    print(f"reciever_account: {record[3]}")
                    print("--------------------")

            print("\nEnd of transaction history.")
        except:
            print("An error occurred")

            self.dashboard2()

    def exit(self):
        print(f"Thank you for banking with {self.name}👍")
        sys.exit()




myapp = Bank()
myapp.dashboard()
