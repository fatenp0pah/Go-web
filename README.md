# gowiki

### **1) Introducing the net/http package (an interlude)**

**Basic Server Setup:**

- **Uses `net/http` package** to create a simple web server.
- **`http.HandleFunc("/", handler)`**: Maps the root path (`"/"`) to the `handler` function.
- **`http.ListenAndServe(":8080", nil)`**: Starts the server on port 8080, logging any errors.


**Handler Function:**

- **`handler(w, r)`**: Defines a basic HTTP handler function.
- **`fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])`**: Sends a response using the URL path (drops the leading `/`).
 - **Example**: Visiting `http://localhost:8080/monkeys` responds with `Hi there, I love monkeys!`


**Serving Wiki Pages:**

- **Page Data**: Saved in `<title>.txt` files.

**viewHandler:**
- **Path Mapping**: Maps paths like `"/view/test"` to load data from `test.txt`.
- **`loadPage(title)`**: Reads file content based on the title.
- **Response**: Sends the page title and content as HTML in the response.
- **Improved `main`**: Maps `/view/` URLs to `viewHandler`.

**Steps to Run:**

- **Content Example**: Save "Hello world" in `test.txt`.
- **Run Commands**:
  - Build with `$ go build wiki.go`
  - Run with `$ ./wiki`
- **View Result**: Visit `http://localhost:8080/view/test` to see "Hello world" displayed on the page titled "test".


### **2)Editing Pages**
- **Add Handlers**: In `main()`, we define three handlers:
  - **`/view/`**: Displays the page.
  - **`/edit/`**: Displays an "edit page" form.
  - **`/save/`**: Saves the edited content.

- **`editHandler(w, r)`**: Loads the page (or creates an empty page if it doesn't exist) and displays an HTML form for editing.
  - The form sends a `POST` request to `/save/<title>`.
  - **HTML Structure**: 
    - Displays a `<textarea>` for editing content.
    - Includes a submit button to save the changes.

- **Improved `main`**: Adds the new handlers to the server.
  - **Code**:
    ```go
    func main() {
        http.HandleFunc("/view/", viewHandler)
        http.HandleFunc("/edit/", editHandler)
        http.HandleFunc("/save/", saveHandler)
        log.Fatal(http.ListenAndServe(":8080", nil))
    }
    ```

- **`editHandler(w, r)` Function**:
  - **Code**:
    ```go
    func editHandler(w http.ResponseWriter, r *http.Request) {
        title := r.URL.Path[len("/edit/"):]

        p, err := loadPage(title)
        if err != nil {
            p = &Page{Title: title}
        }

        fmt.Fprintf(w, "<h1>Editing %s</h1>"+
            "<form action=\"/save/%s\" method=\"POST\">"+
            "<textarea name=\"body\">%s</textarea><br>"+
            "<input type=\"submit\" value=\"Save\">"+
            "</form>", p.Title, p.Title, p.Body)
    }
    ```

- **Improvement**: The hard-coded HTML in `editHandler` can be made cleaner by separating it into an HTML template.
  
### **3)The html/template package**
  
** Using `html/template` Package**
- The `html/template` package is used to separate HTML from Go code.
- It helps keep the layout of pages modular and easier to maintain.

**Creating `edit.html` Template**
- The `edit.html` template is used to display an editable form for the page content.
- The template includes a form with a textarea for editing the body and a submit button to save changes.

** `editHandler(w, r)` with Template**
- The `editHandler` loads the page (or creates a new empty page if it doesn't exist) and renders the `edit.html` template with the page content.
- It uses the `html/template` package to execute the template.

**Using `html/template` for Safety**
- The `html/template` package ensures that user input is safe by automatically escaping any unsafe HTML tags.
- This helps prevent issues like HTML injection by replacing special characters (e.g., `>` with `&gt;`).

** Creating `view.html` Template**
- The `view.html` template displays the title and body of the page, along with an "edit" link.
- This template is used for viewing the content of the page.

** Refactoring with `renderTemplate(w, tmpl, p)`**
- A helper function `renderTemplate` is created to reduce code duplication between `viewHandler` and `editHandler`.
- This function takes care of rendering the appropriate template based on the page content.

** Modified Handlers**
- Both the `viewHandler` and `editHandler` are modified to use the `renderTemplate` function to render the templates and display the page content or editing form.

### **4) Handling Non-Existent Pages**
- If a page doesn't exist (e.g., `/view/APageThatDoesntExist`), it should redirect the user to the edit page instead of displaying empty HTML.
- The `http.Redirect` function is used to send a 302 HTTP status and a Location header to redirect to the appropriate page.

### **5) Saving Pages**
- The `saveHandler` processes form submissions from the edit page. It extracts the page title from the URL and the content from the form, creates a new `Page` object, and saves it.
- After saving the content, the user is redirected to the view page using `http.Redirect`.

### **6) Error Handling**
- Proper error handling is added to functions like `renderTemplate` and `saveHandler` to ensure that errors are not ignored and are communicated to the user.
- The `http.Error` function sends an HTTP status code and an error message to inform the user of any issues.

### **7) Template Caching**
- To improve performance, templates are parsed once during program initialization and stored in a global variable. 
- The `ExecuteTemplate` method is then used to render the templates, which avoids re-parsing them every time a page is rendered.

### **8) Validation**
- The title for each page is validated using a regular expression to ensure it only contains valid characters.
- The `getTitle` function checks the request URL against the regex and returns the page title if valid, otherwise returns a "404 Not Found" error.

### **9) Introducing Function Literals and Closures**
- To reduce code duplication, a wrapper function `makeHandler` is introduced. It abstracts the title validation and calls the appropriate handler (`viewHandler`, `editHandler`, `saveHandler`).
- The `makeHandler` function returns a closure that performs title extraction and validation before invoking the handler with the valid title.

### **10) Simplified Handler Functions**
- After wrapping handlers with `makeHandler`, the individual handler functions are simplified as they no longer need to handle title extraction and validation.


 
