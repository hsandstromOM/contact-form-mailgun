# contact-form-mailgun
![Travis Cl](https://travis-ci.org/hsandstromOM/contact-form-mailgun.svg?branch=master)

<div class="post">

# Enabling Contact Forms with Node.js, NodeMailer, and MailGun

<span class="post-date">08 Oct 2016</span>

Node.js makes it easy to add a working contact form to your website.  

## Starting with a static website

Let’s assume our application is called A03 (short for assignment 03).

Start a responsive website with [Initializr](http://www.initializr.com/).

Select the desired options for a new BootStrap site and download the code. This creates a website with the index.html in a root folder, with subfolders css, fonts, img, and js.

Create a contact.html page with a contact form, the input forms you would like, and a submit button.

<div class="highlighter-rouge">

         <form action="/contact" method="post" id=contactform class="well form-horizontal">

</div>

Finish creating and customizing your site. Your contact page may look something like the following.

[![](http://denisecase.github.io/project/44563-A03/assets/img/2016-10-08_1056.png)](http://denisecase.github.io/project/44563-A03/assets/img/2016-10-08_1056.png)

See a [demo](http://denisecase.github.io/project/44563-A03/assets/index.html) of a sample static site.

## Set up an emailing account on MailGun

1.  Go to https://mailgun.com.
2.  Set up your account by providing your information.
3.  Confirm your (real) email address.
4.  Login to your Mailgun account and activate it with your cell phone number.
5.  Click domains. Click the settings (gear) icon next to your sandbox and select ‘Domain settings’.
6.  Add the email to which your comments should be sent. Confirm the test message sent to this address.
7.  Note the domain you want to use for sending emails (I used the sandbox as this is just for testing).
8.  Note the API key provided.

Confirm your willingness to have emails automatically sent to you from Mailgun.

[![](http://denisecase.github.io/project/44563-A03/assets/img/2016-10-08_1110.png)](http://denisecase.github.io/project/44563-A03/assets/img/2016-10-08_1110.png)

## Adding server-side functionality with Node.js

We’ll create our Node.js server content in the root folder, so create a subfolder in your application called assets and move your website code to this folder.

In the root folder (A03), create a .gitignore file and a README.md file (used for all types of projects).

Add the following to the .gitignore.

<div class="highlighter-rouge">

    node_modules
    access.log
    config.json

</div>

In the root folder (A03), create a _package.json_, and an _app.js_ for the Node.js application.

You package.json should include the following dependencies (app.js is described later).

<div class="language-json highlighter-rouge">

    {
      "dependencies": {  
            "express": "latest",  
            "morgan": "latest",  
            "body-parser": "latest",  
            "nodemailer": "latest",  
            "nodemailer-mailgun-transport": "latest",  
            "nconf": "latest"  
        }  
    }  

</div>

Add one more file to the root folder, _config.json_. This will hold confidential connection information that we don’t want to check into the cloud repository. Get your Mailgun domain and api-key noted above and add these two items to your config.json file in the following format.

<div class="language-json highlighter-rouge">

    {
        "auth": {
            "api_key": "key-123456abcdef",
            "domain": "sandbox1234abcd5678.mailgun.org"
        }
    }

</div>

In the app.js file, include the dependencies, create an express app and use it to configure the server. Provide access to your static client-side files, include the body-parser to help read the information submitted on the form, and configure an HTTP request logger (Morgan).

<div class="language-javascript highlighter-rouge">

    var path = require("path");
    var express = require("express");
    var fs = require('fs')
    var logger = require("morgan");  
    var nodemailer = require('nodemailer');
    var mg = require('nodemailer-mailgun-transport');
    var bodyParser = require('body-parser');
    var nconf = require('nconf');
    var auth =  require('./config.json');
    const port = process.env.PORT || 8081;

    // make a request app and create the server
    var app = express();
    var server = require('http').createServer(app);

    // include client-side assets and use the bodyParser
    app.use(express.static(__dirname + '/assets'));
    app.use(bodyParser.urlencoded({ extended: false }));
    app.use(bodyParser.json());

    // log requests to stdout and also
    // log HTTP requests to a file in combined format
    var accessLogStream = fs.createWriteStream(__dirname + '/access.log', { flags: 'a' });
    app.use(logger('dev'));
    app.use(logger('combined', { stream: accessLogStream }));

</div>

Create nice URLs for your pages and serve up your html:

<div class="language-javascript highlighter-rouge">

    // http GET default page at /
    app.get("/", function (request, response) {
      response.sendFile(path.join(__dirname + '/assets/index.html'));
    });

    // 404 for page not found requests
    app.get(function (request, response) {
      response.sendFile(path.join(__dirname + '/assets/404.html'));
    });

    // http GET /about
    app.get("/about", function (request, response) {
      response.sendFile(path.join(__dirname + '/assets/about.html'));
    });

    // http GET /contact
    app.get("/contact", function (req, res) {
      res.sendFile(path.join(__dirname + '/assets/contact.html'));
    });

</div>

Handle the POST call when the user submits their contact form.

<div class="language-javascript highlighter-rouge">

    // http POST /contact
    app.post("/contact", function (req, res) {
      var name = req.body.inputname;
      var email = req.body.inputemail;
      var company = req.body.inputcompany;
      var comment = req.body.inputcomment;
      var isError = false;

      if (company) {
        isError = true;
      }
      console.log('\nCONTACT FORM DATA: '+ name + ' '+email + ' '+ comment+'\n');

      // create transporter object capable of sending email using the default SMTP transport
      var transporter = nodemailer.createTransport(mg(auth));

      // setup e-mail data with unicode symbols
      var mailOptions = {
        from: '"Denise Case" <denisecase@gmail.com>', // sender address
        to: 'dcase@nwmissouri.edu, denisecase@gmail.com', // list of receivers
        subject: 'Message from Website Contact page', // Subject line
        text: comment,
        err: isError

      };
      // send mail with defined transport object
      transporter.sendMail(mailOptions, function (error, info) {
        if (error) {
          console.log('\nERROR: ' + error+'\n');
          //   res.json({ yo: 'error' });
        } else {
             console.log('\nRESPONSE SENT: ' + info.response+'\n');
          //   res.json({ yo: info.response });
        }
      });
    });

</div>

Finally, start up the server and listen on the specified port.

<div class="language-javascript highlighter-rouge">

     // Listen for an application request on designated port
    server.listen(port, function () {
      console.log('Web app started and listening on http://localhost:' + port);
    });

</div>

## Run your website locally

1.  Open a command window in your c:\44563\a03 folder or from VS Code menu, chose View / Integrated Terminal
2.  Install nodemon globally with npm install -g nodemon
3.  Install the dependencies listed in package.json with npm install.
4.  Run nodemon to start the server. (Hit CTRL-C to stop.)

<div class="highlighter-rouge">

    > npm install -g nodemon
    > npm install
    > nodemon

</div>

Point your browser to `http://localhost:8081`.

## Video tutorial

For more information, see the video [How to send Email from Node.js Application with MailGun, SendGrid, MailChimp Serivces](https://www.youtube.com/watch?v=9RNQNwHCvSU).

</div>
