# 🔐Basic Login System
---
## ⚠️ Requirements
- React Project
- API Auth endpoint
---

> We will use the [🔐Basic Authentication System]() example on this case.

### 1: Creating an AuthWrapper
The AuthWrapper will be like a box where we define all the needs for the login purposes.

Lets create a React component called **AuthWrapper.jsx**

```
import { createContext, useContext, useState, useEffect } from "react"
import { RenderMenu, RenderRoutes } from "../components/structure/RenderNavigations";


const AuthContext = createContext();
export const AuthData = () => useContext(AuthContext);


export const AuthWrapper = () => {
     const [user, setUser] = useState({ name: "", isAuthenticated: false });

     useEffect(() => {
          // Check if user cookie exists
          const userCookie = getCookie("user");
          if (userCookie) {
              const userData = JSON.parse(decodeURIComponent(userCookie));
              setUser(userData);
          }
      }, []);

     const login = (userName, password) => {
     return new Promise((resolve, reject) => {
          const myHeaders = new Headers();
          myHeaders.append("Content-Type", "application/json");
          myHeaders.append("Accept", "application/json");
          myHeaders.append("x-api-key", "API_KEY");

          const raw = JSON.stringify({
               "username": userName,
               "password": password
          });

          const requestOptions = {
               method: "POST",
               headers: myHeaders,
               body: raw,
               redirect: "follow"
          };

          fetch("http://localhost:8080/api/users/auth", requestOptions)
               .then((response) => response.json()) // Parse response as JSON
               .then((result) => {
                    if (result.status === "success") {
                         document.cookie = `user=${encodeURIComponent(JSON.stringify({ name: userName, isAuthenticated: true }))}; path=/`;
                         setUser({ name: userName, isAuthenticated: true });
                         resolve("success");
                    } else {
                         reject("Incorrect credentials");
                    }
               })
               .catch((error) => {
                    reject("Error occurred while logging in");
               });
     });
          
          
     }
     const logout = () => {
          document.cookie = "user=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
          setUser({ name: "", isAuthenticated: false });
     }

     const getCookie = (name) => {
          const cookies = document.cookie.split("; ");
          for (let i = 0; i < cookies.length; i++) {
              const cookie = cookies[i].split("=");
              if (cookie[0] === name) {
                  return cookie[1];
              }
          }
          return null;
      };


     return (
          
               <AuthContext.Provider value={{user, login, logout}}>
                    <>
                         <RenderMenu />
                         <RenderRoutes />
                    </>
                    
               </AuthContext.Provider>
          
     )

}
```

- We create a context.
- We create a user object were we indicate the username and if its authenticated or not.
- A login request to the API that stores the information into the user object.
- A cookie system that allows us to save the user session until the logut.

### 2: Define the navigation of the website
Once we have the AuthWrapper created we need to define a system to store al the routes and details:

**Navigations.jsx**
```
import Home from "../../views/Home.jsx"
import Login from "../../views/Login.jsx"
import Dashboard from "../../views/Dashboard.jsx"

export const nav = [
     { path:     "/",         name: "Home",        element: <Home />,       isMenu: true,     isPrivate: false  },
     { path:     "/login",    name: "Login",       element: <Login />,      isMenu: false,    isPrivate: false  },
     { path:     "/dashboard",  name: "Dashboard",     element: <Dashboard />,    isMenu: true,     isPrivate: true  },
]
```
Here we import the components or view we want to render on each route and then we declare if they are private or not.

### 3: Render navigations
We need to create a file that will allow us to render the routes and the menu.

```
import { Link, Route, Routes } from "react-router-dom";
import { AuthData } from "../../auth/AuthWrapper";
import { nav } from "./Navigations.jsx";

export const RenderRoutes = () => {
    const { user } = AuthData();
        
    return (
         <Routes>
         { nav.map((r, i) => {
              
              if (r.isPrivate && user.isAuthenticated) {
                   return <Route key={i} path={r.path} element={r.element}/>
              } else if (!r.isPrivate) {
                   return <Route key={i} path={r.path} element={r.element}/>
              } else return false
         })}
         
         </Routes>
    )
}

export const RenderMenu = () => {
   
    const { user, logout } = AuthData()

    const MenuItem = ({r}) => {
         return (
               <li className="mr-5 text-md text-dark-200">
                    <Link to={r.path}>{r.name}</Link>
               </li>      
         )
    }
    return (
          <header>
                <ul>
                    { nav.map((r, i) => {
                        
                        if (!r.isPrivate && r.isMenu) {
                            return (
                                <MenuItem key={i} r={r}/>
                            )
                        } else if (user.isAuthenticated && r.isMenu) {
                            return (
                                <MenuItem key={i} r={r}/>
                            )
                        } else return false
                        } )}

                        { user.isAuthenticated ?
                        <li>
                        <Link to={'/login'} onClick={logout}>Log out</Link>
                        </li>
                        :
                        <li><Link to={'login'}>Login</Link></li> 
                    }
                </ul>              
          </header>
    )
}

```

### 4: Modify the main.jsx
We need to change the root file to wrap the routes inside our AuthWrapper like this:

```
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom';
import './index.css'
import { AuthWrapper } from "./auth/AuthWrapper.jsx"

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <AuthWrapper />
    </BrowserRouter>    
  </React.StrictMode>,
)
```