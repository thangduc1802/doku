/*
CSS Stylesheet for Web Application
-----------------------------------
This stylesheet defines the overall styling for the web application, including layout, typography, buttons, forms, and responsive design.

Overview:
---------
- Uses the Google Font 'Roboto' for consistent typography.
- Sets global styles for body, headers, paragraphs, and links.
- Provides styling for containers, buttons, forms, and cards.
- Includes responsive design adjustments for smaller screens.

Styles:
-------
1. **General Settings**:
   - Body styling with font family, background color, and text color.
   - Heading styles for h1 and h2.

2. **Container Styles**:
   - `.container`: Centered layout with max-width.
   - `.home-container`: Flexbox layout for home page.

3. **Button Styles**:
   - `.btn-large` and `.btn-small`: Large and small button styles with hover effects.

4. **Form Styles**:
   - Input fields and buttons styled for consistency and accessibility.

5. **Card Design**:
   - `.card-container` and `.card`: Grid layout for displaying book information and effects on hover.

6. **Header and Footer**:
   - Header with navigation styling.
   - Footer with centered text and background color.

7. **Responsive Design**:
   - Media queries for smaller screen adjustments.

8. **Selected State**:
   - Styling for selected cards.

9. **Filter Form Styles**:
   - Styles for the filter form including labels and buttons.

Author: Paul, Tim, Thang
Date: 06.10.2024
Version: 0.1.0
License: Free
*/

@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

/* General settings */
body {
    font-family: 'Roboto', sans-serif;  /* Use Roboto for the entire page */
    background-color: #f0f2f5;
    margin: 0;
    padding: 0;
    color: #333;
}

h1, h2 {
    font-weight: 700;  /* Bold text for headings */
    color: #333;
}

h1 {
    font-size: 2em;
    margin-top: 20px;
    text-align: center;
}

h2 {
    font-size: 1.5em;
    margin-bottom: 15px;
}

p, label {
    font-weight: 400;  /* Normal weight for text and labels */
    color: #555;
    font-size: 1em;
}

a {
    text-decoration: none;
    color: inherit;
}

/* Centered layouts */
.container {
    width: 90%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    text-align: center;
}

.home-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 80vh;
}

.home-buttons {
    display: flex;
    justify-content: center;
    gap: 20px;
    margin-top: 40px;
}

/* Large buttons on the home page */
.btn-large {
    padding: 20px 40px;
    font-size: 1.5em;
    background-color: #007BFF;
    color: white;
    border-radius: 8px;
    transition: background-color 0.3s ease;
}

.btn-large:hover {
    background-color: #0056b3;
}

/* Smaller buttons for more info */
.btn-small {
    padding: 8px 16px;
    font-size: 0.9em;
    background-color: #007BFF;
    color: white;
    border-radius: 4px;
    transition: background-color 0.3s ease;
}

.btn-small:hover {
    background-color: #0056b3;
}

/* Form styles */
input[type="text"],
input[type="password"],
select {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-family: 'Roboto', sans-serif;
    font-size: 1em;
}

button {
    font-family: 'Roboto', sans-serif;
    font-size: 1.1em;
    padding: 10px 20px;
    background-color: #007BFF;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s;
}

button:hover {
    background-color: #0056b3;
}

/* Card design for search results and favorites */
.card-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
    margin-top: 20px;
}

.card {
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    padding: 20px;
    text-align: left;  /* Left-align text */
    transition: transform 0.3s ease;
}

.card:hover {
    transform: translateY(-10px);
}

.card h2 {
    color: #007BFF;
    font-size: 1.5em;
}

.card p {
    color: #555;
    font-size: 1em;
    margin: 5px 0;
}

/* Book description (initially hidden) */
.book-description {
    margin-top: 10px;
    padding: 10px;
    background-color: #f9f9f9;
    border-left: 4px solid #007BFF;
    display: none; /* Initial state: hidden */
    color: #555;
    font-size: 0.95em;
}

/* Header */
header {
    background-color: #007BFF;
    padding: 10px 0;
    text-align: center;
}

header nav ul {
    list-style-type: none;
    padding: 0;
}

header nav ul li {
    display: inline;
    margin: 0 15px;
}

header nav ul li a {
    color: white;
    text-decoration: none;
    font-size: 1.1em;
}

/* Footer */
footer {
    text-align: center;
    padding: 10px 0;
    background-color: #343a40;
    color: #ffffff;
    width: 100%;
    margin-top: 40px;
}

/* Responsive design for smaller screens */
@media (max-width: 600px) {
    .home-buttons {
        flex-direction: column;
        gap: 15px;
    }

    .btn-large {
        font-size: 1.2em;
        padding: 15px 30px;
    }

    .card-container {
        grid-template-columns: 1fr;
    }
}

/* Highlight selected tiles */
.card.selected {
    border: 2px solid #007BFF;
    background-color: #e6f7ff;
}

/* Margin below the cards for the remove button */
.btn-large {
    margin-top: 20px;  /* Adds some space above the button */
    display: block;
}

/* Filter form styling */
.filter-form {
    text-align: left;
    margin-bottom: 20px;
}

.filter-form label {
    font-weight: 500;
    margin-right: 10px;
}

.filter-form select {
    padding: 8px;
    font-size: 1em;
    margin-right: 10px;
}

.filter-form .btn-small {
    padding: 8px 16px;
    font-size: 0.9em;
    background-color: #007BFF;
    color: white;
    border-radius: 4px;
    transition: background-color 0.3s ease;
}

.filter-form .btn-small:hover {
    background-color: #0056b3;
}