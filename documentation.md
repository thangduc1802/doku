"""
Book Recommendation App

- This Flask application serves as a front-end for a book recommendation system. 
- Users can register, log in, search for books using the Google Books API, and save their favorites. 
- Favorites are stored in JSON format, and users can manage them through the app.

Author: Paul, Tim, Thang
Date: 06.10.2024
Version: 0.1.0 (major.minor.bugfix)
License: Free
"""
from flask import Flask, render_template, request, redirect, session, url_for
from flask.cli import load_dotenv
from backend import database, google_books_api, json_storage
import logging

app = Flask(__name__)
"""We used one static hardcoded key, for developing reasons"""
app.secret_key = ('VJ8_zl5jBLD2Rh0KM9M8xv1S7Jh7-UzXGvHb0ljhA1x7x3u4a6mA')

# Set up logging configuration
logging.basicConfig(filename='application.log', 
                    level=logging.INFO, 
                    format='%(asctime)s - %(levelname)s - %(message)s')

@app.route('/')
def index():
    """
    Displays the homepage with streak information if the user is logged in.
    
    Returns:
        Rendered homepage (index.html).

    Tests:
        1. **Logged-in User**:
            - Input: A user is logged in (user_id exists in the session).
            - Expected Outcome: The homepage is displayed, and streak data is correctly fetched and shown.

        2. **Anonymous User**:
            - Input: No user is logged in (session does not contain a user_id).
            - Expected Outcome: The homepage is displayed without streak data.
    """
    user_id = session.get('user_id')

    # Initialize streak data as empty
    streak_data = None

    # If the user is logged in, get the streak data
    if user_id:
        streak_data = database.get_user_streak_data(user_id)
        logging.info(f"Displayed streak data for user {user_id}.")
    else:
        logging.info("Homepage accessed without login.")

    return render_template('index.html', streak_data=streak_data)




@app.route('/register', methods=['GET', 'POST'])
def register():
    """
    Tests:
        1. **Successful Registration**:
            - Input: A valid username and password.
            - Expected Outcome: The user is registered, and the page redirects to the login page.

        2. **Registration Form Rendering**:
            - Input: A GET request to the registration page.
            - Expected Outcome: The registration form is displayed.

    """
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        
        # Benutzer registrieren
        database.register_user(username, password)
        return redirect('/login')
    
    return render_template('register.html')




@app.route('/login', methods=['GET', 'POST'])
def login():
    """
    Tests:
        1. **Successful Login**:
            - Input: Valid username and password.
            - Expected Outcome: The user is authenticated and redirected to the homepage.

        2. **Failed Login**:
            - Input: Invalid username or password.
            - Expected Outcome: The login form is re-displayed with an error message.
    """
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        
        user = database.authenticate_user(username, password)
        if user:
            session['user_id'] = user[0]  # Store user ID in session
            return redirect('/')
        else:
            return 'Login failed. Please check your username and password.'
    
    return render_template('login.html')





@app.route('/streaks')
def streaks():
    """
    Tests:
        1. **Logged-in User Access**:
            - Input: A logged-in user attempts to access the streaks page.
            - Expected Outcome: The streaks page is displayed.

        2. **Unauthorized Access**:
            - Input: A user who is not logged in tries to access the streaks page.
            - Expected Outcome: The user is redirected to the login page with a warning logged.
    """
    if 'user_id' not in session:
        logging.warning("User attempted to access streaks page without logging in.")
        return "Log in to track your reading streaks."
    
    # Rest of your code to display streaks
    return render_template('streaks.html', user_id=session['user_id'])



@app.route('/logout')
def logout():
    """
    Tests:
        1. **Successful Logout**:
            - Input: A logged-in user clicks the logout button.
            - Expected Outcome: The user is logged out, session is cleared, and they are redirected to the homepage.

        2. **Logging**:
            - Input: A user logs out.
            - Expected Outcome: An informational log entry is created, noting the user has logged out.
    """
    session.pop('user_id', None)
    logging.info("User logged out.")
    return redirect('/')


