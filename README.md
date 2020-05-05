# OAuth 2:

OAuth 2 is an authorization framework. It allows your application to access a user's account on an HTTP service, such as Google, Facebook, GitHub...etc.

The access you get is limited of course. If you've ever signed up with google or github, you've used OAuth 2.

## OAuth Flow:

### Roles:

Before we talk about the flow, we need to know the main roles in OAuth.
1- The client: The clien in OAuth is our application.
2- Resource Owner: This is the user that will sign up. Resource here refers to the user account information.
3- Resource Server: is the server responsible for holding and securing the user accounts such as Google or Facebook.

### Authorization Flow:

Now that we know the main roles in OAuth flow, let's see how it works. We will use google as example here.

1- When the resource owner (user) clicks the login/signup button, the client (our application) shows a popup screen requesting the permission of the user to use their google account for signup.

2- If the user agrees, they give the application an `authorization grant`. Which means the application can now request access to the user's information from google.

3- The application then requests an `access token` from google.

4- Google then issues the access token to our application, which means we have limited access to the user account. This includes user information, making api calls to other google services such as mail...etc.

5- The authorization is now complete as far as google is concerned. We will talk about more stuff as we move to the code part.

---

## Using Google OAuth API:

In order to use the Google OAuth API, we need to register an application with Google, and set it up. To do so, go to [this link](https://developers.google.com/identity/sign-in/web/sign-in).

You will find a blue button that says configure a project like the picture below. Click it.
![configure a project](https://i.imgur.com/hE5QKVH.png)

Create a new project. Enter a project name, then in the following step a product name, then it will ask you where you are calling the API from.

- select web server.
- Under Authorized redirect URIs, enter: `http://localhost:3000`

#### Redirect URIs:

redirect URIs are the endpoints that google is supposed to redirect the user to after signing up/logging in. Since we will be running locally, we want google to redirect the browser to localhost:3000.

Now we have created the application and registered it with google. We don't need to worry about the credentials it shows. We can access these any time we want from the developer console, which we will be using next.

Now One more step is left to complete the project setup. Let's head to the developer console.

### The Dev Console:

Go to the [google developer console](https://console.developers.google.com/). It should look like the picture below:
![google dev console](https://i.imgur.com/DhGbOFt.png)

In the navbar, you see the title Google APIs, and next to it you see in the picture above OAuth 2 experiment. This is the name of my registered API. You should see your project that you created. If not, click it and select your project from the drop-down menu.

- Now, in the left menu, go to credentials.

The page should look like below:
![credentials](https://i.imgur.com/PV5ixgf.png)

Under OAuth 2.0 Client IDs, you will find a `web client` and an OAuth client. Click the OAuth client.

You will find two input fields:

- **Authorized Origins**. This will be blank, which will cause a problem.
- **Authorized Redirect URIs**, which is what we filled during the creation process. It should be localhost:3000.

Authorized Origins is the server from which your application can send a request to google API. This should be the address on which our server is running. In our case, it is http://localhost:3000.

- Input http://localhost:3000 in the authorized origins field and save the changes.

Now we are ready to get started. Notice in this page you can also see on the right side of the page the `client ID` and the `client secret`. These are going to be important.

---

## Creating React App:

1- Setup:

- `mkdir react-OAuth && cd react-OAuth`
- `npx create-react-app client`
- `mkdir src && touch src/index.js`

For the first step, we will only deal with our react app. We will come back to `src` later.

So let's go to client and get to work on our OAuth.

- `cd client`.
- To use OAuth with react with no trouble, we need to use a library for it. `npm install react-google-login`.

The react-google-login is a react component that creates a login button and allows us to configure it to use google OAuth in our project.

In `App.js`, add the following:

```jsx
import { GoogleLogin } from 'react-google-login';

const successResponse = (response) => {
	console.log(response);
};

const failureResponse = (response) => {
	console.log('error', response);
};
```

and inside the App component add the following:

```jsx
<GoogleLogin
	clientId="YOUR_CLIENT_ID.apps.googleusercontent.com"
	buttonText="Login"
	onSuccess={successResponse}
	onFailure={failureResponse}
	cookiePolicy={'single_host_origin'}
/>
```

Your App.js should look like this now:

```jsx
import React from 'react';
import logo from './logo.svg';
import './App.css';
import { GoogleLogin } from 'react-google-login';

const successResponse = (response) => {
	console.log(response);
};

const failureResponse = (response) => {
	console.log('error', response);
};

function App() {
	return (
		<div className="App">
			<header className="App-header">
				<img src={logo} className="App-logo" alt="logo" />
				<p>
					Edit <code>src/App.js</code> and save to reload.
				</p>
				<a
					className="App-link"
					href="https://reactjs.org"
					target="_blank"
					rel="noopener noreferrer"
				>
					Learn React
				</a>
				<GoogleLogin
					clientId="YOUR_CLIENT_ID.apps.googleusercontent.com"
					buttonText="Login"
					onSuccess={successResponse}
					onFailure={failureResponse}
					cookiePolicy={'single_host_origin'}
					isSignedIn={true}
				/>
			</header>
		</div>
	);
}

export default App;
```

- make sure the `clientId` prop in the button is set to your client ID from the credentials tab in your developer console.
- the `isSignedIn` prop is used to keep the user logged in, so every time we refresh the page the authorization with google is automatically executed.
- Run the start script `npm start`.
  - if you get an error create a .env file inside the client directory and add `SKIP_PREFLIGHT_CHECK=true` to it. (this is a create-react-app internal conflict issue).
  - `echo SKIP_PREFLIGHT_CHECK=true >> .env`

You will now see a giant google login button. We don't need to worry about the style for now. Click it and check out the console.

- if you receive an error object with a message that says `idpiframe_initialization_failed`, it means that you haven't properly configured the `Authorized Origins` in the credentials tab in your console. Make sure to set it to localhost:3000. If it is set properly, it means you need to clear the browser cache (history).

Now you should be able to login to our react app with your google account. If you login, the console output will show your account object. It has basic information about your account.

- Modify the `successResponse` function as follows:

```js
const successResponse = (response) => {
	const { accessToken, tokenId } = response;
	console.log('access token', accessToken);
	console.log('token ID', tokenId);
};
```

Now we can see the access token and the tokenId of your user. For this workshop, we will not need access token. We will be using the `tokenId` to do some backend stuff.

Modify the `successRespnse` function again to send a post request to your backend. (We will create a simple express server in the next step, and use it to get your user detail from google.)

- `npm i axios`

```js
const successResponse = (response) => {
	const { tokenId } = response;

	axios.post('/login/google', { tokenId }).then(console.log).catch(console.log);
};
```

## Using Nodejs with React OAuth:

First, install express, body-parser and axios, and set up a simple server that listens on port 4000 in our src/index.js.

- `npm init -y`
- `npm i express axios body-parser`

```js
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');

const app = express();

// MIDDLEWARE
app.use(bodyParser.json({ extended: false }));

// ROUTES
app.post('/login/google', (req, res) => {
	console.log(req.body);
});

app.listen(4000, () => {
	console.log('server is listening on port 4000');
});
```

Now all we need is to see if our access token is sent to the server.

- Modify the client `package.json` and add the following key to it:
  - `"proxy": "http://localhost:4000"`
  - restart the react server.
  - run the nodejs server.

Now refresh the page and check the terminal console. It should log the tokenId.

### Obtaining User Account from the Server:

Now that we have the tokenId, we can obtain a lot of user details from google API. To do that, we need the google library for node.

- npm install google-auth-library
- Add the library to our server and the following code:

```js
const verify = async () => {
	const ticket = await client.verifyIdToken({
		idToken: tokenId,
		audience: CLIENT_ID,
	});
	const payload = ticket.getPayload();
	const userid = payload['sub'];
	// If request specified a G Suite domain:
	//const domain = payload['hd'];
};
verify().catch(console.error);

// request user details:
const idInfo = await axios.get(
	`https://oauth2.googleapis.com/tokeninfo?id_token=${req.body.tokenId}`
);
console.log(idInfo);
```

Our `index.js` should look like this by now:

```js
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');
const { OAuth2Client } = require('google-auth-library');

const app = express();

// client ID should come from process.env
const CLIENT_ID = 'YOUR_CLIENT_ID.apps.googleusercontent.com';
const client = new OAuth2Client(CLIENT_ID);

// MIDDLEWARE
app.use(bodyParser.json({ extended: false }));

// ROUTES
app.post('/login/google', async (req, res) => {
	const { tokenId } = req.body;
	const verify = async () => {
		const ticket = await client.verifyIdToken({
			idToken: tokenId,
			audience: CLIENT_ID,
		});
		const payload = ticket.getPayload();
		const userid = payload['sub'];
		// If request specified a G Suite domain:
		//const domain = payload['hd'];
	};
	verify().catch(console.error);

	// request user details:
	const idInfo = await axios.get(
		`https://oauth2.googleapis.com/tokeninfo?id_token=${req.body.tokenId}`
	);
	console.log(idInfo);
});

app.listen(4000, () => {
	console.log('server is listening on port 4000');
});
```

Now refresh your react app on localhost and check the console. The returned object should be much larger than the one we saw in the front end side. It includes your name, email and profile picture link. This is very useful as we can insert it into our database directly and link a user with a google account.

Before we continue let's explain the code above.

- The verify function is a function provided by the library itself.
  - It is used to verify with google API that your application has the correct permission to access the user data.
  - It does so by checking the tokenId (this is for the user) against your clientId (your application id).
  - Notice the verify function has a `.catch` but no `.then`. That's because if the verification is successful, it does not return a response.
- Once the user is verified, we then can send a request to google APIs with the idToken. This will get us the user account details.

---

## Recap:

- OAuth is an authentication framework that allows us to use services such as google and github to authenticate our users and manage their account security.
- In OAuth there are three main roles.
  - The resource owner (user).
  - The client (our app).
  - The authentication server (google/facebook...etc).
- Authorized Origins are the domains from which our app can send a request to Google APIs.
- Authorized Redirect URIs are the endpoints to which Google OAuth will redirect the user once the authorization process is complete.
- Access token is used by the client to send requests to Google APIs (such as mail or calendar).
- ID token is used to obtain user account information from Google.

## Challenge:

Now that we can view the user details. Mimick the behavior of a database.

- create a json file called `users.json`
- When we send a request, check if the user exist (check via email)
- If the user exists, return a response to the client that says ('welcome back')
- if the user does not exist, write a user object in a json file and return a response that says "you have successfully signed up".
- user object should include:
  - first name
  - last name
  - email
  - profile picture link

The `users.json` should hold only one user (your account) by the end of the challenge. No matter how many times you refresh your react app.

Good luck.
