<!--
HTML Template for My Book App (For the base.html file, pls delete this small comment in the paratheses, just for clearer specification)
------------------------------
This HTML document serves as the base template for the My Book App web application. It incorporates navigation, content blocks, and footer elements.

Overview:
---------
- Uses the Roboto font from Google Fonts for consistent typography.
- Includes a JavaScript function to toggle the visibility of book descriptions.
- Utilizes Flask's templating engine to allow for dynamic content rendering.

Structure:
----------
1. **Head Section**:
   - Character encoding set to UTF-8 for universal compatibility.
   - Responsive meta tag for mobile device support.
   - Title block that can be overridden in child templates.
   - Link to Google Fonts and external stylesheet for CSS.

2. **Body Section**:
   - **Wrapper**: Main container for the page layout.
   - **Header**: Contains navigation links for different sections of the app.
   - **Content Block**: A placeholder for dynamic content using Flask's block feature.
   - **Footer**: Displays copyright information.

Author: Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Book App{% endblock %}</title>
    
    <!-- Google Fonts - Roboto -->
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
    
    <link rel="stylesheet" href="/static/styles.css">

    <!-- JavaScript for toggling description visibility -->
    <script>
        function toggleDescription(index) {
            var description = document.getElementById('description-' + index);
            if (description.style.display === 'none') {
                description.style.display = 'block';
            } else {
                description.style.display = 'none';
            }
        }
    </script>
</head>
<body>
    <div class="wrapper">
        <header>
            <nav>
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/search">Search Books</a></li>
                    <li><a href="/favorites">Favorites</a></li>
                    <li><a href="/bookmark">Bookmarks</a></li>
                    <li><a href="/learnings">Learnings</a></li> <!-- Adding Learnings page -->
                    {% if session.user_id %}
                        <li><a href="/logout">Logout</a></li>
                    {% else %}
                        <li><a href="/login">Login</a></li>
                        <li><a href="/register">Register</a></li>
                    {% endif %}
                </ul>
            </nav>
        </header>
        
        <div class="content">
            {% block content %}{% endblock %}
        </div>
        
        <footer>
            <p>&copy; 2024 My Book App</p>
        </footer>
    </div>
</body>
</html>

<!--
HTML Template for Bookmarks Page - My Book App
-----------------------------------------------
This HTML document extends the base template for the My Book App web application and is specifically designed to display the user's bookmarks.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Bookmarks page.
- Displays a table listing the user's saved bookmarks, allowing updates on the last read page.

Structure:
----------
1. **Head Section**:
   - Title block set to "Deine Lesezeichen" for the Bookmarks page.

2. **Body Section**:
   - **Container**: Main section to hold all content related to the user's bookmarks.
   - **Bookmarks Table**:
     - If bookmarks exist, displays a table with columns for the book title, author, last page read, and an update feature.
     - Each bookmark has a separate form for updating the current page read, allowing users to easily keep track of their reading progress.
     - If no bookmarks are available, prompts the user to add books to their favorites.

3. **Action Button**: 
   - Provides a button for users to navigate to the book search page if no bookmarks are saved.

Author: Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Deine Lesezeichen{% endblock %}

{% block content %}
<div class="container">
    <h1>Deine Lesezeichen</h1>

    {% if favorites %}
    <table>
        <thead>
            <tr>
                <th>Buchtitel</th>
                <th>Autor</th>
                <th>Letzter Stand</th>
                <th>Seitenzahl Aktualisieren</th>
            </tr>
        </thead>
        <tbody>
            {% for book in favorites %}
            <tr>
                <td>{{ book.title }}</td>
                <td>{{ book.author }}</td>
                <td>{{ book.current_page }}</td>
                <td>
                    <!-- Jedes Buch bekommt sein eigenes Formular -->
                    <form method="POST" action="/update_favorite_page">
                        <input type="number" name="current_page" value="{{ book.current_page }}" min="0" required>
                        <input type="hidden" name="book_isbn" value="{{ book.isbn }}">
                        <button type="submit" class="btn-small">Aktualisieren</button>
                    </form>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
    {% else %}
    <p>Du hast keine gespeicherten Bücher. Füge ein Buch zu deinen Favoriten hinzu, um deine virtuellen Lesezeichen einzutragen.</p>

    <!-- Button für "Bücher suchen" mit blauer Farbe -->
    <a href="/search" class="btn btn-primary" style="background-color: #007bff; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;">Bücher suchen</a>
    {% endif %}
</div>
{% endblock %}