@app.route('/search', methods=['GET', 'POST'])
def search():
    """
    Displays the search form and processes the user's search query.
    
    POST:
        Retrieves books from the Google Books API based on user input (field and topic).

    Returns:
        Renders the search results page if successful or the search form for a GET request.

    """
    Tests:
        1. **Search Query**:
            - Input: A logged-in user submits a search form with a field of interest and topic.
            - Expected Outcome: Books matching the search criteria are fetched from Google Books API and displayed.

        2. **Unauthorized Access**:
            - Input: A user tries to access the search page without logging in.
            - Expected Outcome: The user is redirected to the login page with a warning logged.
    """
    """
    if 'user_id' not in session:
        logging.warning("Unauthorized access to search page.")
        return redirect('/login')  # User must be logged in

    if request.method == 'POST':
        field_of_interest = request.form['field']
        specific_topic = request.form.get('topic', '')

        # Retrieve books from the Google Books API based on search criteria
        books = google_books_api.search_books(field_of_interest, specific_topic)
        logging.info(f"Search query '{field_of_interest}' with topic '{specific_topic}' returned {len(books)} results.")
        return render_template('results.html', books=books, category=field_of_interest)

    return render_template('search.html')


@app.route('/favorites', methods=['GET'])
def favorites():
    """
    Displays the user's favorite books.
    
    Returns:
        Renders the favorites page with the user's favorite books filtered by category if applicable.

    Tests:
        1. **Favorites Display**:
            - Input: A logged-in user has favorite books stored.
            - Expected Outcome: The user's favorites are correctly loaded and displayed.

        2. **Category Filter**:
            - Input: A category filter is applied to the favorite books list.
            - Expected Outcome: Only favorite books from the selected category are shown.
    """
    user_id = session.get('user_id')

    # Load all favorites and filter only the current user's favorites
    all_favorites = json_storage.load_all_favorites()
    favorites_json = all_favorites.get(str(user_id), [])

    # Optional: Filter by category
    category_filter = request.args.get('category', None)
    if category_filter:
        favorites_json = [book for book in favorites_json if book.get('category') == category_filter]

    logging.info(f"Favorites displayed for user {user_id} with category filter '{category_filter}'.")
    return render_template('favorites.html', favorites=favorites_json, category_filter=category_filter)


@app.route('/remove_favorites', methods=['POST'])
def remove_favorites_view():
    """
    Handles removing selected favorite books for a user.
    
    POST:
        Removes the books with the selected ISBNs from the user's favorites list.

    Returns:
        Redirects to the favorites page.

    Tests:
        1. **Removing Favorites**:
            - Input: A logged-in user selects books to remove from their favorites.
            - Expected Outcome: The selected books are removed from the user's favorites, and the page reloads.

        2. **Unauthorized Access**:
            - Input: A non-logged-in user attempts to remove favorite books.
            - Expected Outcome: The user is redirected to the login page with a warning logged.
    """
    if 'user_id' in session:
        user_id = session['user_id']
        selected_isbns = request.form.getlist('selected_books')

        if selected_isbns:
            json_storage.remove_favorites(user_id, selected_isbns)
            logging.info(f"Removed favorites for user {user_id}: {selected_isbns}")

        return redirect(url_for('favorites'))
    logging.warning("Unauthorized attempt to remove favorites.")
    return redirect(url_for('login'))

@app.route('/add_favorite', methods=['POST'])
def add_favorite():
    """
    Adds selected books to the user's favorites.
    
    POST:
        Saves the selected books (with title, author, ISBN, publication year) to the user's favorites list.

    Returns:
        Redirects to the favorites page.

    Tests:
        1. **Successful Addition**:
            - Input: A logged-in user selects books and adds them to their favorites.
            - Expected Outcome: The selected books are added to the user's favorites, and the page redirects to the favorites page.

        2. **Missing Data**:
            - Input: A book with missing title, author, ISBN, or publication year is added.
            - Expected Outcome: An error message is returned with a 400 status code, and the favorite is not added.
    """
    user_id = session.get('user_id')

    selected_books = request.form.getlist('selected_books')
    if not selected_books:
        logging.error("No books selected to add to favorites.")
        return "No books selected.", 400

    for index in selected_books:
        title = request.form.get(f'title_{index}')
        author = request.form.get(f'author_{index}')
        isbn = request.form.get(f'isbn_{index}')
        publication_year = request.form.get(f'publication_year_{index}')
        category = request.form.get(f'category_{index}', 'Uncategorized')

        if not all([title, author, isbn, publication_year]):
            logging.error("Missing data for one of the books.")
            return "Missing data for one of the books.", 400

        # Prepare book details
        book_details = {
            'title': title,
            'author': author,
            'isbn': isbn,
            'publication_year': publication_year,
            'category': category
        }

        json_storage.save_favorite(user_id, book_details)
        logging.info(f"Added favorite book '{title}' for user {user_id}.")

    return redirect('/favorites')


