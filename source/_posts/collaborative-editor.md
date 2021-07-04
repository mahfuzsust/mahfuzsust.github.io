---
title: Building a real-time collaborative editor
date: 2021-07-04 13:57:02
tags:
  - socket
  - node.js
  - ckeditor
categories:
  - System design
---
![alt text](https://miro.medium.com/max/2000/1*msFlUe_uZknWisLa8Ef7Rg.jpeg "Editor")

A text editor can be a real game changer when it's equipped with the functionality of collaborative editing. There are so many tools out there providing such features with great excellence such as Google Doc, Microsoft word etc.

I planned to design and develop such a system which can handle basic text editing and can scale on need. So I tried developing a POC(Proof of concept) which can handle real-time editing and went through some ideas to make this solution scalable.

Obviously, there are scopes for improvement and please share your ideas which will give better insights.

## Requirement
Let's define some project requirement to start on
1. Basic editing
2. Realtime collaboration
3. Basic operational transform
4. List of joined users
5. Add / Remove users based on event

## Tools
1. CKEditor 4
2. Plain & Simple Javascript
3. Node.js & Express
4. Socket.io
5. For operational trasform I've used the CKEditor 5 diff library to adapt changes to the editor.

## Initial setup
First we need a Node.js server which will host our application server and respond the front end in root endpoint.

{% codeblock %}
// package.json
{
  "name": "<NAME>",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@socket.io/redis-adapter": "^7.0.0",
    "express": "^4.17.1",
    "http": "^0.0.1-security",
    "redis": "^3.1.2",
    "socket.io": "^4.1.2",
    "uuid": "^8.3.2"
  },
  "devDependencies": {
    "nodemon": "^2.0.9"
  }
}
{% endcodeblock %}

I've added a redirection in the root endpoint to generate document id and load the html. If we store the document user specific then this step is completely unnecessary.

{% codeblock lang:javascript %}
//server.js

const express = require('express');
const path = require('path');
const { v4: uuidv4 } = require('uuid');
const app = express();
const port = process.env.PORT || 3000;

app.use('/public', express.static('public'));

app.get('/:id', (req, res) => {
    const fileDirectory = path.resolve(__dirname);
    res.sendFile('index.html', { root: fileDirectory }, (err) => {
        if (err) {
            console.error(err);
            throw (err);
        }
        res.end();
    });
});
app.get('/', (req, res) => {
    res.redirect(307, '/' + uuidv4());
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})

{% endcodeblock %}

Adding CKEditor in our front end application along with bootstrap for simplicity.

{% codeblock lang:html %}
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">

<head>
    <title>Collaborative Editor</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM"
        crossorigin="anonymous"></script>

    <script src="https://cdn.ckeditor.com/4.16.1/standard/ckeditor.js"></script>
    <script src="public/writer.js"></script>
</head>

<body class="container">
    <div class="row" style="padding-top: 20px;">
        <div class="col-7">
            <h1>Collaborative Editor</h1>
        </div>
    </div>

    <div id="editor-block" class="row" style="display: none;">
        <div class="form-group">
            <textarea id="textarea" class="form-control" rows="3"></textarea>
            <script>
                CKEDITOR.replace('textarea');
            </script>
        </div>
    </div>
</body>

</html>
{% endcodeblock %}

We will get the document ID from the path

{% codeblock lang:javascript %}
// writer.js
const documentId = new URL(window.location.href).pathname.substring(1);

const editor = CKEDITOR.instances.textarea;

// Get data
editor.getData();

// Set data
editor.setData("Hello world");
{% endcodeblock %}

## Collaborative editing
To add collaboration in our editor, we have different way to implement.

### Asynchronous collaborative editing
In this approach, the editor synchronize the content based on user event. For an example, after the user manually save the document or trigger an event, the content delivered to different clients associated with the content.
To publish local content, manual action is involved.

### Real-time editing
The content of the editor synchronize with other clients in real-time.

We will be moving forward with the real-time sync approach.

## Socket

In this phase, we are going to add socket implementation in our project.

Our server will listen to socket ports.

{% codeblock lang:javascript %}
// server.js
...
const httpServer = require("http").createServer(app);
const io = require("socket.io")(httpServer);
...
io.on("connection", socket => {
    console.log('socket connected..', socket.id);
});

