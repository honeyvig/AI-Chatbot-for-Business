# AI-Chatbot-for-Business
build ai chat bot to add to businesses for appointment setting cancellation ect..
-------
To build an AI-powered chatbot for appointment scheduling, cancellation, and other business interactions, we'll focus on a few key functionalities:

    Natural Language Understanding (NLU): Understand user requests, such as appointment setting and cancellations.
    Dialog Management: Manage the flow of conversation, keeping track of the user's intent and context.
    Backend Integration: For setting up and canceling appointments (This would involve integrating with a database or an external calendar API like Google Calendar).
    Frontend Interface: A simple chat interface, which could be integrated into websites or apps.

In this example, we will use Python, with libraries like Rasa, Flask, and spacy for NLP processing, to build the chatbot. Rasa is an open-source platform for building conversational AI and is well-suited for developing appointment scheduling bots.
Step 1: Install Dependencies

First, you'll need to install some required packages.

pip install rasa flask spacy
python -m spacy download en_core_web_sm

Step 2: Rasa Setup for NLU and Dialog Management

Rasa will handle the natural language understanding (NLU) and dialog management, which are essential for understanding appointment-related intents.

    Initialize Rasa project:

rasa init --no-prompt

    Create intents and entities for the bot:

In the data/nlu.yml file, define the intents and entities that the bot will handle. For example, we'll add intents for setting and canceling appointments.

version: "2.0"
nlu:
  - intent: set_appointment
    examples: |
      - I want to schedule an appointment.
      - Can I book an appointment for tomorrow?
      - Schedule an appointment with the doctor.
      - Set up an appointment for next week.
  
  - intent: cancel_appointment
    examples: |
      - I want to cancel my appointment.
      - Can you cancel my appointment for tomorrow?
      - Cancel my meeting with the doctor.
      - I need to cancel my appointment.
  
  - intent: greet
    examples: |
      - Hi
      - Hello
      - Good morning
      - Hey
  
  - intent: goodbye
    examples: |
      - Goodbye
      - See you later
      - Bye

    Define Actions: To handle booking or canceling appointments, we’ll need a custom action. You can define custom actions in the actions.py file.

In actions.py, you'll define the logic for setting up and canceling appointments.

from rasa_sdk import Action
from rasa_sdk.executor import CollectingDispatcher
from rasa_sdk.events import SlotSet
import datetime

class ActionSetAppointment(Action):
    def name(self):
        return "action_set_appointment"

    def run(self, dispatcher, tracker, domain):
        # Extract date information
        appointment_date = tracker.get_slot('appointment_date')
        dispatcher.utter_message(text=f"Your appointment has been set for {appointment_date}.")
        return []

class ActionCancelAppointment(Action):
    def name(self):
        return "action_cancel_appointment"

    def run(self, dispatcher, tracker, domain):
        # Extract appointment date or other identifiers
        appointment_date = tracker.get_slot('appointment_date')
        dispatcher.utter_message(text=f"Your appointment for {appointment_date} has been canceled.")
        return []

    Create Stories: In the data/stories.yml file, define how the conversation flows. Rasa uses stories to define dialogue patterns.

version: "2.0"
stories:
  - story: schedule appointment
    steps:
      - intent: greet
      - action: utter_greet
      - intent: set_appointment
      - action: action_set_appointment
  
  - story: cancel appointment
    steps:
      - intent: greet
      - action: utter_greet
      - intent: cancel_appointment
      - action: action_cancel_appointment

    Train the Rasa model:

rasa train

Step 3: Integrate with Flask

Now, let's create a simple API in Flask that can be used to integrate this Rasa-based chatbot with a frontend for appointment setting and cancellations.

In the app.py file:

from flask import Flask, request, jsonify
from rasa.core.agent import Agent
from rasa.core.run import serve_application
import logging

app = Flask(__name__)

# Load the trained Rasa model
agent = Agent.load("models/20211219-131317.tar.gz")  # Replace with the actual model path

@app.route("/chat", methods=["POST"])
def chat():
    user_message = request.json.get("message")
    
    if user_message:
        response = agent.handle_text(user_message)
        return jsonify({"response": response[0]['text']})
    return jsonify({"response": "Sorry, I didn't get that."}), 400

if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    app.run(port=5005)

Step 4: Run the Flask API and Rasa Bot

    Start the Flask API:

python app.py

    Run Rasa Actions Server: Ensure that you start the action server to handle custom actions for appointment scheduling and cancellation.

rasa run actions

    Testing the Chatbot API: To test, you can send a POST request to the Flask API with a JSON payload containing the message.

curl -X POST -H "Content-Type: application/json" -d '{"message": "I want to schedule an appointment for tomorrow"}' http://localhost:5005/chat

The bot should reply with an appropriate response such as "Your appointment has been set for tomorrow."
Step 5: Frontend Integration (Optional)

To integrate this chatbot with a frontend, you could create a simple chat interface in HTML and JavaScript. Here is a minimal example using JavaScript:

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Appointment Chatbot</title>
  <style>
    #chat-box {
      width: 300px;
      height: 400px;
      border: 1px solid #ccc;
      overflow-y: scroll;
    }
    #message-input {
      width: 300px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div id="chat-box"></div>
  <input type="text" id="message-input" placeholder="Type a message..." />
  <button id="send-button">Send</button>

  <script>
    const sendButton = document.getElementById("send-button");
    const messageInput = document.getElementById("message-input");
    const chatBox = document.getElementById("chat-box");

    sendButton.addEventListener("click", async function() {
      const userMessage = messageInput.value;
      if (userMessage.trim()) {
        chatBox.innerHTML += `<div>User: ${userMessage}</div>`;
        
        // Send the message to the Flask API
        const response = await fetch('http://localhost:5005/chat', {
          method: 'POST',
          headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({ message: userMessage })
        });

        const data = await response.json();
        chatBox.innerHTML += `<div>Bot: ${data.response}</div>`;
        messageInput.value = '';  // Clear input
      }
    });
  </script>
</body>
</html>

Final Thoughts:

    The above implementation creates a basic AI-powered chatbot for appointment scheduling and cancellation.
    You can expand it by adding more intents, enhancing NLP models, and connecting it with external systems such as calendars or databases for real-world scheduling functionality.
    This solution can be extended to include more complex functionalities such as reminders, rescheduling, or querying existing appointments.

By integrating the chatbot with a frontend (e.g., a website or mobile app), users can easily interact with the bot for managing appointments.
