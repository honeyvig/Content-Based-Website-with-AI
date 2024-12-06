# Content-Based-Website-with-AI
Content Website (Main Page you first see when you are not signed up, For Businesses, Main App Page, Checkout Page (Customised Stripe Page))
Account Authorisation
Chat Functions And Abilities (Uploads, STT, Web Search)
Upgrade and Payment Processing
Upgraded Account Additional Features


Free Account:
When trying to access premium features an upgrade pop-up comes up.
50 messages per day limit
Only GPT-4o-Mini
No File Uploads
No Speech to Text
Web Results with adaptive UI using only GPT-4o-Mini (Use this project as basis: https://github.com/rashadphz/farfalle?tab=readme-ov-file#-deploy)


Paid Account:
No message limit, but requests limit per account. Only one request at a time. No more than 50 messages every 2 hours.
Access to GPT-4o
Access to o1
File Uploads
Microphone Use
Web Results with adaptive UI GPT-4o and Advanced Search (100 Searches Per day Limit (Could Change based on cost)


Sign Up process (Specific Sign Up Process  that connects with the user): It asks for your name, that is set for your profile.
It asks what do you do in your free time, like what are the things you love.
It asks what do you do for work
It tells you how it can help with your work so you can enjoy more of the things you love.
This is part of the sign up process
Then shows a quick tutorial


Note about Web Results: The app determines on its own if it should search the web or not based on a user chat/query


All Designs are ready and will be provided in .PSD form

The project is a chatgpt app targeted towards the Greek Market, but you do not need to speak Greek since everything like texts and prompts will be provided.
==============
The project you've described involves building a content-based website with various features, including a chat application, account management, payment processing, and AI capabilities like GPT-4 for different user tiers (Free and Paid accounts). Here's a breakdown of how to implement this system using Python for the backend (with frameworks like Flask or Django) and integrating with necessary APIs like Stripe for payment processing.
Key Features and Steps

    Main Website (Non-Signed Up Landing Page)
    Account Authorisation (Sign-up/Login)
    Main App Page (Chat Interface, Web Search, etc.)
    Checkout Page (Stripe Integration for Paid Accounts)
    Free Account Limitations
    Paid Account Features and Limitations
    Speech to Text (STT) and File Uploads (for Paid Accounts)
    Upgrade Process (Pop-up for Upgrading to Premium)
    Web Search (Adaptive UI)

Steps to Implement
1. Set Up Backend and Frontend

We’ll use Flask for the backend, where you’ll manage user sessions, account types, and integrate Stripe for payment processing. For the frontend, you can use basic HTML, CSS, and JavaScript, or use a frontend framework like React or Vue.js.
2. Install Necessary Packages

pip install flask stripe openai requests

3. Create the Folder Structure

/app
    /static
    /templates
    app.py
    models.py
    config.py
    stripe_config.py

    /static: For CSS, JavaScript, and media files
    /templates: For HTML templates
    app.py: Main application file where the Flask app will run
    models.py: Database models (for user accounts, payments)
    config.py: App configuration (API keys, database settings)
    stripe_config.py: Stripe-specific configuration

4. Set Up User Authentication (Sign Up/Login)

models.py:

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)
    password = db.Column(db.String(100), nullable=False)
    account_type = db.Column(db.String(50), default='free')
    free_message_limit = db.Column(db.Integer, default=50)
    paid_message_limit = db.Column(db.Integer, default=100)

app.py (User Registration/Authentication)

from flask import Flask, render_template, request, redirect, url_for, flash, session
from models import db, User
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your_secret_key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
db.init_app(app)

@app.route('/signup', methods=['GET', 'POST'])
def signup():
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        password = generate_password_hash(request.form['password'])
        
        new_user = User(name=name, email=email, password=password)
        db.session.add(new_user)
        db.session.commit()
        
        flash('Sign up successful!', 'success')
        return redirect(url_for('login'))
    
    return render_template('signup.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        email = request.form['email']
        password = request.form['password']
        
        user = User.query.filter_by(email=email).first()
        if user and check_password_hash(user.password, password):
            session['user_id'] = user.id
            return redirect(url_for('main_app'))
        flash('Invalid login credentials.', 'danger')
    
    return render_template('login.html')

@app.route('/main_app')
def main_app():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    
    user = User.query.get(session['user_id'])
    return render_template('main_app.html', user=user)

signup.html:

<form method="POST">
  <input type="text" name="name" placeholder="Your Name" required>
  <input type="email" name="email" placeholder="Your Email" required>
  <input type="password" name="password" placeholder="Your Password" required>
  <button type="submit">Sign Up</button>
</form>

5. Chat Functionality (GPT-4 Integration)

app.py (GPT-4 Integration):

import openai

openai.api_key = 'your_openai_api_key'

def get_chat_response(user_input, model='gpt-4o-mini'):
    response = openai.Completion.create(
        model=model,
        prompt=user_input,
        max_tokens=150
    )
    return response.choices[0].text.strip()

main_app.html (Simple Chat Interface):

<form id="chat-form">
  <input type="text" id="user-input" placeholder="Type your question..." required>
  <button type="submit">Send</button>
</form>

<div id="chat-output"></div>

<script>
  document.getElementById('chat-form').addEventListener('submit', function(e) {
    e.preventDefault();
    let userInput = document.getElementById('user-input').value;

    fetch('/get_response', {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify({user_input: userInput}),
    })
    .then(response => response.json())
    .then(data => {
      document.getElementById('chat-output').innerHTML += `<p>User: ${userInput}</p><p>AI: ${data.response}</p>`;
    });
  });
</script>

app.py (Handling Chatbot Response):

from flask import jsonify

@app.route('/get_response', methods=['POST'])
def get_response():
    data = request.get_json()
    user_input = data.get('user_input')
    user = User.query.get(session['user_id'])

    # Decide on model based on account type
    if user.account_type == 'paid':
        model = 'gpt-4o'
    else:
        model = 'gpt-4o-mini'

    response = get_chat_response(user_input, model)
    return jsonify({'response': response})

6. Upgrade Pop-up for Free Account

You can add a JavaScript pop-up that triggers when a free user tries to access premium features.

<script>
  function showUpgradePopup() {
    alert("Upgrade to a paid plan to access this feature!");
  }
</script>

7. Stripe Payment Integration

stripe_config.py:

import stripe

stripe.api_key = "your_stripe_secret_key"

def create_checkout_session(amount):
    session = stripe.checkout.Session.create(
        payment_method_types=['card'],
        line_items=[{
            'price_data': {
                'currency': 'usd',
                'product_data': {
                    'name': 'Premium Account',
                },
                'unit_amount': amount,
            },
            'quantity': 1,
        }],
        mode='payment',
        success_url='http://localhost:5000/success',
        cancel_url='http://localhost:5000/cancel',
    )
    return session.url

Checkout Page (Payment Button):

<a href="{{ url_for('checkout') }}">Upgrade Now</a>

<script>
  function redirectToCheckout() {
    fetch('/create_checkout_session')
      .then(response => response.json())
      .then(data => window.location.href = data.session_url);
  }
</script>

app.py (Stripe Checkout):

from stripe_config import create_checkout_session

@app.route('/create_checkout_session')
def create_checkout_session_route():
    session_url = create_checkout_session(1000)  # 1000 cents = $10
    return jsonify({'session_url': session_url})

8. File Upload (Paid Feature)

app.py (Handle File Uploads):

from flask import request

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'user_id' not in session:
        return redirect(url_for('login'))

    user = User.query.get(session['user_id'])
    if user.account_type == 'free':
        return 'Upgrade to upload files.'

    if 'file' not in request.files:
        return 'No file part'

    file = request.files['file']
    file.save(f"./uploads/{file.filename}")
    return 'File uploaded successfully!'

9. Web Search Integration (Adaptive UI)

You can use the requests library to fetch web results if the system determines that the query requires a web search.

import requests

def web_search(query):
    response = requests.get(f"https://api.duckduckgo.com/?q={query}&format=json")
    results = response.json()
    return results['RelatedTopics']

Conclusion

The above code offers a high-level framework for building an AI-powered chat application with different tiers of account access (Free and Paid), Stripe payment integration, file uploads, and web search functionality. The features are expandable and customizable based on further user requirements or adjustments.