httpServer.listen(port);

{% endcodeblock %}

## Real-time user management (Optional)
We will show the connected users and update the list real-time.

{% codeblock lang:javascript %}
// server.js
...
let roomMembers = {};
let socketRoom = {};

io.on("connection", socket => {
    console.log('socket connected..', socket.id);

    socket.on('register', function (data) {
        const room = data.documentId;
        socket.join(room);

        socketRoom[socket.id] = room;
        if (roomMembers[room]) {
            roomMembers[room].push({ id: socket.id, name: data.handle });
        } else {
            roomMembers[room] = [{ id: socket.id, name: data.handle }];
        }

        // Let all the clients know about new user to the document
        socket.to(room).emit('register', { id: socket.id, name: data.handle });
        
        // Post all the user list registered to this document
        io.in(room).emit('members', roomMembers[room]);
    });
    socket.on('disconnect', function (data) {
        console.log("Disconnected")
        let room = socketRoom[socket.id];
        if (room) {
            roomMembers[room] = roomMembers[room].filter(function (item) {
                return item.id !== socket.id
            });
            if (roomMembers[room].length == 0) {
                delete roomMembers[room];
            }
            delete socketRoom[socket.id];
            socket.to(room).emit('user_left', { id: socket.id });
        }
    });
});
...
{% endcodeblock %}

Add a list node to show all the connected users.

{% codeblock lang:html %}
...
<div class="row">
    <div class="col-5">
        <input type="text" class="form-control" id="handle" placeholder="User" />
        <button class="btn btn-outline-success" type="button" id="register">Register</button>
    </div>
</div>
...
<div class="row">
    <div class="col-5">
        <div class="card">
            <div class="card-header">
                Editors
            </div>
            <ul class="list-group list-group-flush" id="editors">
            </ul>
        </div>
    </div>
</div>
...
{% endcodeblock %}

Dynamically add the users to DOM

{% codeblock lang:javascript %}
// writer.js
...
let socket;
const handle = document.getElementById('handle');
const register = document.getElementById('register');

function addEditor(writer) {
    var ul = document.getElementById("editors");
    var li = document.createElement("li");
    li.appendChild(document.createTextNode(writer.name));
    li.className = "list-group-item";
    li.id = writer.id;
    ul.appendChild(li);
}
function removeEditor(id) {
    var elem = document.getElementById(id);
    return elem.parentNode.removeChild(elem);
}

function registerUserListener() {
    handle.style.display = 'none';
    register.style.display = 'none';

    const editorBlock = document.getElementById('editor-block');
    editorBlock.style.display = 'block';

    socket = io();
    socket.emit('register', {
        handle: handle.value,
        documentId: documentId
    });
    socket.on('register', (data) => {
        addEditor(data);
    });
    socket.on('user_left', (data) => {
        removeEditor(data.id);
    });
    socket.on('members', (members) => {
        members.forEach(member => {
            addEditor(member);
        });
        socket.off('members');
    });
}
register.addEventListener('click', registerUserListener);
...
{% endcodeblock %}

## Collaborative editing

In terms of collaborative editing we can face different type of issues. The most critical of them is conflict resolution. When multiple user changing same text at a time can cause conflict which has to resolve in terms of real time collaboration. There are many algorithms to solve this issue in certain extent, but they have different pros and cons in different situations. The most popular of them is operational transform.

