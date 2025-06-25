# React frontend for Chatbot

## Table of contents

1. [Introduction](#Introduction)
2. [How to run and develop it](#How-to-run-and-develop-it)
   - [Setting up your environment ](#Setting-up-your-environment)
   - [Debugging](#Debugging)
3. [File descriptions](#File-descriptions)
4. [About .jsx files](#About-jsx-files)

## Introduction

Here you can find more information on the frontend of the project "Chatbot for assessing system security with OpenAI GPT-3". 
The frontend was implemented using React https://react.dev/. 
UI elements were created with the help of Bootstrap https://getbootstrap.com/.
Vite was used to help set up the project and for the ease of development https://vitejs.dev/. 

## How to run and develop it

### Setting up your environment

1. Start by cloning the repository

```
git clone https://github.com/Zippo00/Chatbot-for-Assessing-System-Security.git
cd Chatbot-For-Assessing-System-Security/app/frontend/react-chat
```

2. Install npm & requirements

```sh
sudo apt install npm
npm install bootstrap@3
```

3. Configure the correct IP address for your API requests on `Chat.jsx` around line 48, in my case I am going to use my private IP. 

```
# Chat.jsx
let aiResponse = await fetch("http://192.168.0.179:5000/", {
<SNIP>
}
```

4. Install NGINX:
```sh
sudo apt install nginx
```

5. Configure UFW - Allow the port you use for connecting to the Virtual Machine (22 for SSH):
```sh
sudo ufw allow 22
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

6. Check that NGINX is working by heading to your public IP address:
```
http://<YOUR IP ADDRESS>
```

You should be greeted with NGINX Welcome page.

7. Create a new server block for NGINX:
```sh
sudo nano /etc/nginx/sites-available/alice_frontend
```
Copy the following to the `alice_frontend` file to handle `HTTP` traffic (for `HTTPS`, add another server block with corresponding port to listen to):
```sh
server {
    listen 80;
    root /home/ubuntu/public_chatbot_frontend/react-chat; #<PATH TO react-chat DIRECTORY>
    index index.html index.htm index.nginx-debian.html;
    server_name <YOUR IP ADDRESS>;
    location / {
        proxy_pass <ADDRESS:PORT WHERE REACT-APP IS RUNNING>; #run 'npm run dev' in react-chat dir for URL
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
    error_log  /var/log/nginx/vue-app-error.log;
    access_log /var/log/nginx/vue-app-access.log;
}
```

8. Link the newly created file to `sites-enabled`
```sh
sudo ln -s /etc/nginx/sites-available/alice_frontend /etc/nginx/sites-enabled
```

9. Check NGINX for any syntax errors:
```sh
sudo nginx -t
```

You should see:
```sh
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

10. Restart the NGINX service:
```sh
sudo systemctl restart nginx
```

11. Start the frontend, and check the output, it will contain the URL of frontend (in my case it is http://192.168.0.179:5173/ - yours might be different)

```
npm run dev

VITE v4.3.5  ready in 559 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: http://192.168.0.179:5173/
  ➜  press h to show help
```

12. If you have not cloned the backend repository yet, do it now. https://github.com/Zippo00/public_chatbot_backend

13. Make a text file called `openaiapikey.txt` in `public_chatbot_backend` directory, where `app.py` is, and put your OpenAI API key in there

14. Go to backend directory on another terminal & run the flask API

```
cd public_chatbot_backend
flask run --host=0.0.0.0
```

15. Browse to the given URL (in my case it is http://192.168.0.179:5173/). Type a message to the input field and send it by pressing the enter key on your keyboard.

16. If everything is set up correctly, you should see the responses from the AI to the messages you sent in the frontend.

### Debugging

1. Check logs on flask backend
2. Check logs on frontend by pressing F12
3. If the Frontend is otherwise working properly, but the web elements are disorganized (Web page not looking like it's supposed to): Most likely an issue with `node_modules`.

## File descriptions

This part contains a brief description of the contents of the files. I have also included some comments in the code to hopefully make it a bit easier to understand.

The first step during the implementation of the frontend was to set up Vite. This process automatically creates all the necessary files needed for the base of a functioning frontend, such as `index.html`, `package.json` and `main.jsx`. In this implementation, these mainly remained unchanged and I will not cover their descriptions here. The following link contains documentation on how to set up Vite https://vitejs.dev/guide/#scaffolding-your-first-vite-project.

### `App.jsx`

Contains the “main function” of the frontend application. At this moment it only returns the Chat component.

### `Chat.jsx `

Contains most of the functionalities and UI elements of the frontend implementation. Most of the implementation's UI elements are created here, such as the chat window, input bar and side navigation bar. Also contains variables and functions that make sure the chat functions as it should. These include things such as: messages sent, received and displayed.

### `Chat.css`

Contains stylizing for `Chat.jsx`, namely for the side navigation bar and chat window.

### `ChatMessage.jsx`

Contains code for the chat message fields that show up in the chat window when messages are sent and received. It sets the sender name, message and user icons.

### `ChatMessage.css`

Contains stylizing for `ChatMessage.jsx`. Sets message’s background color and sender’s icon size.

## About .jsx files

If you not familiar at all with React JSX, this website might help you to learn the basics https://react.dev/learn.

In this repository you can notice a few .jsx files. Here's an example of the core structure of an example .jsx file.

```javascript
//import everything you may need, such as .css file and chakra UI
import "./Example.css";
import { Text, Input } from "@chakra-ui/react";

//Start of our Example function
function Example() {
  //Declaring functions, variables etc here
  const handleKeyDown = (event) => {
    if (event.key === "Enter") {
      console.log(message);
    }
  };

  //Inside return here we can construct the UI elements. In this example Chakra is used, but in the implementation found in this repository bootstrap was used instead.
  //about chakra: https://chakra-ui.com/
  //about bootstrap: https://getbootstrap.com/
  
  return (
    <div>
      <Text fontSize="6xl" color="White" ml={6}>
        Security ChatBot
      </Text>
      <Input type="text" onKeyDown={handleKeyDown} />
    </div>
  );
}
//export the Example function
export default Example;
```