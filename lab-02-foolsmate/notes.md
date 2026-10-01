
# ♟️ TryHackMe: Fools Mate — Web Security Writeup

## 📌 Room Overview
* **Target:** Web Application (Chess Game Engine)
* **Category:** Web Security / Client-Side Logic Bypass
* **Difficulty:** Easy
* **Core Vulnerability:** Insecure Client-Side Trust & Missing Backend Validation

---

## 1. 🔍 Reconnaissance & Code Analysis

During initial interaction with the target web application, valid checkmating moves were blocked by a frontend warning message when attempted through the graphical user interface.

### Step 1: Investigating Frontend JavaScript
Using the browser's Developer Tools (**F12** → **Sources** / **Debugger** tab), I inspected the loaded scripts and identified `app.js`. 

Searching for state validation hooks revealed the following client-side restriction:

```javascript
function preMoveCheck(from, to, promotion) {
  // Client-side checkmate interception
  if (result && probe.isCheckmate()) {
    showSystemNotice("I'll shut down your PC if you play that.");
    return false; // Intercepts and cancels move execution
  }
}

```

### Step 2: Analyzing the Security Flaw

* The application relied entirely on `preMoveCheck()` inside `app.js` to prevent the user from completing the winning move.
* Because all JavaScript executes within the client's browser environment, the user maintains total control over code execution and data manipulation before it reaches the network level.

---

## 2. ⚡ Exploitation Methods

Two distinct techniques were used to bypass the restriction and submit the winning state.

### Method 1: In-Memory Function Overriding (Browser Console)

By leveraging the browser's Developer Console, I redefined `preMoveCheck` in memory to bypass the checkmate interceptor:

1. Open **Developer Tools** (`F12` or `Ctrl + Shift + I`) → Click the **Console** tab.
2. Execute the following override script:

```javascript
// Redefining the function to bypass the checkmate check
preMoveCheck = function(from, to, promotion) {
  return true; // Force-allow all moves regardless of checkmate state
};

```

3. Execute the winning move on the interactive board. The frontend logic allowed the move to proceed without triggering the notice.

---

### Method 2: Direct API Payload Manipulation (`cURL`)

To bypass browser execution controls entirely, I analyzed the application's network traffic under the **Network** tab when making regular moves.

#### 1. Network Observation

* **Endpoint:** `http://<TARGET_IP>/api/move`
* **HTTP Method:** `POST`
* **Header:** `Content-Type: application/json`
* **Payload Structure:** `{"from": "<square>", "to": "<square>"}`

#### 2. Crafting the Exploit

Using `cURL` in the terminal, I sent a handcrafted HTTP POST request containing the winning coordinates directly to the API endpoint, completely bypassing the frontend UI and JavaScript validation:

```bash
curl -X POST http://<TARGET_IP>/api/move \
  -H "Content-Type: application/json" \
  -d '{"from":"a1","to":"a8"}'

```

#### Command Breakdown:

* **`curl`**: Command-line tool for transferring data using network protocols.
* **`-X POST`**: Specifies the HTTP request method as POST.
* **`-H "Content-Type: application/json"`**: Sets the HTTP request header to indicate a JSON-formatted payload.
* **`-d '{"from":"a1","to":"a8"}'`**: Sends the JSON data payload representing the board positions directly to the backend.

---

## 3. 🛡️ Mitigation & Security Best Practices

### The Fundamental Rule of Web Security

> **Never trust user input or client-side execution.**

1. **Server-Side Validation:** All game rules, move validations, and state transitions must be verified on the backend server before updating state.
2. **Defense in Depth:** Treat client-side scripts purely as UI/UX components. Any security decision enforced in JavaScript should be treated as non-existent by backend APIs.

```