## Operational transform
According to [wikipedia](https://en.wikipedia.org/wiki/Operational_transformation)
> Operational transformation (OT) is a technology for supporting a range of collaboration functionalities in advanced collaborative software systems. OT was originally invented for consistency maintenance and concurrency control in collaborative editing of plain text documents. 

Implementing OT is something complex which takes a lot of time to implement and be good at. There is a famous quote by Joseph Gentle, one of the early people implemented OT

> Unfortunately, implementing OT sucks. There’s a million algorithms with different tradeoffs, mostly trapped in academic papers. The algorithms are really hard and time consuming to implement correctly. […] Wave took 2 years to write and if we rewrote it today, it would take almost as long to write a second time. 

Let's look at a example of operational transform. Let's say we have a text ```CA``` and two different user is doing operation 

String --> CA
User 1 --> CAT (operation O1)
USER 2 --> HAT (opearion O2)

O1 = [insert T at position 2]
O2 = [insert T at position 2, delete C at position 0, insert H at position 0]

For User 1, local operation is O1 and incoming change operation is O2
For User 2, local operation is O2 and incoming change operation is O1

To add transformation, we have to apply both these changes to the string. The transformation will be,

    For User 1, String --> O2 --> O1  
        String --> CA 
        O2     --> HAT
        O1     --> HAT (Result)
    For User 2, String --> O1 --> O2 
        String --> CA
        O1     --> CAT
        O2     --> HAT (Result)

The states are synchronized for both the user.

This way we'll get the same result at the end of the transformation and for every changes we made we will apply that with the last sync value.

For this project, I have used CKEditor 5 [```diffToChanges```](https://ckeditor.com/docs/ckeditor5/latest/api/module_utils_difftochanges.html) util to check the changes(operations) from sync state and then apply in the editor.

> The most difficult part of OT is not the code, but the difficulty in proving that your system is correct. Therefore, maintaining OT code is difficult. Either you need to prove your code correct repeatedly (and historically people make mistakes), or you need a powerful testing infrastructure for concurrent/distributed systems (which is also hard to write)[1]


## OT implementation

{% codeblock lang:javascript %}
// server.js
...
socket.on('content_change', (data) => {
    const room = data.documentId;
    socket.to(room).emit('content_change', data.changes);
});
...
{% endcodeblock %}

Now, We need to apply transformation to the editor content. In the code, I publish changes to the socket after the user stopped typing. In this case, the user will compare the changes with the sync state and then apply the changes to the state and publish the changes to the socket.

While receiving the change, first it'll check with the sync state and then apply it there. After that, it'll apply the local changes to the state and apply and set value of the editor. For the local additional changes it'll then publish again to the socket.

{% codeblock lang:javascript %}
// writer.js
...
// Sync state store
let syncValue = Array();
let keypressed = false;

function getChanges(input, output) {
    return diffToChanges(diff(input, output), output);
}

function applyChanges(input, changes) {
    changes.forEach(change => {
        if (change.type == 'insert') {
            input.splice(change.index, 0, ...change.values);
        } else if (change.type == 'delete') {
            input.splice(change.index, change.howMany);
        }
    });
}
function applyLocalChanges() {
    if (keypressed && editor.checkDirty()) {
        let currentData = editor.getData();
        let input = Array.from(syncValue);
        let output = Array.from(currentData);
        let changes = getChanges(input, output);
        applyChanges(input, changes);
        if (output.join('') == input.join('')) {
            socket.emit('content_change', {
                documentId: documentId,
                changes: changes
            });
            editor.resetDirty();
            syncValue = input;
        }
        keypressed = false;
    }
}

socket.on('content_change', (incomingChanges) => {
    let input = Array.from(syncValue);
    applyChanges(input, incomingChanges);
    syncValue = input;
    applyLocalChanges();

    let ranges = editor.getSelection().getRanges();
    editor.setData(syncValue.join(''));
    editor.getSelection().selectRanges(ranges);
    editor.resetDirty();
});

var timeout = setTimeout(null, 0);
editor.on('key', () => {
    clearTimeout(timeout);
    keypressed = true;
    timeout = setTimeout(applyLocalChanges, 1000);
});
...
{% endcodeblock %}

## Containerization
{% codeblock %}
FROM node:16-alpine
ENV NODE_ENV=production

WORKDIR /app

COPY ["package.json", "package-lock.json*", "./"]

RUN npm install --production

COPY . .

EXPOSE 3000

CMD [ "node", "server.js" ]
{% endcodeblock %}

In the next part, I'll share the ideas and implementation to scale this application.

Full source code is hosted in [Github](https://github.com/mahfuzsust/collab) and a quick [demo](https://collab-edit-1.herokuapp.com)

## Reference
1. [https://digitalfreepen.com/2018/01/04/operational-transform-hard.html](https://digitalfreepen.com/2018/01/04/operational-transform-hard.html)
[1]: https://digitalfreepen.com/2018/01/04/operational-transform-hard.html