<!--
HTML Template for Favorites Page - My Book App (For the favorites.html)
-----------------------------------------------
This HTML document extends the base template for the My Book App web application and is specifically designed to display the user's favorite books. 

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Favorites page.
- Implements a filter function to categorize favorites by subject.
- Displays a list of the user's favorite books with options to remove them.

Structure:
----------
1. **Head Section**:
   - Title block set to "Deine Favoriten" for the Favorites page.

2. **Body Section**:
   - **Container**: Main section to hold all content related to the user's favorites.
   - **Filter Form**: Allows users to filter their favorites by category with a dropdown menu.
   - **Favorites Display**: 
     - If favorites exist, displays a form to remove selected favorites, listing each favorite book with details such as title, author, ISBN, publication year, and category.
     - If no favorites are available, provides a message prompting the user to search for books.

3. **JavaScript Function**: 
   - Implements a function to toggle the visibility of descriptions, enhancing user interactivity.

Author: Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Deine Favoriten{% endblock %}

{% block content %}
    <div class="container">
        <h1>Deine Favoriten</h1>

        <!-- Filterfunktion nach Kategorie -->
        <form method="GET" action="/favorites" class="filter-form">
            <label for="category">Favoriten nach Kategorie filtern:</label>
            <select name="category" id="category">
                <option value="">Alle Kategorien</option>
                <option value="Biologie" {% if category_filter == 'Biologie' %}selected{% endif %}>Biologie</option>
                <option value="Technologie" {% if category_filter == 'Technologie' %}selected{% endif %}>Technologie</option>
                <option value="Wirtschaft" {% if category_filter == 'Wirtschaft' %}selected{% endif %}>Wirtschaft</option>
                <option value="Mathematik" {% if category_filter == 'Mathematik' %}selected{% endif %}>Mathematik</option>
                <option value="Physik" {% if category_filter == 'Physik' %}selected{% endif %}>Physik</option>
                <option value="Chemie" {% if category_filter == 'Chemie' %}selected{% endif %}>Chemie</option>
                <option value="Medizin" {% if category_filter == 'Medizin' %}selected{% endif %}>Medizin</option>
                <option value="Informatik" {% if category_filter == 'Informatik' %}selected{% endif %}>Informatik</option>
                <option value="Ingenieurwesen" {% if category_filter == 'Ingenieurwesen' %}selected{% endif %}>Ingenieurwesen</option>
                <option value="Umweltwissenschaften" {% if category_filter == 'Umweltwissenschaften' %}selected{% endif %}>Umweltwissenschaften</option>
                <option value="Philosophie" {% if category_filter == 'Philosophie' %}selected{% endif %}>Philosophie</option>
                <option value="Geschichte" {% if category_filter == 'Geschichte' %}selected{% endif %}>Geschichte</option>
                <option value="Kunstwissenschaften" {% if category_filter == 'Kunstwissenschaften' %}selected{% endif %}>Kunstwissenschaften</option>
                <option value="Musikwissenschaft" {% if category_filter == 'Musikwissenschaft' %}selected{% endif %}>Musikwissenschaft</option>
                <option value="Soziologie" {% if category_filter == 'Soziologie' %}selected{% endif %}>Soziologie</option>
                <option value="Pädagogik" {% if category_filter == 'Pädagogik' %}selected{% endif %}>Pädagogik</option>
                <option value="Psychologie" {% if category_filter == 'Psychologie' %}selected{% endif %}>Psychologie</option>
                <option value="Politikwissenschaft" {% if category_filter == 'Politikwissenschaft' %}selected{% endif %}>Politikwissenschaft</option>
                <option value="Rechtswissenschaften" {% if category_filter == 'Rechtswissenschaften' %}selected{% endif %}>Rechtswissenschaften</option>
                <option value="Geografie" {% if category_filter == 'Geografie' %}selected{% endif %}>Geografie</option>
                <option value="Sportwissenschaften" {% if category_filter == 'Sportwissenschaften' %}selected{% endif %}>Sportwissenschaften</option>
                <option value="Astronomie" {% if category_filter == 'Astronomie' %}selected{% endif %}>Astronomie</option>
                <option value="Meteorologie" {% if category_filter == 'Meteorologie' %}selected{% endif %}>Meteorologie</option>
                <option value="Ozeanografie" {% if category_filter == 'Ozeanografie' %}selected{% endif %}>Ozeanografie</option>
            </select>
            <button type="submit" class="btn-small">Filtern</button>
        </form>

        {% if favorites %}
            <!-- Formular zum Entfernen von Favoriten -->
            <form method="POST" action="/remove_favorites">
                <div class="card-container">
                    {% for book in favorites %}
                    <div class="card" id="card-{{ loop.index }}">
                        <label>
                            <input type="checkbox" name="selected_books" value="{{ book.isbn }}" id="checkbox-{{ loop.index }}">
                            <span>Kachel auswählen</span>
                        </label>
                        <h2>{{ book.title }}</h2>
                        <p><strong>Autor:</strong> {{ book.author }}</p>
                        <p><strong>ISBN:</strong> {{ book.isbn }}</p>
                        <p><strong>Erscheinungsjahr:</strong> {{ book.publication_year }}</p>
                        <p><strong>Kategorie:</strong> {{ book.category }}</p>
                    </div>
                    {% endfor %}
                </div>
                <button type="submit" class="btn-large">Ausgewählte Favoriten entfernen</button>
            </form>
        {% else %}
            <!-- Nachricht und "Bücher suchen"-Button, wenn keine Favoriten vorhanden sind -->
            <p>Du hast noch keine Favoriten gespeichert oder keine Favoriten in dieser Kategorie.</p>

            <!-- Button für "Bücher suchen" mit blauer Farbe -->
            <a href="/search" class="btn btn-primary" style="background-color: #007bff; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;">Bücher suchen</a>
        {% endif %}
    </div>

    <script>
        function toggleDescription(index) {
            var description = document.getElementById('description-' + index);
            if (description.style.display === 'none') {
                description.style.display = 'block';
            } else {
                description.style.display = 'none';
            }
        }
    </script>
{% endblock %}