@app.route('/test_json_favorites', methods=['GET'])
def test_json_favorites():
    """
    Test route to display the contents of the favorites JSON file.
    
    Returns:
        Renders a template displaying the contents of the JSON file.

    Tests:
        1. **Display JSON Data**:
            - Input: A logged-in user accesses the test route.
            - Expected Outcome: The contents of the JSON favorites file are correctly displayed.

        2. **Logging**:
            - Input: A user accesses the test route.
            - Expected Outcome: An info log is generated, noting that the JSON favorites data was displayed.
    """
    favorites_data = json_storage.load_all_favorites()
    logging.info("Displayed test JSON favorites data.")
    return render_template('test_json.html', favorites_data=favorites_data)


@app.route('/bookmark', methods=['GET'])
def bookmark():
    """
    Displays the user's bookmarks.
    
    Returns:
        Renders the bookmarks page with the user's bookmarked books.

    Tests:
        1. **Bookmarks Display**:
            - Input: A logged-in user with bookmarked books.
            - Expected Outcome: The user's bookmarks are correctly loaded and displayed.

        2. **Unauthorized Access**:
            - Input: A non-logged-in user attempts to access bookmarks.
            - Expected Outcome: The user is redirected to the login page with a warning logged.
    """
    user_id = session.get('user_id')
    if not user_id:
        logging.warning("Unauthorized access to bookmarks.")
        return redirect('/login')

    # Load all favorites and filter for the current user
    all_favorites = json_storage.load_all_favorites()
    favorites = all_favorites.get(str(user_id), [])

    logging.info(f"Displayed bookmarks for user {user_id}.")
    return render_template('bookmarks.html', favorites=favorites)


@app.route('/update_favorite_page', methods=['POST'])
def update_favorite_page():
    """
    Updates the current page for a favorite book and the user's reading streak.

    Tests:
        1. **Valid Page Update**:
            - Input: A valid page number is submitted for a favorite book.
            - Expected Outcome: The page number is successfully updated in the JSON file.

        2. **Invalid Page Number**:
            - Input: An invalid page number is submitted (non-integer).
            - Expected Outcome: A 400 error is returned, and the page is not updated.
    """
    user_id = session.get('user_id')
    if not user_id:
        logging.warning("Unauthorized attempt to update favorite page.")
        return redirect('/login')  # User must be logged in

    # Get data from the form
    book_isbn = request.form['book_isbn']
    current_page = request.form['current_page']

    try:
        current_page = int(current_page)  # Ensure page number is a valid integer
    except ValueError:
        logging.error(f"Invalid page number '{current_page}' provided.")
        return "Invalid page number", 400

    # Update the current page in the JSON file
    json_storage.update_favorite_page(user_id, book_isbn, current_page)
    logging.info(f"Updated page to {current_page} for book with ISBN {book_isbn} for user {user_id}.")

    # Update the user's reading streak
    database.update_reading_streak(user_id)

    # Reload the page with the updated data
    favorites = json_storage.load_user_favorites(user_id)
    return render_template('bookmarks.html', favorites=favorites)


@app.route('/learnings', methods=['GET', 'POST'])
def learnings():
    """
    Displays and saves learning notes for a user's favorite books.
    
    POST:
        Saves learning notes for a book in the user's favorites.

    Returns:
        Renders the learning notes page.

    Tests:
        1. **Saving Learning Note**:
            - Input: A learning note is submitted for a favorite book.
            - Expected Outcome: The learning note is saved to the JSON file, and the user is redirected to the learnings page.

        2. **Unauthorized Access**:
            - Input: A non-logged-in user attempts to save a learning note.
            - Expected Outcome: The user is redirected to the login page with a warning logged.
    """
    user_id = session.get('user_id')
    if not user_id:
        logging.warning("Unauthorized access to learnings.")
        return redirect('/login')

    if request.method == 'POST':
        book_isbn = request.form['book_isbn']
        learning = request.form['learning']

        json_storage.save_favorite_learning(user_id, book_isbn, learning)
        logging.info(f"Saved learning for book with ISBN {book_isbn} for user {user_id}.")

        return redirect('/learnings')

    favorites = json_storage.load_user_favorites(user_id)
    return render_template('learnings.html', favorites=favorites)

