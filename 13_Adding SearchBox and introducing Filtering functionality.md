# Adding a Search Box to Filter Monsters in React

This example introduces a search box that filters the list of monsters dynamically as the user types. The search functionality is implemented using an input box with event handling and React state management.

---

## **Code Overview**

### **Full Code:**
```javascript
import { Component } from 'react';

import logo from './logo.svg';
import './App.css';

class App extends Component {
  constructor() {
    super();
    this.state = {
      monsters: [],
    };
    console.log('rendering 1');
  }

  componentDidMount() {
    console.log('rendering 3');
    fetch('https://jsonplaceholder.typicode.com/users')
      .then((response) => response.json())
      .then((users) =>
        this.setState(
          () => {
            return { monsters: users };
          },
          () => {
            console.log(this.state);
          }
        )
      );
  }

  render() {
    console.log('rendering 2');
    return (
      <div className="App">
        <input
          className="search-box"
          type="search"
          placeholder="Search monsters"
          onChange={(event) => {
            console.log(event.target.value);
            const searchString = event.target.value.toLocaleLowerCase();
            const filteredMonsters = this.state.monsters.filter((monster) => {
              return monster.name.toLocaleLowerCase().includes(searchString);
            });

            this.setState(() => {
              return { monsters: filteredMonsters };
            });
          }}
        />
        {this.state.monsters.map((monster) => {
          return (
            <div key={monster.id}>
              <h1>{monster.name} </h1>
            </div>
          );
        })}
      </div>
    );
  }
}

export default App;
```


### **Key Code Snippet:**
```javascript
<input
  className="search-box"
  type="search"
  placeholder="Search monsters"
  onChange={(event) => {
    console.log(event.target.value);
    const searchString = event.target.value.toLocaleLowerCase();
    const filteredMonsters = this.state.monsters.filter((monster) => {
      return monster.name.toLocaleLowerCase().includes(searchString);
    });

    this.setState(() => {
      return { monsters: filteredMonsters };
    });
  }}
/>
```

### **Key Points Explained:**

#### **Why Use `className="search-box"` Instead of `class="search-box"`?**
- **Reason:** In React, `className` is used instead of `class` to avoid conflict with the JavaScript `class` keyword.
- React internally maps `className` to the `class` attribute in the generated HTML.

#### **Why Use `type="search"`?**
- **Purpose:** The `type="search"` input is optimized for search operations. It:
  - **Provides a Clear Button**: On some browsers (e.g., Chrome, Safari), a clear button appears inside the search box, allowing users to quickly clear the input.
  - **Supports Accessibility**: The `type="search"` is semantically meaningful for screen readers, improving usability for users with assistive technologies.
  - **Optimized Behavior**: Some browsers apply unique styling or behaviors (e.g., specific keyboard layouts on mobile devices) tailored to search operations.

#### **Why Use `placeholder="Search monsters"`?**
- **Purpose:** The placeholder text guides users on what they can type into the input box.

---

## **Event Handling Explanation**

### **React Events vs. Browser Events**
- React wraps native browser events in a **synthetic event** for performance optimization and cross-browser compatibility.
- **How to Identify a React Event:**
  - Log the event object in the console (`console.log(event)`) and inspect it.
  - React events include properties like `nativeEvent`, `persist()`, and `_reactName`.

### **Why Avoid Properties with `_`?**
- Properties with `_` (e.g., `_reactName`) are internal to React and should not be accessed or modified.
- **Reason:** These are private and subject to change in future React versions, which can break your code if used improperly.

### **`event.target.value` Explained**
- **Purpose:** `event.target.value` retrieves the current value of the input field as the user types.
- **How It Works:**
  - The value gets updated with every keystroke.
  - This allows dynamic filtering based on the latest input.
- **Example Flow:**
  - User types `A` → `event.target.value` is `"a"`.
  - User types `Al` → `event.target.value` is `"al"`.

---

## **Current Functionality and Issues**

### **Current Functionality:**
1. The user types into the search box.
2. The input value (`event.target.value`) is converted to lowercase.
3. The monsters are filtered based on whether their name includes the search string.
4. The filtered monsters are displayed dynamically.

### **Identified Issue:**
- **Problem:** Once filtered, the original list of monsters cannot be restored.
- **Cause:** The `setState` method updates the `monsters` array directly, overwriting the original data.

### **Solution Preview:**
In the next tutorial, we will:
- Maintain a separate copy of the original `monsters` list in the state.
- Use it as the source for filtering, ensuring the original list remains intact.

---

## **Key Takeaways**

1. **`className` vs. `class`:** Use `className` in React to avoid conflicts with JavaScript keywords.
2. **React Synthetic Events:** React provides synthetic events for consistency and optimization.
3. **Avoid `_` Properties:** Do not access or modify React's internal properties prefixed with `_`.
4. **Dynamic Filtering:** Use `event.target.value` to dynamically update the filtered list based on user input.
5. **Identifying Issues:** Understand the side effects of overwriting the state and plan to maintain a source of truth for data.

By addressing the identified issue in the next steps, we can create a robust and functional search feature.


