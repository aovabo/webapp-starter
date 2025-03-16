Auth for Generative AI Applications
Enjoy securing your GenAI apps, with the developer experience Auth0 is known for.

Use Cases
Authenticate users
Easily implement login experiences, tailor made for AI agents. Whether for chatbots or background agents.

authenticate-users
Call APIs on users' behalf
Use secure standards to get API tokens for Google, Github and more... Seamlessly integrate your app with other products.

authenticate-users
Authorization for RAG
Only retrieve documents users have access to. Avoid leaking data to a user that should not have access to it.

authenticate-users
Async User Confirmation
Let your autonomous, async agents do work in the background. Use Async Auth to request approval when needed.

authenticate-users

Explained Prompt
: Show me forecast for ZEKO
Authorization for RAG
Only retrieve documents users have access to. Avoid leaking data to a user that should not have access to it.

Authorization for RAG
Scenario
When a user submits a forecast inquiry for a specific company, like Zeko, the chatbot will generate a response using relevant documents retrieved from the vector store. By default, Market0 will only include publicly available filings. However, users may also have access to analyst-level forecasts, providing them with additional insights when the response is generated.


Okta Fine Grained Authorization (FGA) is used to check which documents the user has access to based on their permissions.

How it works
User Forecast Request: The user requests a forecast for a specific company, such as ZEKO.
Document Retrieval: Market0 handles the request and employs a retriever to search its vector store for documents relevant to the requested information. It applies filters to ensure only the documents the user has access to are considered.
Response Generation: Based on the retrieved documents, Market0 compiles a tailored response for the user. Depending on user's permissions the response could be based on analyst-level forecasts.

Explained Prompt
: Show me ZEKO upcoming events
Call APIs on users' behalf
Use secure standards to get API tokens for Google, Github and more... Seamlessly integrate your app with other products.

Call APIs on users' behalf
Scenario
When a user requests information about upcoming events for a company, such as ZEKO, the chatbot will retrieve and display a list of relevant events. Additionally, it will offer to check the user's availability in their Google Calendar to add reminders for selected events.


To check availability, integration with the Google Calendar API is necessary, with Auth0 handling user authentication and authorization for seamless access.

How it works
Users Requests Upcoming Events: The user requests upcoming events for a specific company such as ZEKO.
Displaying Events and Availability Check: Market0 handles the user's request and displays the list of upcoming events. As part of the response, it offers an option for the user to check their availability in Google Calendar.
Third-Party Service Authorization: When the user clicks the "Check" button, Auth0 will handle the user's authorization to call the Google Calendar API on their behalf.
API Call on Behalf of User: Once authorized, the app checks the user’s Google Calendar availability, allowing users to add event reminders to their schedule.

Explained Prompt
: Buy ZEKO if P/E ratio is above 15
Async Human in the Loop
Let your autonomous, async agents do work in the background. Request user permissions when needed.

Async Human in the Loop
Scenario
When the user requests to buy a specific company stock when its P/E ratio is above 15, the chatbot will monitor this metric and seek the user's approval (authorization) before executing any transaction.


This process is streamlined through Auth0, utilizing the CIBA (client-initiated backchannel authentication) flow to facilitate asynchronous user approval as needed. This ensures the user consents to the operation before any purchase order is executed.

How it works
User Initiates Purchase Request: The user instructs the chatbot to monitor a specific company's stock, indicating a desire to purchase if the P/E ratio exceeds zero.
Monitoring and Notification: Market0 continuously tracks the company's P/E ratio. When it rises above zero, the system notifies the user (using CIBA) to approve the purchase.
User Confirmation: Upon notification, the user reviews the details and confirms whether to proceed with the transaction.
Purchase Completion: Once the user approves, Market0 completes the purchase of 10 shares of the specified company.