<!--
HTML Template for Home Page - My Book App (For the index.html)
--------------------------------------------
This HTML document extends the base template for the My Book App web application and serves as the main entry point for users.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Home page.
- Welcomes users and presents options for navigating the application.

Structure:
----------
1. **Head Section**:
   - Title block set to "Startseite" for the homepage.

2. **Body Section**:
   - **Container**: Main section to hold the welcome message and navigation options.
   - **Welcome Message**: Displays a greeting and invites users to choose their next action.
   - **Action Buttons**: Two large buttons for navigation:
     - **Search**: Directs users to the book search page.
     - **Favorites**: Directs users to view their saved favorites.

Author: Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Startseite{% endblock %}

{% block content %}
    <div class="container home-container">
        <h1>Willkommen zur Büchersuche</h1>
        <p>Wähle eine der folgenden Optionen:</p>
        <div class="home-buttons">
            <a href="/search" class="btn-large">Suchanfrage stellen</a>
            <a href="/favorites" class="btn-large">Favoriten anzeigen</a>
        </div>
    </div>
{% endblock %}

<!--
HTML Template for Learnings Page - My Book App (For learnings.html)
-----------------------------------------------
This HTML document extends the base template for the My Book App web application and serves as the user interface for managing key learnings from favorite books.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Learnings page.
- Displays users' favorite books alongside an option to note key learnings.

Structure:
----------
1. **Head Section**:
   - Title block set to "Deine Learnings" for the learnings page.

2. **Body Section**:
   - **Container**: Main section to display the user's learnings.
   - **Table**: Lists the user's favorite books with the following columns:
     - **Buchtitel**: Displays the title of the book.
     - **Autor**: Displays the author of the book.
     - **Deine Key Learnings**: Contains a form for users to input and save their key learnings associated with each book.
   - **Empty State Message**: If no favorite books are present, a message prompts the user to add books to their favorites to record learnings.

Author: name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Deine Learnings{% endblock %}

{% block content %}
<div class="container">
    <h1>Deine Learnings</h1>

    {% if favorites %}
    <table>
        <thead>
            <tr>
                <th>Buchtitel</th>
                <th>Autor</th>
                <th>Deine Key Learnings</th>
            </tr>
        </thead>
        <tbody>
            {% for book in favorites %}
            <tr>
                <td>{{ book.title }}</td>
                <td>{{ book.author }}</td>
                <td>
                    <form method="POST" action="/learnings">
                        <input type="hidden" name="book_isbn" value="{{ book.isbn }}">
                        <textarea name="learning" rows="2" cols="40">{{ book.learning }}</textarea>
                        <button type="submit">Speichern</button>
                    </form>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
    {% else %}
    <p>Du hast keine gespeicherten Bücher. Füge Bücher zu deinen Favoriten hinzu, um deine Key Learnings zu notieren und zu speichern.</p>
    {% endif %}
</div>
{% endblock %}

