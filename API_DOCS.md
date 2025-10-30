

ECHOLINK – API v3 (Updated Docs – Registration Flow)  

*AI Phone Agents Using User’s Own Twilio Number – No WebSockets  
All interaction via TwiML + REST calls (text in/out)

BASE URL

[https://api.echolink.studio/v1](https://api.echolink.studio/v1)

AUTHENTICATION

Endpoint Type

Auth Required

Notes

Initial Connection (POST /connect-twilio)

No Auth

Issues the JWT for the user.

User Endpoints (/bots, /my-number, etc.)

Authorization: Bearer <JWT>

Uses the token obtained from /connect-twilio.

Twilio Webhooks (/voice, /voice-response)

No Auth

Called by Twilio.

NEW USER FLOW (Combined Registration & Auth)

Step

Endpoint

Action

Result

1 (Initial)

POST /connect-twilio

User registers identity (email, name) and connects Twilio credentials.

Returns JWT (access_token)

2 (API Access)

GET /my-number

User uses JWT to access protected resources.

API returns data.

ENDPOINTS

POST /connect-twilio  

Register User, Connect Twilio Account, and Issue JWT

This is the initial endpoint used by every new developer to set up their EchoLink account and receive their long-lived JWT for subsequent API access.

Request

{
  "first_name": "...",
  "last_name": "...",
  "email": "...",
  "account_sid": "AC...",
  "auth_token": "xxx",
  "phone_number_sid": "PN..."
}




Field

Required

Description

first_name

Yes

User's first name for account creation.

last_name

Yes

User's last name for account creation.

email

Yes

User's email (unique identifier).

account_sid

Yes

Twilio Account SID.

auth_token

Yes

Twilio Auth Token.

phone_number_sid

Yes

SID of the Twilio number to use.

Success Response (200)

{ 
  "message": "Connected", 
  "phone_number": "+14155551234",
  "access_token": "eyJhbGciOiJIUzI1NiI...",
  "token_type": "Bearer",
  "expires_in": 86400
}




GET /my-number  

Protected Endpoint

Request

GET /my-number
Authorization: Bearer <YOUR_JWT_HERE>




Response (200)

{ "phone_number": "+14155551234", "bots_count": 2 }




POST /bots  

Protected Endpoint

Request

POST /bots
Authorization: Bearer <YOUR_JWT_HERE>
Content-Type: application/json




{
  "goal": "Book table",
  "webhook": "[https://api.eatery.com/book](https://api.eatery.com/book)",
  "context": "Open 5-10PM. Tables: 2,4,6"
}




(Other protected bot endpoints like GET /bots, PATCH /bots/{id}, DELETE /bots/{id} follow the same authentication pattern.)

POST /voice – INITIAL TWILIO WEBHOOK

Called once per call when customer dials in. No Auth.

Request

POST /voice?bot_id=bot_abc123&CallSid=CA...
Content-Type: application/x-www-form-urlencoded




Success Response (200) – TwiML

<Response>
  <Say voice="man">Hi, welcome to your AI assistant. How can I help you today?</Say>
  <!-- ... rest of TwiML/Gather elements ... -->
</Response>




POST /voice-response – SPEECH RESULT WEBHOOK

Called every time user speaks. No Auth.

Request (from Twilio)

POST /voice-response?bot_id=bot_abc123&CallSid=CA...
Content-Type: application/x-www-form-urlencoded

SpeechResult=I+want+to+book+a+table+for+4
Confidence=0.95




Success Response (200) – TwiML

<Response>
  <Say voice="woman">Got it! For how many people?</Say>
  <!-- ... rest of TwiML/Gather elements ... -->
</Response>