# Explanation of the Search Fix in React

In the updated code, we have resolved the issue where the original list of monsters couldn’t be restored after filtering. This fix ensures that the list dynamically updates based on the current search input and restores the original list when the input is cleared.

---

## **Key Changes in the Fix**

### **1. Introduction of `searchString` in State**
```javascript
this.state = {
  monsters: [],
  searchString: '',
};
```
- **Why?**
  - A separate `searchString` state is added to hold the value of the search input.
  - The `monsters` array remains untouched and always holds the original list fetched from the API.
  - Filtering is based on the `searchString`, ensuring the original data is not overwritten.

---

### **2. Filtering Logic Moved to `render`**
```javascript
const filteredMonsters = this.state.monsters.filter((monster) => {
  return monster.name.toLocaleLowerCase().includes(this.state.searchString);
});
```
- **How It Works:**
  - The `monsters` array is filtered based on whether `monster.name` includes the `searchString`.
  - Since `searchString` is updated dynamically as the user types, the `filteredMonsters` list is recalculated before every render.
  - If `searchString` is empty, the entire list of monsters is displayed as no filter is applied.

- **Why This Fixes the Issue:**
  - The `monsters` array remains unmodified.
  - This ensures that deleting characters in the search box restores the original list.

---

### **3. Handling Input Change with `onChange`**
```javascript
<input
  className="search-box"
  type="search"
  placeholder="Search monsters"
  onChange={(event) => {
    console.log(event.target.value);
    const searchString = event.target.value.toLocaleLowerCase();
    this.setState(() => {
      return { searchString };
    });
  }}
/>
```
- **`event.target.value`:** Retrieves the current value of the input field as the user types.
- **Lowercasing with `toLocaleLowerCase()`:** Ensures case-insensitive filtering.
- **Updating `searchString`:**
  - The `setState` call updates the `searchString` in the component’s state.
  - This triggers a re-render where the filtering logic dynamically recalculates the `filteredMonsters` array.

---

## **Key Improvements**

### **1. Separation of Concerns**
- The `monsters` array remains unaltered.
- The `searchString` state is solely responsible for determining the displayed list of monsters.

### **2. Dynamic Updates**
- As the user types or deletes characters:
  - The `searchString` state updates.
  - The UI reflects the filtered results or the full list dynamically.

### **3. Improved Maintainability**
- With this fix, additional filters or enhancements (e.g., filtering by multiple fields like `email` or `address`) can be added easily without altering the original data structure.

---

## **How It Works: Example Flow**

1. **Initial State:**
   - `monsters`: Fetched data from the API.
   - `searchString`: `''` (empty).
   - **Result:** The full list of monsters is displayed.

2. **User Types "a":**
   - `searchString`: `"a"`.
   - `filteredMonsters`: Only monsters with "a" in their names are displayed.

3. **User Deletes Input:**
   - `searchString`: `''` (empty again).
   - `filteredMonsters`: The original list of monsters is displayed because no filtering condition applies.

---

## **Console Log Observations**

1. **`console.log('constructor')`:**
   - Logs when the component is initialized.
2. **`console.log('componentDidMount')`:**
   - Logs when the component is mounted, indicating the API call is being made.
3. **`console.log(event.target.value')`:**
   - Logs the current input value as the user types, helping to debug the search functionality.

---

## **Summary of the Fix**

- **Problem:** Filtering modified the `monsters` array, making it impossible to restore the original list.
- **Solution:** Introduced a `searchString` state to handle filtering dynamically without altering the original `monsters` array.
- **Benefits:**
  - The original list remains intact.
  - The list updates dynamically based on user input.
  - Improved separation of concerns for easier maintenance and future enhancements.

By resolving this issue, we ensure a more robust and dynamic search functionality. Let me know if you’d like further clarifications!



### **Fully Fixed Code:**
```
import { Component } from 'react';

import logo from './logo.svg';
import './App.css';

class App extends Component {
  constructor() {
    super();
    this.state = {
      monsters: [],
      searchString: '',
    };
    console.log('constructor');
  }

  componentDidMount() {
    console.log('componentDidMount');
    fetch('https://jsonplaceholder.typicode.com/users')
      .then((response) => response.json())
      .then((users) =>
        this.setState(
          () => {
            return { monsters: users };
          },
          () => {
            console.log(this.state);
          }
        )
      );
  }

  render() {
    console.log('render');

    const filteredMonsters = this.state.monsters.filter((monster) => {
      return monster.name.toLocaleLowerCase().includes(this.state.searchString);
    });

    return (
      <div className="App">
        <input
          className="search-box"
          type="search"
          placeholder="Search monsters"
          onChange={(event) => {
            console.log(event.target.value);
            const searchString = event.target.value.toLocaleLowerCase();
            this.setState(() => {
              return { searchString };
            });
          }}
        />
        {filteredMonsters.map((monster) => {
          return (
            <div key={monster.id}>
              <h1>{monster.name} </h1>
            </div>
          );
        })}
      </div>
    );
  }
}

export default App;

```