<!--
HTML Template for Login Page - My Book App (For login.html)
--------------------------------------------
This HTML document extends the base template for the My Book App web application and serves as the user interface for user authentication.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Login page.
- Contains a form for users to enter their credentials for authentication.

Structure:
----------
1. **Head Section**:
   - Title block set to "Anmeldung" for the login page.

2. **Body Section**:
   - **Container**: Main section for the login form.
   - **Form**: Collects user input for login credentials:
     - **Benutzername**: Input field for the username (required).
     - **Passwort**: Input field for the password (required).
   - **Submit Button**: A button labeled "Anmelden" to submit the form for authentication.

Author: name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Anmeldung{% endblock %}

{% block content %}
    <div class="container">
        <h1>Anmelden</h1>
        <form method="POST" action="/login">
            <label for="username">Benutzername:</label>
            <input type="text" name="username" required>
            <br><br>
            <label for="password">Passwort:</label>
            <input type="password" name="password" required>
            <br><br>
            <button type="submit" class="btn-large">Anmelden</button>
        </form>
    </div>
{% endblock %}

<!--
HTML Template for Registration Page - My Book App (For register.html)
---------------------------------------------
This HTML document extends the base template for the My Book App web application and serves as the user interface for new user registration.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Registration page.
- Contains a form for users to create a new account by providing their username and password.

Structure:
----------
1. **Head Section**:
   - Title block set to "Registrierung" for the registration page.

2. **Body Section**:
   - **Container**: Main section for the registration form.
   - **Form**: Collects user input for account creation:
     - **Benutzername**: Input field for the username (required).
     - **Passwort**: Input field for the password (required).
   - **Submit Button**: A button labeled "Registrieren" to submit the form for account creation.

Author: name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Registrierung{% endblock %}

{% block content %}
    <div class="container">
        <h1>Registrieren</h1>
        <form method="POST" action="/register">
            <label for="username">Benutzername:</label>
            <input type="text" name="username" required>
            <br>
            <label for="password">Passwort:</label>
            <input type="password" name="password" required>
            <br>
            <button type="submit" class="btn">Registrieren</button>
        </form>
    </div>
{% endblock %}

<!--
HTML Template for Search Results Page - My Book App (For results.html)
---------------------------------------------
This HTML document extends the base template for the My Book App web application and displays the search results for books based on user queries.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Search Results page.
- Displays a list of books retrieved from the search query, allowing users to save their favorites.

Structure:
----------
1. **Head Section**:
   - Title block set to "Suchergebnisse" for the search results page.

2. **Body Section**:
   - **Container**: Main section for displaying the search results.
   - **Book List**: Conditionally rendered based on the presence of books:
     - **Form**: Allows users to add selected books to their favorites:
       - **Card for Each Book**: Displays details for each book, including:
         - Title
         - Author
         - ISBN
         - Publication Year
         - Description (toggleable with a button)
       - **Checkbox**: Option to add the book to favorites.
       - **Hidden Inputs**: Store book details for form submission.
     - **Submit Button**: A button labeled "Favoriten speichern" to save selected favorites.

3. **Fallback Message**: Displays a message if no books are found.

Author: Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Suchergebnisse{% endblock %}

{% block content %}
    <div class="container">
        <h1>Suchergebnisse</h1>
        {% if books %}
            <!-- Form to save selected favorites -->
            <form method="POST" action="/add_favorite">
                <div class="card-container">
                    <!-- Iterate through the list of books -->
                    {% for book in books %}
                        <div class="card">
                            <h2>{{ book.title }}</h2>
                            <p><strong>Autor:</strong> {{ book.author }}</p>
                            <p><strong>ISBN:</strong> {{ book.isbn }}</p>
                            <p><strong>Erscheinungsjahr:</strong> {{ book.publication_year }}</p>
                            
                            <!-- "More Info" button and description -->
                            <button type="button" class="btn-small" onclick="toggleDescription({{ loop.index }})">Mehr Infos</button>
                            <div id="description-{{ loop.index }}" class="book-description" style="display: none;">
                                <p>{{ book.description }}</p>
                            </div>
                            
                            <!-- Form with hidden fields for adding to favorites -->
                            <input type="hidden" name="title_{{ loop.index }}" value="{{ book.title }}">
                            <input type="hidden" name="author_{{ loop.index }}" value="{{ book.author }}">
                            <input type="hidden" name="isbn_{{ loop.index }}" value="{{ book.isbn }}">
                            <input type="hidden" name="publication_year_{{ loop.index }}" value="{{ book.publication_year }}">
                            <input type="hidden" name="category_{{ loop.index }}" value="{{ book.category }}">
                            <p>
                                <input type="checkbox" name="selected_books" value="{{ loop.index }}">
                                Zu Favoriten hinzufügen
                            </p>
                        </div>
                    {% endfor %}
                </div>
                <button type="submit" class="btn-large">Favoriten speichern</button>
            </form>
            
        {% else %}
            <p>Keine Bücher gefunden.</p>
        {% endif %}
    </div>
{% endblock %}