@app.route('/update_reading_progress', methods=['POST'])
def update_reading_progress():
    """
    Updates the user's reading progress and their streak.

    Tests:
        1. **Valid Progress Update**:
            - Input: A valid current page is submitted, and the reading streak is updated.
            - Expected Outcome: The user's reading streak is updated based on the reading progress.

        2. **Unauthorized Access**:
            - Input: A non-logged-in user attempts to update reading progress.
            - Expected Outcome: The user is redirected to the login page.
    """
    user_id = session.get('user_id')
    if user_id:
        current_page = request.form.get('current_page')
        # Update the reading streak based on the progress
        streak_data = database.update_reading_streak(user_id)
        return redirect('/')
    else:
        return redirect('/login')


if __name__ == '__main__':
    app.run(debug=True)



---------------------------------------------------------------------------------------------------------------------------

"""
Database Module

- This module handles the SQLite database connection and operations such as user registration, authentication, and table creation for a book recommendation application.


Author: Paul, Tim, Thang
Date: 06.10.2024
Version: 0.1.0
License: Free
"""

import sqlite3

def connect_db():
    """
    Establishes a connection to the SQLite database.

    Returns:
        Connection object: SQLite connection to 'database.db'.

    Tests:
        1. **Database Connection Success**:
            - Input: Attempt to connect to the database.
            - Expected Outcome: A connection object is returned if the database exists.
        
        2. **Database Connection Error Handling**:
            - Input: (Simulate) Attempt to connect to a non-existent database.
            - Expected Outcome: An exception should be raised indicating that the database cannot be found (this requires a mock or testing environment).
    """
    return sqlite3.connect('database.db')

