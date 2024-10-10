
# Requirements
- Install dependencies:  
  ```bash
  pip install -r requirements.txt
  ```
- List of installed packages:  
  ```
  pip list
  ```
  (Include versions of the modules listed in the `requirements.txt`)
- To check the installed version of pip and the environment path, run:  
  ```bash
  pip -V
  ```


# Who We Are
- **Duc Thang Phung** - [Link to profile]
- **Paul Willy** - [Link to profile]
- **Tim Roesch** - [Link to profile]

# Description

## How to Start
1. Clone the repository:
   ```bash
   git clone https://github.com/TimR04/1
   ```
2. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the Flask app:
   ```bash
   python app.py
   ```
4. Open browser and navigate to:
   ```
   http://127.0.0.1:5000/
   ```

## What We Want to Show
- User Authentication: A secure system for user registration, login, and session management.
- Book Search Functionality: Users can search for books by entering their fields of interest and specific topics. The results are fetched from the Google Books API.
- Favorites System: Users can save their favorite books and view them later. Books can be categorized based on fields of interest.
- Reading Progress: The app allows users to save their current reading progress for each book.
- Learning Notes: Users can write and store personal learning notes for each book they have saved.
- Bookmarks: Users can bookmark books for future reference.
- Book Details: Each book includes title, author, ISBN, publication year, and a description.
- Book Recommendations: Users can add their favorite books to their collection and revisit them when needed.

## Special Operations
- Search Books by Topic: The app integrates with the Google Books API to retrieve books based on user preferences. The search is dynamic and includes fields such as interest and topic.
- Store Favorites in JSON: The user's favorite books are saved in a JSON file, allowing the user to easily manage and remove them later.
- Session Management: User sessions are managed through Flask's built-in session handling, ensuring secure access to user-specific data.
- Page Progress Update: Users can save their reading progress for each book. The app updates the page number of where the user left off reading.
- Learning Notes Storage: Personal notes for each book are stored and associated with the user’s account in the JSON file, so users can record important insights or information about books.
- Remove Favorites: Users can remove multiple selected favorites from their saved list in one operation.
- Authentication and Authorization: Secure login and session management are used to restrict access to user-specific actions like adding favorites or saving progress.

# Additional README Information
- The documentation is auto-generated from the code's docstrings using **mkdocs**. To build the docs, run:
  ```bash
  mkdocs build
  ```
# HTML Description
1. Navigation bar: contains links to various sections of the app:
   - Startseite: the home page
   - Bücher suchen: a page to search for books
   - Favoriten: a page displaying favorite books
   - Lesezeichen: a page for magaging bookmarks
   - Learnings: a page to track personal learnings
   - Logout: logs out the user when clicked (visible when logged in)
   - Login/Registrierung: Links for logging in or registering when the user is not logged in
2. Homepage
   - Welcome message: A brief introductory message welcoming users to the virtual library.
   - Streaks: Reading streak of the user is displayed
3. Search result page
   - Search input fields: 2 text boxes for users to enter their search query including field of expertise and more specifically a theme
   - Search button labeled "Suchen initiates the search
   - Results display: a section that lists the search results, including:
      - Book titles
      - Authors
      - "Mehr Infos" button for each book to access detailed information
4. Favorite page
   - Favorites List: Displays a list of books marked as favorites. Each entry includes:
      - Book title
      - Author
      - "Entfernen" button to remove the book from favorites
   - Filter Dropdown: A dropdown menu that allows users to filter favorites by categories, with a corresponding "Filtern" button.

5. Book management section
   - Current Page Input: A text input field for users to update the current page of a favorite book.
   - Aktualisieren Button: A button to save the updated page number for the selected book.
  
6. Learning management page
   - Learning Notes Section: Allows users to create and manage their learning notes.
   - Add Learning Note: An input field for entering a new learning note and an "Speichern" button to save it.

## Test Section
Navigation Tests

Test 1: Navigation Links Exist
Description: Verify that all navigation links (Startseite, Bücher suchen, Favoriten, Lesezeichen, Learnings) are present in the navigation bar.
Expected Result: All links should be visible and functional.

Test 2: Logout Link Visibility
Description: Verify that the logout link appears when the user is logged in.
Expected Result: The logout link should be visible.

Test 3: Login and Register Links Visibility
Description: Verify that the login and register links are displayed when the user is not logged in.
Expected Result: Both links should be visible.

Home Page Tests

Test 4: Home Page Loads Correctly
Description: Verify that the home page displays the correct title and introductory message.
Expected Result: The title should be "Willkommen in Ihrer virtuellen Lernbibliothek."

Test 5: Content Display on Home Page
Description: Check that relevant content is displayed on the home page.
Expected Result: The main content area should show welcome information and any featured sections.
Search Functionality Tests

Test 6: Search Functionality
Description: Input a book title in the search bar and click the search button.
Expected Result: The search results should display books matching the search criteria.

Test 7: No Results Found
Description: Search for a title that does not exist in the database.
Expected Result: A message indicating no results found should be displayed.

Test 8: More Info Button in Search Results
Description: Click the "Mehr Infos" button for a book in the search results.
Expected Result: Additional information for that book should be displayed.

Favorites Tests

Test 9: Adding a Book to Favorites
Description: Click the button to add a book to favorites from the search results.
Expected Result: The book should appear in the favorites section.

Test 10: Display of Favorites List
Description: Navigate to the favorites page and check that the list of favorite books displays correctly.
Expected Result: All added favorites should be displayed with relevant details.

Test 11: Filtering Favorites
Description: Select a category from the filter dropdown and click the "Filtern" button.
Expected Result: The favorites list should refresh and show only the books that match the selected category.

Test 12: Removing a Book from Favorites
Description: Click the remove button for a favorite book.
Expected Result: The book should no longer appear in the favorites list.
Learning Management Tests

Test 13: Adding Learning Notes
Description: Input a new learning note and click the add button.
Expected Result: The note should appear in the learning section.

Test 14: Display Learning Notes
Description: Navigate to the learnings page and check that all notes are displayed correctly.
Expected Result: The learning notes should show the title and content of each note.

Test 15: Updating Learning Notes
Description: Click the update button for a learning note and change the content.
Expected Result: The updated content should be displayed after submission.

Book Management Tests

Test 16: Updating Current Page of a Favorite Book
Description: Input a new page number for a favorite book and click the "Aktualisieren" button.
Expected Result: The page should reload, and the book's current page should reflect the updated value.

Test 17: Incorrect Page Number Submission
Description: Input an invalid page number (e.g., negative or non-numeric) and submit.
Expected Result: An error message should indicate invalid input.

Layout and Design Tests

Test 18: Responsive Design Check
Description: Resize the browser window and verify that the layout adapts correctly on different screen sizes.
Expected Result: The layout should remain user-friendly and functional on all screen sizes.

Test 19: Font and Styling Verification
Description: Check that the specified fonts and styles are applied throughout the app.
Expected Result: The app should display consistent styling as defined in the CSS.

Test 20: Button Visibility and Functionality
Description: Verify that all buttons (e.g., "Aktualisieren," "Filtern," "Mehr Infos") are visible and functional.
Expected Result: Each button should respond to user interactions as intended.

--- 