<!--
HTML Template for Book Search Page - My Book App (For search.html)
---------------------------------------------
This HTML document extends the base template for the My Book App web application and allows users to search for books based on their interests in various fields.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Book Search page.
- Contains a form for users to specify their areas of interest when searching for books.

Structure:
----------
1. **Head Section**:
   - Title block set to "Bücher suchen" for the search page.

2. **Body Section**:
   - **Container**: Main section for the search form.
   - **Search Form**: Users can select a field of interest and optionally enter a specific topic:
     - **Dropdown Menu**: Allows users to select from various academic fields (e.g., Biology, Technology, etc.), which is a required field.
     - **Text Input**: Optional field for users to specify a particular topic of interest.
     - **Submit Button**: A button labeled "Suchen" to initiate the search.

Author: Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Bücher suchen{% endblock %}

{% block content %}
    <div class="container">
        <h1>Bücher suchen</h1>
        <form method="POST" action="/search">
            <label for="field">Für welches Fachgebiet interessierst du dich?</label>
            <select name="field" required>
                <option value="Biologie">Biologie</option>
                <option value="Technologie">Technologie</option>
                <option value="Wirtschaft">Wirtschaft</option>
                <option value="Mathematik">Mathematik</option>
                <option value="Physik">Physik</option>
                <option value="Chemie">Chemie</option>
                <option value="Medizin">Medizin</option>
                <option value="Informatik">Informatik</option>
                <option value="Ingenieurwesen">Ingenieurwesen</option>
                <option value="Umweltwissenschaften">Umweltwissenschaften</option>
                <option value="Philosophie">Philosophie</option>
                <option value="Geschichte">Geschichte</option>
                <option value="Kunstwissenschaften">Kunstwissenschaften</option>
                <option value="Musikwissenschaft">Musikwissenschaft</option>
                <option value="Soziologie">Soziologie</option>
                <option value="Pädagogik">Pädagogik</option>
                <option value="Psychologie">Psychologie</option>
                <option value="Politikwissenschaft">Politikwissenschaft</option>
                <option value="Rechtswissenschaften">Rechtswissenschaften</option>
                <option value="Geografie">Geografie</option>
                <option value="Sportwissenschaften">Sportwissenschaften</option>
                <option value="Astronomie">Astronomie</option>
                <option value="Meteorologie">Meteorologie</option>
                <option value="Ozeanografie">Ozeanografie</option>
            </select>
            <br><br>
            <label for="topic">Gibt es ein bestimmtes Themen - Teilgebiet, das dich interessiert?</label>
            <input type="text" name="topic">
            <br><br>
            <button type="submit" class="btn-large">Suchen</button>
        </form>
    </div>
{% endblock %}

<!--
HTML Template for Displaying JSON Data - My Book App (For test_jason.html)
---------------------------------------------
This HTML document extends the base template for the My Book App web application and is designed to display the contents of a JSON file in a formatted manner.

Overview:
---------
- Inherits layout and styles from base.html.
- Provides a title block for the Test JSON Data page.
- Displays JSON data, formatted for readability.

Structure:
----------
1. **Head Section**:
   - Title block set to "Test JSON Data" for the page.

2. **Body Section**:
   - **Container**: Main section for displaying JSON contents.
   - **Header**: A heading titled "JSON File Contents" to inform users of the displayed data.
   - **Preformatted Text**: Utilizes the `<pre>` tag to preserve the formatting of the JSON data, making it easier to read.
     - The `favorites_data` variable is converted to JSON format using the `tojson` filter with indentation for better readability.

Author: Your Name
Date: 06.10.2024
Version: 0.1.0
License: Free
-->

{% extends "base.html" %}

{% block title %}Test JSON Data{% endblock %}

{% block content %}
    <div class="container">
        <h1>JSON File Contents</h1>
        
        <pre>{{ favorites_data | tojson(indent=4) }}</pre>  <!-- Formats and displays the JSON data -->
    </div>
{% endblock %}

