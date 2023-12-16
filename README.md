![CI logo](https://codeinstitute.s3.amazonaws.com/fullstack/ci_logo_small.png)

Based on the frontend steps for building the moments fullstack app [in this repo](https://github.com/Code-Institute-Solutions/moments).

## Setting up the project

Bootstrap [4.6 getting started](https://react-bootstrap-v4.netlify.app/)

```sh
npm install react-bootstrap@1.6.3 bootstrap@4.6.0
```

Add ES7+ React/Redux/React-Native snippets v4.4.3

[Font awesome](https://fontawesome.com/) [free](https://fontawesome.com/search?m=free&o=r) icons.

[Css Modules in React](https://medium.com/@ralph1786/using-css-modules-in-react-app-c2079eadbb87)

[Adding a CSS Modules Stylesheet](https://create-react-app.dev/docs/adding-a-css-modules-stylesheet/)

# Routing

Instead of the command ```npm install react-router-dom``` use this ```npm install react-router-dom@5.3.0```

I'm so used to seeing example code that looks like this:

```js
<Switch>
    <Route exact path="/" render={() => <h1>Home page</h1>} />
    <Route exact path="/signin" render={() => <h1>Sign in</h1>} />
    ...
</Switch>
```

and having to convert that to router 4:

```js
<Routes>
    <Route path="/" element={<section>...</section>} />
    <Route path="/posts/:postId" element={<SinglePostPage />} />
```

Just thought I would point that out in case it helps someone.

## Authentication

Provide the Heroku keys:

```txt
CLIENT_ORIGIN = the react app url.
CLIENT_ORIGIN_DEV = the gitpod url.
```

Import axios into the frontend:

```sh
npm i axios
```

Create an src/api/axiosDefaults.js

```js
import axios from "axios";

axios.defaults.baseURL = "https://drf-api-rec.herokuapp.com/";
axios.defaults.headers.post["Content-Type"] = "multipart/form-data";
axios.defaults.withCredentials = true;

export const axiosReq = axios.create();
export const axiosRes = axios.create();
```

### Bootstrap forms

Add/update these style sheets:

- App.module.css
- Button.module.css
- SignInUpForm.module.css

Add the pages/auth directory with [this file](https://github.com/Code-Institute-Solutions/moments-starter-code/blob/master/06-starter-code/SignUpForm.js).

[Bootstrap forms](https://react-bootstrap-v4.netlify.app/components/forms/) have sample code that was modified a bit to look like the example below.

To hide elements *use the .d-none class or one of the .d-{sm,md,lg,xl}-none classes for any responsive screen variation* (from [these docs](https://getbootstrap.com/docs/4.4/utilities/display/)).

```tsx
<Form.Group controlId="username">
    <Form.Label className="d-none">username</Form.Label>
    <Form.Control
        className="styles.Input"
        type="text"
        placeholder="username"
        name="username"
        value={username}
        onChange={handleChange}
    />
</Form.Group>
```

### The submit handler

POST the sign-up form to the API and redirect to the sign-in page using React history.push method.

Set the form onSubmit function to call the event handler.

I have also added some notes on the nifty use of computed properties, destructuring and the spread operator.

```js
import { Link, useHistory } from "react-router-dom";
...

const SignUpForm = () => {
    /** Initialize the field variables with computed properties. */
    const [signUpData, setSignUpData] = useState({
        username: "",
        password1: "",
        password2: "",
    });
    // restructure the signup data so that the dot notation is not needed to access them
    const { username, password1, password2 } = signUpData;
    const [errors, setErrors] = useState({});

    /** To redirect to the sign in page after the registration call */
    const history = useHistory();

    /**
     * Instead of writing separate handlers for each variable, this function
     * sets the [property]=value pair using a spread operator to copy the other fields
     * and the left-hand bracket notation to set the appropriate property name.
     * @param {*} event
     */
    const handleChange = (event) => {
        setSignUpData({
            ...signUpData,
            [event.target.name]: event.target.value,
        });
    };

    const handleSubmit = async (event) => {
      event.preventDefault(); // so the page doesn't refresh as in the old HTML norm
      try {
        await axios.post("/dj-rest-auth/registration/", signUpData);
        history.push("/signin");
      } catch (err) {
        setErrors(err.response?.data);
      }
    };

    return (
      <Row className={styles.Row}>
          <Col className="my-auto py-2 p-md-2" md={6}>
              <Container className={`${appStyles.Content} p-4 `}>
                  <h1 className={styles.Header}>sign up</h1>

                  <Form onSubmit={{ handleSubmit }}>
                  ...
                   </Form.Group>
                  {errors.username?.map((message, idx) => (
                    <Alert variant="warning" key={idx}>
                      {message}
                    </Alert>
                  ))}
```

The submit button can show errors when the password don't match with a null_field_error:

```js
<Button
    className={`${btnStyles.Button} ${btnStyles.Wide} ${btnStyles.Bright}`}
    type="submit"
>
    Sign up
</Button>
{errors.non_field_errors?.map((message, idx) => (
    <Alert key={idx} variant="warning" className="mt-3">
        {message}
    </Alert>
))}
```

## Creating the SignIn form

This is a challenge section with the following brief:

“As a user I can sign in to the app so that I can access functionality for logged in users”

- The form
- Handle Events
- Handle Errors

### The form

- A field for the Username
- A field for the Password
- A Sign In Button

And the basic setup steps:

1. Create SignInForm.js in "src/pages/auth"
2. Import it into App.js
3. Add "/signin" to the route render prop

### Handling Events

- Handle Changes from the Username or Password inputs
- Handle a Form Submit Event
- Redirect to the Sign In page, once a form has successfully been submitted.

### Handling Errors for SignInForm

- Errors on the Username Field
- Errors on the Password Field
- Non_field_errors, such as providing incorrect credentials.

## The useContext hook

To make these getters and setters available to all children of the providers, here are the basics.

[src/App.js](https://github.com/mr-fibonacci/moments/blob/a6d063e846e748d68b203b7d8f2d76068a1ccb4a/src/App.js)

```js
import { createContext, useEffect, useState } from "react";
import axios from "axios";

export const CurrentUserContext = createContext();
export const SetCurrentUserContext = createContext();

function App() {
  const [currentUser, setCurrentUser] = useState(null);

  const handleMount = async () => {
    try {
      const { data } = await axios.get("dj-rest-auth/user/");
      setCurrentUser(data);
    } catch (err) {
      console.log(err);
    }
  };

  useEffect(() => {
    handleMount();
  }, []);

  return (
    <CurrentUserContext.Provider value={currentUser}>
      <SetCurrentUserContext.Provider value={setCurrentUser}>
```

This feature is used to update the user on login and change the navbar accordingly.

### Uncaught TypeError: Cannot read property 'post' of undefined

On a side note, if you see the above error, its just the wrong import format of axios.

Instead of this 'named import':

```js
import { axios } from "axios";
```

Use this 'default import':

```js
import axios from 'axios';
```

A file can only have one default export, but as many named exports as you'd like.

### Using a provider context

In the src/pages/auth/SignInForm.js file, this is how the useContext hook is used with the SetCurrentUserContext defined in the App.js.

```js
import React, { useContext, useState } from "react";
...
import { SetCurrentUserContext } from "../../App";

function SignInForm() {
  const setCurrentUser = useContext(SetCurrentUserContext);

  ...

      const handleSubmit = async (event) => {
        event.preventDefault();
        try {
            const { data } = await axios.post("/dj-rest-auth/login/", signInData);
            setCurrentUser(data.user);
            history.push("/");
        } catch (err) {
            setErrors(err.response?.data);
        }
    };

```

A similar thing is done in the navbar to use the logged in username:

```js
import { CurrentUserContext } from "../App";

const NavBar = () => {
  const currentUser = useContext(CurrentUserContext);
```

In the navar we also now use loggedInIcons loggedOutIcons variables to simplify the UI.  We don't want to show the sign-in/up icons after a user is logged in.

The more business logic and if/else boolean flags used in the UI, the harder it is to read and add features to without breaking something else.  So these variables simplify that with the ternary logic happening in one place:

```js
{currentUser ? loggedInIcons : loggedOutIcons}
```

### Functions with arrows and standards

The above code shows a bit of inconsistency in showing code with the function format when it used the arrow version in the sign up component.

The functional component versus the version with a 'fat' arrow version (sometimes also called a lambda):

#### Snippet: rfce (react functional component w/ export default at the bottom)

```js
function SignInForm() {
```

#### Snippet: rafce (react component utilizing an arrow function)

```js
const SignUpForm = () => {
```

Its best to follow a consistent pattern in a code base.  This stops developers wasting time arguing and re-working code that doesn't change the function of the code in anyway.  Things like formatting and arguments about tabs versus spaces just waste precious time.

Using Prettier to take formatting off the table is a great idea.  Use the default settings unless there is agreement among developers on a specific project that there is a reason to change something.  Manually fixing indentation is also a waste of time.

An example of this is when I started work on a project with a sizable pre-existing code base which had tab size set to 4.

The default Prettier tab size is 2.  We decided to stick with 4 as the only exception to the defaults.

This was done by having this is .vscode\settings.json

```json
{
    "prettier.tabWidth": 4,
}
```

In user settings, it chang be changed for all projects:

Prettier: Tab Width: 2

In this way all projects except the one with an exception can use the default.

## Custom hooks

Here a new directory contexts/CurrentUserContext.js is created to hold the context handleMount and useEffect and context object code from the App.js file shown in the previous section.

The new component then gets used higher up in the index.js file like this:

```js
ReactDOM.render(
  <React.StrictMode>
    <Router>
      <CurrentUserProvider>
        <App />
      </CurrentUserProvider>
    </Router>
  </React.StrictMode>,
  document.getElementById("root")
);
```

Along with the code from the App.js file, two more custom hooks are added to the CurrentUserContext.js file:

```js
export const useCurrentUser = () => useContext(CurrentUserContext);
export const useSetCurrentUser = () => useContext(SetCurrentUserContext);
```

This is done to make accessing current use and set current user easier.  They are then used like this:

```js
import { useSetCurrentUser } from "../../contexts/CurrentUserContext";

function SignInForm() {
  const setCurrentUser = useSetCurrentUser();
```

## Refresh tokens

The data returned from the server on login looks like this:

```json
{
    "access_token": "eyJ0eXA...",
    "refresh_token": "eyJ0eXA...",
    "user": {
        "pk": 14,
        "username": "asdf",
        "email": "",
        "first_name": "",
        "last_name": "",
        "profile": {
            "id": 14,
            "owner": "asdf",
            "created_at": "12 Dec 2023",
            "updated_at": "12 Dec 2023",
            "name": "",
            "content": "",
            "image": "https://res.cloudinary.com/...",
            "is_owner": true,
            "following_id": null,
            "posts_count": 0,
            "followers_count": 0,
            "following_count": 0
        },
        "profile_id": 14,
        "profile_image": "https://res.cloudinary.com/..."
    }
}
```

JWT access tokens only last for five minutes.  The refresh token for one day.

### axios interceptors

The source code for these changes can be found [here](https://github.com/mr-fibonacci/moments/blob/a981c39da1671a70023a3d6f3cf1410164e84e06/src/contexts/CurrentUserContext.js).

Interceptors can:

- automatically intercept both requests and responses from our API
- run custom code before they are passed on.

That will happen in src\contexts\CurrentUserContext.js where a useMemo hook used.

useMemo is usually used to cache complex values that take time to compute.  
here it is used because want to attach the interceptors before the children mount,  
as that’s where we’ll be using  them and making the requests from.

```js
  useMemo(() => {
    axiosReq.interceptors.request.use(
      async (config) => {
        try {
          // try to refresh the token before sending the request
          await axios.post("/dj-rest-auth/token/refresh/");
        } catch (err) {
          setCurrentUser((prevCurrentUser) => {
            if (prevCurrentUser) {
              // if that fails and the user was previously logged in it means that the refresh token has expired
              // so redirect the user to the SignIn page and set currentUser to null
              history.push("/signin");
            }
            return null;
          });
          return config;
        }
        return config;
      },
      (err) => {
        // if there's an error reject the Promise with the returned config
        return Promise.reject(err);
      }
    );

    axiosRes.interceptors.response.use(
      (response) => response,
      async (err) => {
        if (err.response?.status === 401) {
          try {
            await axios.post("/dj-rest-auth/token/refresh/");
          } catch (err) {
            setCurrentUser((prevCurrentUser) => {
              // if the user was logged in redirect to signin and set data to null
              if (prevCurrentUser) {
                history.push("/signin");
              }
              return null;
            });
          }
          // If there’s no error refreshing the token return an axios instance with the error config
          return axios(err.config);
        }
        // In case the error wasn’t 401 reject the Promise with the error
        return Promise.reject(err);
      }
    );
  }, [history]);
  ```

The handleMount function should use the axiosRes.get() instead of axios.get.

Nothing is using the request yet, but that will come.

## Update navBar for loggedIn/loggedOut

The [source code for these changes](https://github.com/mr-fibonacci/moments/tree/747d9d2350c57c995b52b4ff48a2e2b40cbf74b6).

Add the remaining icons to our NavBar for logged in users.  Here is what the add post icon looks like:

```js
const NavBar = () => {
  ...

  const addPostIcon = (
    <NavLink
      className={styles.NavLink}
      activeClassName={styles.Active}
      to="/posts/create"
    >
      <i className="far fa-plus-square"></i>Add post
    </NavLink>
  );
  ...
    return (
    <Navbar className={styles.NavBar} expand="md" fixed="top">
      <Container>
        <NavLink to="/"> ... </NavLink>
        {currentUser && addPostIcon}
  ```

The logged in user will have an avatar image, which is done with a specific component.


```css
.Avatar {
  border-radius: 50%;
  margin: 0px 8px 0px 8px;
  object-fit: cover;
}
```

The object-fitcover:  will make sure the profile image fits its container no matter what the image dimensions are.

```js
import React from "react";
import styles from "../styles/Avatar.module.css";

const Avatar = ({ src, height = 45, text }) => {
  return (
    <span>
      <img
        className={styles.Avatar}
        src={src}
        height={height}
        width={height}
        alt="avatar"
      />
      {text}
    </span>
  );
};

export default Avatar;
```

This used the ‘rafce’ code snippet from the ES7Snippets plugin.

The Avatar component takes some props which is descructures and provides a default height.

Check out [destructuring-assignment default values](9https://zaiste.net/posts/javascript-destructuring-assignment-default-values/) for more about that.

As a note, that can also be done like this:

```js
const Avatar = (props) => {
  const { src, height = 45, text } = props;
  ```

## NavBar burger toggle with a custom hook

The burger menu has an issue that when we click open we have to close it manually.

The changes for this work are [here](https://github.com/mr-fibonacci/moments/tree/35369a14e40c6132b7ad033aac90faa8143a7c75).

This change will references the DOM using the useRef hook.

Bootstrap Nabar has an expanded property which we tie to a state variable.  This get toggled in the ```onClick={() => setExpanded(!expanded)}``` function

```js
const ref = useRef(null);
useEffect(() => {
  const handleClickOutside = (event) => {
    if (ref.current && !ref.current.contains(event.target)) {
      setExpanded(false);
    }
};
```

Used on the tag like this:

```js
        <Navbar.Toggle
          ref={ref}
```

The code is first shown in-situ in the NavBar.  After it works it is refactored by putting the whole thing in a custom hook.  There are good reasons for this.

- It makes the code cleaner with all the logic for our toggle functionality in one place.
- The above follows the Single Responsibility Principal (SRP or the SOLID principals of development) which also allows the navbar to only be responsible for what it does.
- It can be reusable any time we want to toggle any element.

The custom hook looks like this:

```js
import useClickOutsideToggle from "../hooks/useClickOutsideToggle";

const NavBar = () => {
  const currentUser = useCurrentUser();
  const setCurrentUser = useSetCurrentUser();

  const { expanded, setExpanded, ref } = useClickOutsideToggle();
  ...
    return (
    <Navbar
      expanded={expanded}
      className={styles.NavBar}
      expand="md"
      fixed="top"
    >
      <Container>
        <NavLink to="/"> ... </NavLink>
        {currentUser && addPostIcon}
        <Navbar.Toggle
          ref={ref}
          onClick={() => setExpanded(!expanded)}
          aria-controls="basic-navbar-nav"
        />
        ...
```

The ref prop references the DOM element and detects if the mouse clicks outside of it.

It relies on a useEffect hook.

```js
  useEffect(() => {
    const handleClickOutside = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        setExpanded(false);
      }
    };
    document.addEventListener("mouseup", handleClickOutside);
    return () => {
      document.removeEventListener("mouseup", handleClickOutside);
    };
  }, [ref]);
```

The handleClickOutside function has an event argument.

If the element is null which is the initial value.

If the user has clicked outside the referenced button call setExpanded false to close the menu.

The mouseup event listener sets the handleClickOutside as its callback.  This means the hook will run every time the user clicks somewhere (technically after the click).

The cleanup function removes the listener.  Even though the navbar won’t be unmounted, it’s a best practice to remove event  
listeners in case it’s used on an element that could unmount.

The ref is put in the useEffect’s dependency array so this effect runs when it changes.

## Original readme

Welcome,

This is the Code Institute student template for React apps on the Codeanywhere IDE. We have preinstalled all of the tools you need to get started. It's perfectly ok to use this template as the basis for your project submissions.  
DO NOT use this template if you are using the Gitpod IDE. Use the following command instead:  
`npx create-react-app . --template git+https://github.com/Code-Institute-Org/cra-template-moments.git --use-npm`

You can safely delete this README.md file, or change it for your own project. Please do read it at least once, though! It contains some important information about Codeanywhere and the extensions we use. Some of this information has been updated since the video content was created. The last update to this file was: **31st August, 2023**

## Codeanywhere Reminders

In Codeanywhere you have superuser security privileges by default. Therefore you do not need to use the `sudo` (superuser do) command in the bash terminal in any of the lessons.

To log into the Heroku toolbelt CLI:

1. Log in to your Heroku account and go to *Account Settings* in the menu under your avatar.
2. Scroll down to the *API Key* and click *Reveal*
3. Copy the key
4. In Codeanywhere, from the terminal, run `heroku_config`
5. Paste in your API key when asked

You can now use the `heroku` CLI program - try running `heroku apps` to confirm it works. This API key is unique and private to you so do not share it. If you accidentally make it public then you can create a new one with *Regenerate API Key*.

---

Happy coding!

# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm install`

Installs the required npm packages.

### `npm start`

Runs the app in the development mode.\
Open port 3000 to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)
