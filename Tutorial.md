# How to Submit Multipart/Form-Data in JavaScript

## Introduction

When a beginner developer first works with APIs, they tend to focus only on retrieving data, or sending text
to the server. As they become more comfortable and make more complex requests, they may come across one that requires a particularly annoying content type: multipart/form-data. Multipart/form-data is a type of request content that accepts both file and text data, hence "multipart." While it sounds simple enough, it's not uncommon for developers to receive unexpected errors. In this tutorial, I'm going to show how to properly send multipart/form-data content using JavaScript, with the ExpressJS framework handling the backend. 

## Prerequisites

This tutorial assumes that you already have an existing project using React and Express. In order to properly handle images sent to the server, you will need an image processing library. The standard library for this purpose is Multer, which is what will be used for this tutorial. If you haven't done so, download it to your Express folder. You can do so with the following command:

	npm install multer

## Setting up Multer

In order for the server to process image data, Multer will need to be set up. Before doing so, you will need a location to place these images. If your Express project was created with Express Generator, then you may use the public folder's default images subfolder to store data; alternatively, you can create your own folder.

Once you've decided on a location for your images, create a new file in your Express folder and import Multer at the top. This will be your configuration file. Using Multer's diskStorage method, create your storage variable to set up the destinations and file names. Here is an example from one of my own projects:

	const storage = multer.diskStorage({

    destination: (req, file, cb) => {
            if(file.fieldname === 'profilepic') {
                cb(null, '../express-bookmark/public/profilepics');
            }
            else if(file.fieldname === 'chatimage') {
                cb(null, '../express-bookmark/public/chatimages');
            }
            else if(file.fieldname === 'groupimage') {
                cb(null, '../express-bookmark/public/groupimages');
            }
            else if(file.fieldname === 'commentimage') {
                cb(null, '../express-bookmark/public/commentimages');
            }
            else {
                cb(null, '../express-bookmark/public/postimages');
            }
        },
        filename: (req, file, cb) => {
            cb(null, `${file.fieldname}-${Date.now()}`);
        }
    });

Your project, of course, may not need as many folders. Be mindful of the fieldname property, as it is a crucial part of the uploading process. More on that later.

Once storage has been set up, you'll need to create and export an upload function to place in the endpoint(s) that require file uploads. Luckily, all you need to do is assign Multer to a variable and set your storage variable as a parameter of Multer, like so:

	module.exports.upload = multer({storage: storage});

Now that Multer's been configured, you need to call the upload function in the endpoint that takes file uploads. Go to the file that contains the endpoint, and import the upload function. If you're uploading a single file to the server, then you must use upload's single method, which takes a fieldname as a parameter. It may look something like this:

	router.patch('/user/picture', upload.single('profilepic'), userController.edit_profile_picture);

Alternatively, if you want the option to submit multiple files at once, you may use the upload.array() method instead. This takes the fieldname as the first parameter, and the maximum number of images allowed for the second.
Regardless of whether you keep your middleware functions in separate files — as I do — or write them with your routes, the upload call must be placed immediately after your endpoint. If you place the upload function anywhere else, then the function will not fire, and Multer will not process the file.

### Something to Consider:

Before moving on to the frontend, it's a good idea to verify how you're going to store the file data in your database, as you may encounter some issues depending on your approach. For example, if you save an image file's binary data, you could risk overloading your computer's memory when the images are rendered due to the size of the data in question. This could cause freezes, or even crashes. It's recommended to send your images to a third-party host, like Cloudinary or Azure, which will provide a URL that you can then save to your database.

## Sending a File to the Server:
Now that the backend has been set up to process file uploads, you can write your fetch call to send data. By now, you're probably familiar with the basics of sending a POST, PUT, or PATCH request — manually setting the Content-Type header to "application/json" and using the JSON.stringify() method on your request's body — but formatting multipart/form-data content is a little different. Unlike application/json, which takes a JSON object converted to a string, multipart/form-data takes key-value pairs directly from, as you may have guessed, forms. JavaScript has a built-in constructor that can create an object that will store the required data, aptly named FormData. Before you make your fetch call, you will need this object to store your data. Creating it is very simple:

	const form = new FormData();

Once the object's been created, all you need to do is store the value of your data using FormData's append method. FormData.append() takes a key as its first parameter, and the value of your data as the second.
	form.append('profilepic', file);

There are two things worth noting before continuing. First, when uploading a file to the server, the key associated with the file must match the value of the given fieldname in your endpoint's upload function call. If the key and the fieldname do not match, then you will receive an error — typically a 500 HTTP status code — and Multer will not process your file. The second thing to note is that, if you have additional data, then you must make separate calls to FormData.append() until all relevant data has been stored.
	
If you're submitting multiple files to the server, you must append each file individually. If an array of files is sent to the server, then JavaScript will convert it to a string, and the array will instead be sent as [object Object]. This can easily be avoided by looping through the array of files on your frontend, and appending each file to the FormData object with identical keys.

	files.forEach(file => {
        form.append('chatimage', file)}
    );

Now that you've appended your data, you can write your fetch call. Much like with requests accepting application/json data, you will need to specify the endpoint's method and credentials (if authorization is required), but one major difference is that you do not need to specify the request's Content-Type. This is because JavaScript automatically reads file data as multipart/form-data content. One other difference is that, as implied above, you do not need to convert the FormData object to a string — you may simply set the value of the body property to the object itself. Your fetch call may look something like this:

	fetch('http://example.bookmark.com/api/user/picture', {
        method: 'PATCH',
        credentials: 'include',
        body: form
    })
    .then(res => {
        res.json()
    })
    .then(data => {
        console.log(data)
    })

When the request has been sent, be sure to check your assigned image folder for the uploaded file. If the file has been saved, then Multer has successfully processed it.

## Conclusion:

While multipart/form-data can be a bit difficult at first, it's a very common and valuable Content-Type that becomes much easier with Multer. When sending your files, be sure to verify that your fieldnames match and avoid specifying the Content-Type to prevent errors. Also, be mindful of whether you're sending one file or multiple to your server, and adjust your code accordingly to avoid sending an [object Object].
