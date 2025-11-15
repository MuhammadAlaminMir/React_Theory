### The 5-Step Guide to Thinking Declaratively in React

This process helps you transition from thinking about _how_ to manipulate the UI (imperative) to declaring _what_ the UI should look like for any given state (declarative).

---

### Step 1: Identify Your Component’s Different Visual States

Before you write any code, you must understand all the possible "faces" your component can show to the user. Think of yourself as a designer or a computer scientist.

- **The Concept:** In computer science, this is like a **state machine**. In design, these are different **visual mockups**. React lives at the intersection of these two fields. Your goal is to list every distinct visual state.

- **Example (A Contact Form):**

  - **Empty:** The form is visible, but the "Submit" button is disabled.
  - **Typing:** The user has entered text, and the "Submit" button is now enabled.
  - **Submitting:** The form fields and button are all disabled. A loading spinner is shown.
  - **Success:** The form is hidden, and a "Thank you" message is displayed.
  - **Error:** Same as the "Typing" state, but with an extra error message visible below the input field.

- **Best Practice:** For complex components, it can be incredibly helpful to create a "living styleguide" or a Storybook that displays all these states on a single page. This makes it easy to visualize and review every possibility before you even start coding.

---

### Step 2: Determine What Triggers Those State Changes

Once you know the _what_ (the states), you need to identify the _how_ (the transitions between them). What causes the UI to move from one state to another?

- **Two Types of Triggers:**

  1.  **Human Inputs:** Direct user interactions like clicking a button, typing in a field, or hovering over an element. These almost always require an **event handler** (e.g., `onClick`, `onChange`).
  2.  **Computer Inputs:** Events from the system, like a network response arriving, a timeout completing, or an image finishing loading.

- **Visualize the Flow:** A great technique is to draw a simple diagram. Draw each state as a labeled circle and each transition as an arrow. This helps you reason about the user flow and spot potential bugs or missing states long before you write a single line of code.

---

### Step 3: Represent the State in Memory with `useState`

Now, translate your visual states into React state variables. This is where you connect your design to your code.

- **The Goal: Simplicity is Key.** Each piece of state is a "moving piece." The fewer moving pieces you have, the simpler and less buggy your component will be. Aim to represent only the **essential** information.

- **Controlled Components:** When you provide a `value` prop to an input element (like `<input>` or `<textarea>`), it becomes a **controlled component**. This means React is now in charge of that input's value.

  - **Critical Rule:** For a controlled component to work, you **must** provide an `onChange` handler that updates the state. Without it, the input will be read-only because the user is trying to change a value that React is constantly re-rendering with the same state value.

  ```jsx
  // Correct: A controlled component
  const [answer, setAnswer] = useState("");
  <textarea
    value={answer} // React controls the value
    onChange={(e) => setAnswer(e.target.value)} // User can change it via this handler
  />;
  ```

---

### Step 4: Remove Any Non-Essential State Variables

This is a critical refactoring step. You need to ensure your state variables don't contain redundant or contradictory information. Your goal is to prevent a situation where the state in memory doesn't represent any valid UI you'd want a user to see.

Ask yourself these questions about your state variables:

- **Does this state cause a paradox?** Can you set up a combination of state variables that results in an impossible UI (e.g., `isSubmitting: true` and `isSuccess: true` at the same time)?
- **Is the same information available in another state variable?** Don't store the same data in two places.
- **Can you get the same information from the inverse of another state variable?** If you have `isEmpty`, you can probably derive it from `text.length === 0`. You don't need both.

#### **Example: Combining Mutually Exclusive States**

Imagine you have this state:
`const [isTyping, setIsTyping] = useState(false);`
`const [isSubmitting, setIsSubmitting] = useState(false);`
`const [isSuccess, setIsSuccess] = useState(false);`
`const [isError, setIsError] = useState(false);`

This is problematic because `isSuccess` and `isError` should never be `true` at the same time. A better approach is to use a single state variable to hold the current status.

```jsx
// Better: A single state for all mutually exclusive statuses
const [status, setStatus] = useState("empty"); // 'empty' | 'typing' | 'submitting' | 'success' | 'error'

// Now your logic is much simpler
const isTyping = status === "typing";
const isSubmitting = status === "submitting";
// ...and so on
```

---

### Step 5: Connect the Event Handlers to Set State

This is the final step where you bring everything to life. You connect the triggers you identified in Step 2 to the state variables from Step 3.

- **The Loop:**
  1.  An event happens (e.g., user clicks "Submit").
  2.  Your event handler function is called.
  3.  Inside the handler, you call a `set` function (e.g., `setStatus('submitting')`).
  4.  React detects the state change and re-renders the component.
  5.  The JSX in your render function uses the new state to display the correct visual state (from Step 1).

```jsx
function handleSubmit() {
  // Trigger: User clicks submit
  setStatus("submitting"); // Set state to 'submitting' visual state
  sendFormToServer().then((response) => {
    if (response.ok) {
      setStatus("success"); // Set state to 'success' visual state
    } else {
      setStatus("error"); // Set state to 'error' visual state
    }
  });
}

// In your JSX:
{
  status === "submitting" && <Spinner />;
}
{
  status === "success" && <SuccessMessage />;
}
{
  status === "error" && <ErrorMessage />;
}
```

By following these five steps, you force yourself to think declaratively, leading to components that are more predictable, easier to debug, and simpler to maintain.
