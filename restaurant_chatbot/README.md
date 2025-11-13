# AI Restaurant Reservation Chatbot


This is a fully automated AI chatbot designed for a restaurant's website. It acts as a 24/7 digital assistant, handling customer inquiries and managing table reservations.

The entire backend logic is built on a single, powerful **n8n.io** workflow, using a central AI Agent to route user intents and execute tasks.

## 🚀 Core Features

* **Intelligent Q&A:** Uses a **Pinecone** vector database to instantly answer customer questions about the menu, operating hours, location, dress code, etc.
* **Smart Table Reservations:** Allows users to book a table directly through the chat.
* **Real-time Conflict Prevention:** Before confirming, the bot uses **Google Sheets** as a database to check if the requested table, date, and time are already booked, preventing double-bookings.
* **Business Rule Enforcement:** The bot automatically checks requests against the restaurant's operating hours and table capacities (pulled from Pinecone).
* **Automated Email Confirmations:**
    * **To Customer (Success):** Sends an immediate confirmation email via **Gmail** if the booking is successful.
    * **To Customer (Failure):** Sends a polite email via **Gmail** if the slot is already taken.
* **Internal Admin Notifications:**
    * **To Restaurant:** Sends a separate email alert to the restaurant's admin address (via **Gmail**) for every new, successful booking.

## 🛠 Tech Stack

* **Workflow Automation:** n8n.io
* **AI Agent / LLM:** OpenAI (or your chosen model)
* **Vector Database (for RAG):** Pinecone
* **Booking Ledger (Database):** Google Sheets
* **Email Notifications:** Gmail API (x2 nodes)
* **Frontend:** Embedded Chat Widget (HTML/CSS/JS)

## Workflow: How It Works

The n8n workflow is built around a central AI Agent that performs "Intent Routing."

1.  **Receive Message:** The user sends a message from the website embed.
2.  **Intent Routing:** The AI Agent determines the user's goal:
    * **INTENT: General Question**
        * **Action:** Triggers the `knowledge_base_lookup` (Pinecone) tool.
        * **Result:** The bot answers the question ("Our Cacio e Pepe is $24.") and the workflow ends.
    * **INTENT: Book Reservation**
        * **Action:** Triggers the full `Reservation Workflow`.

### The Reservation Workflow

This is the core logic for booking a table:

1.  **Validate:** The bot uses `knowledge_base_lookup` (Pinecone) to check if the user's request is valid (e.g., "Are we open at 1 PM?" or "Can Table T1 fit 10 people?").
2.  **Gather:** The bot collects all required data: `Name`, `Email`, `Date`, `Time`, `PartySize`, `TableID`.
3.  **Normalize Data:** (A critical step!) It cleans the `Date` and `Time` into simple `YYYY-MM-DD` and `7:00 PM` formats to prevent timestamp errors.
4.  **Check for Conflicts:** The bot uses the **`google_sheet_lookup`** tool to query the 'Bookings' sheet. It searches for any existing row where `Date`, `Time`, AND `TableID` all match.
5.  **Execute (Conditional Logic):**
    * **FAILURE (Conflict Found):**
        * The lookup *finds a match*.
        * The bot informs the user in chat.
        * The **`gmail_send_customer`** tool is called to send the "Sorry, slot is taken" email.
    * **SUCCESS (No Conflict):**
        * The lookup *finds no match*.
        * The **`google_sheet_append`** tool is called to write the new reservation to the sheet.
        * The bot informs the user in chat.
        * The **`gmail_send_customer`** tool is called to send the "Confirmed!" email.
        * The **`gmail_send_admin`** tool is called to send the "New Booking Alert!" to the restaurant.

## Setup & Installation

1.  **Clone** this repository.
2.  **n8n Setup:**
    * Import the `workflow.json` into your n8n instance.
    * Create n8n credentials for OpenAI, Pinecone, Google Sheets, and two separate Gmail accounts (or one, if using the same for both).
3.  **Pinecone:** Create your vector index and upload your restaurant data (menu, hours, FAQs).
4.  **Google Sheets:** Create a new sheet named "Bookings" with the required columns (e.g., `Date`, `Time`, `TableID`, `GuestName`, `GuestEmail`, `PartySize`).
5.  **Configure Tools:**
    * In the n8n workflow, connect your credentials to each node.
    * In the `AI Agent` node, make sure your tool names (`gmail_send_customer`, `gmail_send_admin`) match the tool names defined in the prompt.
6.  **Embed:** Use the n8n chat embed code in your `index.html` file.


Workflow: 
<img width="1710" height="984" alt="Screenshot 2025-11-12 at 2 32 08 PM" src="https://github.com/user-attachments/assets/7db2a877-456b-4a84-be93-f91880a4d8ae" />
