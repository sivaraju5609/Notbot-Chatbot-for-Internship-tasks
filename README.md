# Notbot-Chatbot-for-Internship-tasks

this project belongs to chatbot in python code


from flask import Flask, request
from twilioiml.messaging_response import MessagingResponse
import sqlite3

app = Flask(__name__)
db = sqlite3.connect('chatbot')
cursor = db.cursor()

# Create a table to store chat session data
cursor.execute('''CREATE TABLE IF NOT EXISTS sessions
                  (session_id INTEGER PRIMARY KEY AUTOINCREMENT,
                   whatsapp_number TEXT,
                   name TEXT,
                   email TEXT,
                   experience TEXT)''')
db.commit()

@app.route("/webhook", methods=['POST'])
def webhook():
    incoming_message = request.values.get('Body', '').lower()
    phone_number = request.values.get('From', '')

    # Check if the user is new or existing based on their phone number
    cursor.execute("SELECT * FROM sessions WHERE whatsapp_number=?", (phone_number,))
    session_data = cursor.fetchone()

    if not session_data:
        # New user, create a new session
        cursor.execute("INSERT INTO sessions (whatsapp_number) VALUES (?)", (phone_number,))
        db.commit()
        session_id = cursor.lastrowid
        response = "Hi! Are you here to apply for the Internship?\n\nReply with 'Yes' or 'No'."
    else:
        # Existing user, continue the conversation
        session, _, _, _, _ = session_data
        if incoming_message == 'yes':
            response = "Please enter your Name."
        elif incoming_message == 'no':
            response = "Thank you for your time. Goodbye!"
            # Clear the session data for the current user
            cursor.execute("DELETE FROM sessions WHERE session_id=?", (session,))
            db.commit()
        elif not session_data[2]:
            # User has not provided their name yet
            if incoming_message.isdigit():
                response = "Invalid input. Please enter your Name."
            else:
                cursor.execute("UPDATE sessions SET name=? WHERE session_id=?", (incoming_message, session_id))
                db.commit()
                response =Please enter your email ID."
        elif not session_data[3]:
            # User has not provided their email yet
            if incoming_message.isdigit() incoming_message.replace(".", "").isalpha():
                response = "Invalid input. Please enter your email ID."
            else                cursor.execute("UPDATE sessions SET email=? WHERE session_id=?", (incoming_message, session_id))
                db.commit()
                response = "Please select how many years of experience you have with Python/JS/Automation Development:\n\n1 year\n2 years\n3 years\n4 years\n5 years"
        elif not session_data[4]:
            # User has not provided their experience yet
            if incoming_message not in ['1', '2', '3', '4', '5']:
                response = "Invalid input. Please select a valid option."
            else:
                cursor.execute("UPDATE sessions SET experience=? WHERE session_id=?", (incoming_message, session_id))
                db.commit()
                response =Thanks for connecting. We will get back to you shortly."

 # Save the incoming message and response to the database
 cursor.execute("INSERT INTO messages (session_id, message, direction) VALUES (?, ?, ?)",
                   (session_id, incoming_message, "incoming"))
 cursor.execute("INSERT INTO messages (session_id, message, direction) VALUES (?, ?, ?)",
                   (session_id, response,outgoing"))
    db.commit()

 # Prepare the TwiML response
    twiml = MessagingResponse()
    twiml.message(response)

    return str(twiml)


if __name__ == "__main__":
    app.run(debug=True)

