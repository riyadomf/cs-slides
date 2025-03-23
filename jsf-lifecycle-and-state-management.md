---
theme: seriph
layout: cover
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://images.unsplash.com/photo-1475257026007-0753d5429e10?q=80&w=2070&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
background-size: cover
# some information about your slides (markdown enabled)
title: Welcome to Jsf
info: |
  # Slide for Jsf Lifecycle and State management
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: fade
mdc: true
---

# JSF Lifecycle and State Management

Understanding the core concepts of JSF lifecycle and state management.


---

# Agenda

1. JSF Lifecycle Overview
2. JSF Lifecycle Phases
3. State Management in JSF
4. Client-Side vs Server-Side State Saving
5. Handling `ViewExpiredException`
6. Best Practices

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# JSF Lifecycle Overview

- JSF follows a **six-phase lifecycle** to process requests and render responses.
- The lifecycle ensures proper handling of user input, validation, model updates, and UI rendering.

---

# JSF Lifecycle Phases

### Get Request

1. **Restore View**
   - If the request is **new**, JSF creates an empty view.
   - If the request is a **postback**, JSF restores the view from `ViewState`.
2. **Render Response**
   - JSF renders the component tree into HTML.
   - Saves the state of the view for future requests.

### Postback Request

1. **Restore View**
   - Restores the component tree from `ViewState`.
2. **Apply Request Values**
   - Updates component values from request parameters.
3. **Process Validations**
   - Validates user input and converts data types.
4. **Update Model Values**
   - Updates backing beans with validated data.
5. **Invoke Application**
   - Executes action listeners and navigation.
6. **Render Response**
   - Renders the updated component tree into HTML.

---

# State Management in JSF

- **Why JSF Needs State Management**
  - HTTP is stateless, but JSF is stateful.
  - JSF needs to remember form inputs, button states, and dynamic changes.
  - Ensures consistency and prevents tampered requests.

- **How State is Saved**
  - **Client-Side**: State is serialized and stored in a hidden field (`javax.faces.ViewState`).
  - **Server-Side**: State is stored in the session, and only a unique ID is sent to the client.

---

# Client-Side vs Server-Side State Saving

| **Aspect**               | **Client-Side**                          | **Server-Side**                        |
|---------------------------|------------------------------------------|----------------------------------------|
| **Memory Usage**          | Low (state stored on client)             | High (state stored on server)          |
| **Network Bandwidth**     | High (state sent with each request)      | Low (only a unique ID sent to client)  |
| **Security**              | Less secure (state exposed to client)    | More secure (state stored on server)   |
| **View Expiry**           | No `ViewExpiredException`                | Risk of `ViewExpiredException`         |

---

# Handling `ViewExpiredException`

- **What is `ViewExpiredException`?**
  - Occurs when JSF cannot restore a view state (e.g., session expires or view limit is exceeded).

- **How to Handle It**
  - Redirect users to a friendly error page or the home page.
  - Example (`web.xml`):
    ```xml
    <error-page>
        <exception-type>javax.faces.application.ViewExpiredException</exception-type>
        <location>/error/viewExpired.xhtml</location>
    </error-page>
    ```

---

# Best Practices

1. **Minimize State Size**
   - Avoid storing large objects in the component tree or session.
2. **Use Client-Side State Saving**
   - Prevents `ViewExpiredException` and reduces server memory usage.
3. **Disable Browser Caching**
   - Add cache-control headers to ensure users see the latest content.
4. **Handle `ViewExpiredException` Gracefully**
   - Redirect users to a friendly error page or the home page.

---

# Visual Diagram: JSF Lifecycle

```mermaid
graph TD
    A[Restore View] --> B{Is New Request?}
    B -- Yes --> C[Render Response]
    B -- No --> D[Apply Request Values]
    D --> E[Process Validations]
    E --> F[Update Model Values]
    F --> G[Invoke Application]
    G --> H[Render Response]