def create_tables():
    """
    Creates the 'users' and 'reading_progress' tables in the database if they do not exist.
    The 'users' table stores user information, and the 'reading_progress' table stores 
    the user's reading progress with foreign keys linking to the 'users' and 'books' tables.
    
    Returns:
        None

    Tests:
        1. **Table Creation Success**:
            - Input: Call create_tables() when the database is empty.
            - Expected Outcome: Tables 'users' and 'reading_progress' are created successfully without raising any errors.
        
        2. **Table Existence Check**:
            - Input: Call create_tables() twice in a row.
            - Expected Outcome: The second call should not raise any errors, and the tables should remain unchanged.
    """
    conn = connect_db()
    cur = conn.cursor()

    # Create 'users' table if it does not exist
    cur.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT NOT NULL,
            password TEXT NOT NULL
        )
    ''')

    # Create 'reading_progress' table if it does not exist
    cur.execute('''
        CREATE TABLE IF NOT EXISTS reading_progress (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER,
            book_id INTEGER,
            book_title TEXT,
            date DATE DEFAULT (DATE('now')),
            pages_read INTEGER NOT NULL,
            FOREIGN KEY (user_id) REFERENCES users(id),
            FOREIGN KEY (book_id) REFERENCES books(id)
        )
    ''')

    conn.commit()
    conn.close()

def register_user(username, password):
    """
    Registers a new user by inserting their username and password into the 'users' table.

    Args:
        username (str): The username of the new user.
        password (str): The password of the new user.

    Returns:
        None

    Tests:
        1. **User Registration Success**:
            - Input: A valid username and password that do not already exist in the database.
            - Expected Outcome: The user is successfully added to the 'users' table.
        
        2. **Duplicate User Registration Handling**:
            - Input: Attempt to register a username that already exists in the 'users' table.
            - Expected Outcome: An exception or error message is raised, indicating that the username must be unique.
    """
    conn = connect_db()
    cur = conn.cursor()
    
    # Insert user data into the 'users' table
    cur.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))
    
    conn.commit()
    conn.close()

def authenticate_user(username, password):
    """
    Authenticates a user by checking the provided username and password against the 'users' table.

    Args:
        username (str): The username to be checked.
        password (str): The password to be checked.

    Returns:
        tuple: A tuple containing the user data (id, username, password) if authentication is successful.
               None if the credentials are incorrect.

    Tests:
        1. **Successful Authentication**:
            - Input: A valid username and password that exist in the 'users' table.
            - Expected Outcome: A tuple (id, username, password) is returned for the authenticated user.
        
        2. **Unsuccessful Authentication**:
            - Input: A username and password that do not match any record in the 'users' table.
            - Expected Outcome: None is returned, indicating failed authentication.
        3. **Handling of Non-Existent User**:
            - Input: An invalid username or an empty string for the username.
            - Expected Outcome: None is returned, confirming that the user does not exist.
    """
    conn = connect_db()
    cur = conn.cursor()
    
    # Fetch the user data matching the given username and password
    cur.execute("SELECT * FROM users WHERE username = ? AND password = ?", (username, password))
    user = cur.fetchone()  # Returns a tuple: (id, username, password) or None
    
    conn.close()
    
    return user  # Return the user tuple or None if the user is not found

---------------------------------------------------------------------------------------------------------------------------------
    """
Google Books API Module

- This module interacts with the Google Books API to search for books based on user input and retrieve book details using the ISBN.


Author: Paul, Tim, Thang
Date: 06.10.2024
Version: 0.1.0
License: Free
"""

import requests

def search_books(field_of_interest, specific_topic):
    """
    Searches for books based on the field of interest and specific topic using the Google Books API.

    Args:
        field_of_interest (str): The general category or subject area of interest.
        specific_topic (str): An optional specific topic or subcategory within the field of interest.

    Returns:
        list: A list of dictionaries containing details for up to 10 books. 
              Each dictionary contains the following book details:
              - 'title': Title of the book (str).
              - 'author': Author(s) of the book (str).
              - 'isbn': The ISBN of the book (str).
              - 'publication_year': Year of publication (str).
              - 'description': Description of the book (str).
              - 'category': The category or field of interest (str).

    Tests:
        1. **Valid Search Query**:
            - Input: `field_of_interest = "Science"`, `specific_topic = "Physics"`
            - Expected Outcome: The function should return a list of up to 10 dictionaries, each containing book details like 'title', 'author', 'isbn', etc. All fields should be populated with relevant data.

        2. **Empty Specific Topic**:
            - Input: `field_of_interest = "Fiction"`, `specific_topic = ""`
            - Expected Outcome: The function should return a list of up to 10 dictionaries containing fiction books without any filtering by specific topic. The books should still have their details populated correctly.
        
        3. **No Results Found**:
            - Input: `field_of_interest = "NonexistentCategory"`, `specific_topic = "MadeUpTopic"`
            - Expected Outcome: The function should return an empty list `[]`, indicating that no books were found for the given search criteria.
        
        4. **Invalid API Response Handling**:
            - Input: Simulate a scenario where the API returns an unexpected response format.
            - Expected Outcome: The function should handle the error gracefully and not throw an exception. Instead, it should return an empty list or default values.
    """
    query = f'{field_of_interest} {specific_topic}'
    url = f'https://www.googleapis.com/books/v1/volumes?q={query}'
    response = requests.get(url)
    data = response.json()

    books = []
    for item in data.get('items', [])[:10]:  # Limit to 10 books
        book_info = item.get('volumeInfo', {})

        # Add book details
        books.append({
            'title': book_info.get('title', 'N/A'),
            'author': ', '.join(book_info.get('authors', ['N/A'])),
            'isbn': book_info.get('industryIdentifiers', [{'identifier': 'N/A'}])[0]['identifier'],
            'publication_year': book_info.get('publishedDate', 'N/A'),
            'description': book_info.get('description', 'No description available'),
            'category': field_of_interest  # Add the field of interest as the category
        })

    return books


def get_book_by_isbn(isbn):
    """
    Retrieves book details from the Google Books API based on the provided ISBN.

    Args:
        isbn (str): The International Standard Book Number (ISBN) of the book to be retrieved.

    Returns:
        dict or None: A dictionary containing the following book details if found:
            - 'title': Title of the book (str).
            - 'author': Author(s) of the book (str).
            - 'isbn': The ISBN of the book (str).
            - 'publication_year': Year of publication (str).
            - 'description': Description of the book (str).
          Returns `None` if no book is found with the given ISBN.
        
    Tests:
        1. **Valid ISBN**:
            - Input: `isbn = "9780134685991"` (A valid ISBN for a known book)
            - Expected Outcome: The function should return a dictionary containing the book details such as 'title', 'author', 'isbn', etc., with all fields populated with valid information.

        2. **Invalid ISBN**:
            - Input: `isbn = "0000000000000"` (An invalid or nonexistent ISBN)
            - Expected Outcome: The function should return `None`, indicating that no book was found for the given ISBN.
        
        3. **Empty ISBN**:
            - Input: `isbn = ""`
            - Expected Outcome: The function should return `None`, as an empty ISBN is not valid for searching.
        
        4. **Malformed API Response**:
            - Input: Simulate a situation where the API returns an unexpected or malformed response.
            - Expected Outcome: The function should not raise an exception but should return `None`, handling the unexpected response appropriately.
    """
    url = f'https://www.googleapis.com/books/v1/volumes?q=isbn:{isbn}'
    response = requests.get(url)
    data = response.json()

    # Check if 'items' is present in the response JSON
    if 'items' not in data:
        return None  # No results found

    item = data['items'][0]
    book_info = item.get('volumeInfo', {})

    return {
        'title': book_info.get('title', 'N/A'),
        'author': ', '.join(book_info.get('authors', ['N/A'])),
        'isbn': book_info.get('industryIdentifiers', [{'identifier': 'N/A'}])[0]['identifier'],
        'publication_year': book_info.get('publishedDate', 'N/A'),
        'description': book_info.get('description', 'No description available')
    }

---------------------------------------------------------------------------------------------------------------------------------

"""
Favorites Management Module

- This module provides functionality to manage users' favorite books using a JSON file for storage. 
- Users can save their favorite books, add notes about their learning, update the current page they are on, and remove books from their favorites list.

Author: Paul, Tim, Thang
Date: 06.10.2024
Version: 0.1.0
License: Free
"""

import json
import os

# Path to the JSON file for favorites
FAVORITES_JSON_PATH = 'favorites.json'


def load_all_favorites():
    """
    Load all favorites from the JSON file.
    
    Returns:
        dict: A dictionary where keys are user IDs and values are lists of favorite books for each user.
              Returns an empty dictionary if the file doesn't exist or is not readable.
    
    Tests:
        1. **File Exists**:
            - Input: A valid JSON file located at `FAVORITES_JSON_PATH`.
            - Expected Outcome: The function should return a dictionary containing the user's favorite books as loaded from the JSON file.
        
        2. **File Does Not Exist**:
            - Input: A scenario where the JSON file does not exist.
            - Expected Outcome: The function should return an empty dictionary.
        
        3. **Corrupted JSON File**:
            - Input: A corrupted JSON file (e.g., missing brackets).
            - Expected Outcome: The function should catch the JSONDecodeError and return an empty dictionary.
    """
    if not os.path.exists(FAVORITES_JSON_PATH):
        return {}

    try:
        with open(FAVORITES_JSON_PATH, 'r') as file:
            return json.load(file)
    except json.JSONDecodeError:
        return {}


def load_user_favorites(user_id):
    """
    Load favorites for a specific user.

    Args:
        user_id (int or str): The ID of the user whose favorites should be loaded.

    Returns:
        list: A list of favorite books for the given user, or an empty list if the user has no favorites.
    
    Tests:
        1. **User Has Favorites**:
            - Input: A user ID that exists in the favorites JSON file.
            - Expected Outcome: The function should return a list of favorite books for that user.
        
        2. **User Has No Favorites**:
            - Input: A user ID that does not exist in the favorites JSON file.
            - Expected Outcome: The function should return an empty list.
    """
    all_favorites = load_all_favorites()
    return all_favorites.get(str(user_id), [])


def save_favorite(user_id, book_details):
    """
    Save a favorite book for a user in the JSON file.

    Args:
        user_id (int or str): The ID of the user adding a favorite.
        book_details (dict): A dictionary containing the details of the book (e.g., title, author, ISBN).
    
    Returns:
        None
    
    Tests:
        1. **Valid Save**:
            - Input: A user ID that exists and valid book details.
            - Expected Outcome: The book should be added to the user's favorites in the JSON file.
        
        2. **Duplicate Book**:
            - Input: A user ID that exists and book details that are already in the user's favorites.
            - Expected Outcome: The user's favorites should remain unchanged, and the duplicate book should not be added.
    """
    all_favorites = load_all_favorites()

    if str(user_id) not in all_favorites:
        all_favorites[str(user_id)] = []

    # Check if the book already exists
    existing_book = next((book for book in all_favorites[str(user_id)] if book['isbn'] == book_details['isbn']), None)

    if not existing_book:
        all_favorites[str(user_id)].append(book_details)

        # Write the updated favorites back to the JSON file
        with open(FAVORITES_JSON_PATH, 'w') as file:
            json.dump(all_favorites, file, indent=4)


def save_favorite_learning(user_id, book_isbn, learning):
    """
    Save a learning note for a book in the favorites JSON file.

    Args:
        user_id (int or str): The ID of the user.
        book_isbn (str): The ISBN of the book for which the learning note is being saved.
        learning (str): The learning note or takeaway from the book.

    Returns:
        None

    Tests:
        1. **Valid Learning Note Update**:
            - Input: A user ID that exists, a valid ISBN, and a learning note.
            - Expected Outcome: The learning note should be updated in the user's favorite book entry in the JSON file.
        
        2. **Nonexistent Book**:
            - Input: A user ID that exists but a book ISBN that is not in the user's favorites.
            - Expected Outcome: The JSON file should remain unchanged since there’s no matching book.
    """
    all_favorites = load_all_favorites()

    if str(user_id) in all_favorites:
        # Find the book by its ISBN and update its learning note
        for book in all_favorites[str(user_id)]:
            if book['isbn'] == book_isbn:
                book['learning'] = learning
                break

        # Write the updated favorites back to the JSON file
        with open(FAVORITES_JSON_PATH, 'w') as file:
            json.dump(all_favorites, file, indent=4)


def remove_favorites(user_id, selected_isbns):
    """
    Remove selected favorites for a user from the JSON file.

    Args:
        user_id (int or str): The ID of the user.
        selected_isbns (list): A list of ISBNs for the books to be removed from the user's favorites.

    Returns:
        None

    Tests:
        1. **Valid Removal**:
            - Input: A user ID that exists and a list of ISBNs that are present in the user's favorites.
            - Expected Outcome: The specified books should be removed from the user's favorites in the JSON file.
        
        2. **No Matching ISBNs**:
            - Input: A user ID that exists and a list of ISBNs that are not in the user's favorites.
            - Expected Outcome: The user's favorites should remain unchanged, and no books should be removed.
    """
    all_favorites = load_all_favorites()

    if str(user_id) in all_favorites:
        # Filter out the books whose ISBN is in the selected_isbns list
        all_favorites[str(user_id)] = [
            book for book in all_favorites[str(user_id)] if book['isbn'] not in selected_isbns
        ]

        # Write the updated favorites back to the JSON file
        with open(FAVORITES_JSON_PATH, 'w') as file:
            json.dump(all_favorites, file, indent=4)


def update_favorite_page(user_id, book_isbn, current_page):
    """
    Update the current page number of a favorite book for the user.

    Args:
        user_id (int or str): The ID of the user.
        book_isbn (str): The ISBN of the book being updated.
        current_page (int): The new current page number the user is on.

    Returns:
        None

    Tests:
        1. **Valid Page Update**:
            - Input: A user ID that exists, a valid ISBN of a favorite book, and a new current page number.
            - Expected Outcome: The current page number for the specified book should be updated in the user's favorites in the JSON file.
        
        2. **Nonexistent Book Update**:
            - Input: A user ID that exists and an ISBN that does not match any book in the user's favorites.
            - Expected Outcome: The JSON file should remain unchanged since there’s no matching book to update.
    """
    all_favorites = load_all_favorites()

    if str(user_id) in all_favorites:
        # Find the book by its ISBN and update the current page number
        for book in all_favorites[str(user_id)]:
            if book['isbn'] == book_isbn:
                book['current_page'] = current_page
                break

        # Write the updated favorites back to the JSON file
        with open(FAVORITES_JSON_PATH, 'w') as file:
            json.dump(all_favorites, file, indent=4)

------------------------------------------------------------------------------------------------------------------------------------------------------------

"""
Database Initialization Module

- This module is responsible for initializing the database by creating the necessary tables.

Author: Paul, Tim, Thang
Date: 06.10.2024
Version: 0.1.0
License: Free
"""

import database

# This function creates the necessary tables
database.create_tables()
print("Database tables were successfully created")
