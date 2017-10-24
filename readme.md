# YelpCamp

### Refactored by Ian Schoonover

## Image Upload Instructions
Note: Before you get started it may be a good idea to use git to checkout a new branch to add this feature on. If you don't know how to use git then use [this free coupon](https://www.udemy.com/intro-to-git/?couponCode=WDBCLASS) to take my git course.
You'll need to use `git checkout -b image-upload` to create a new branch, then once you're done adding the feature, if it works properly, go back to master and merge the branches with `git checkout master` and `git merge image-upload`

## Update node
This tutorial will be using newer syntax from the lastest versions of ECMAScript. I'll be linking to various YouTube tutorials throughout this document to better help you learn the syntax if you haven't seen or used it prior. 

In order to use the new syntax you'll need the latest version of node. Use one of the two instructions from below depending on your development environment.

**Cloud9:** From your terminal run `nvm use 8`. You should be good to go, but see [here](https://community.c9.io/t/how-to-update-node-js/1273) for more on updating node js with cloud9 if you run into trouble.

**Local Env:** [install nvm](https://github.com/creationix/nvm) then run `nvm install node` and `nvm use node`.

You can check your version number with `nvm ls`, version 8.0.0 or newer is needed.

## Sign up for Cloudinary
- Visit [cloudinary](https://cloudinary.com/users/register/free) and create a free account
- Once signed in a popup window will ask you "What is the primary development framework you use?". Click "Continue later" in the bottom left corner of the popup window
- Leave the dashboard open, we'll come back to it shortly

## Install dotenv
**If you already have dotenv installed and configured you can skip this step**

We need a way to include sensitive information, like our API key for cloudinary, without it being exposed to the public when we push our code to places like GitHub. For that we'll use dotenv.

- Open the terminal and in your project's root directory enter the following `npm install dotenv --save`
- Add the following code to the very top of your app.js file `require('dotenv').config()`
- Open the terminal and create a file named `.env` in the root directory of your project with `touch .env`
- Open the .env file in your code editor and add the following:

```
CLOUDINARY_API_KEY=<api-key-here>
CLOUDINARY_API_SECRET=<api-secret-here>
```
- Replace <api-key-here> and <api-secret-here> with the information from your Cloudinary dashboard under *Account Details*
	- Note: You'll need to click the Reveal link to get the API Secret.
- Save the .env file

## Configure multer and cloudinary
- Open the terminal and run the following from the root directory of your application: `npm i -S multer && npm i -S cloudinary`
- Create a new directory named `uploads` inside of `/public`, so the structure looks like this: `/public/uploads`
- Create a file inside your `/middleware` directory named `cloudinary.js` and enter the following code into it then save the file:

```JS
//************* Image Upload Configuration *************\\
const multer = require('multer');
const storage = multer.diskStorage({
  destination: function(req, file, callback) {
    callback(null, './public/uploads');
  },
  filename: function(req, file, callback) {
    callback(null, Date.now() + file.originalname);
  }
});
const imageFilter = function (req, file, cb) {
    // accept image files only
    if (!file.originalname.match(/\.(jpg|jpeg|png|gif)$/i)) {
        return cb(new Error('Only image files are allowed!'), false);
    }
    cb(null, true);
};
const upload = multer({ storage : storage, fileFilter: imageFilter})

const cloudinary = require('cloudinary');
cloudinary.config({ 
  cloud_name: '<your-cloudname-here>', 
  api_key: process.env.CLOUDINARY_API_KEY, 
  api_secret: process.env.CLOUDINARY_API_SECRET
});
//************* END Image Upload Config *************\\

// export upload and cloudinary
module.exports = {
	cloudinary: cloudinary,
	upload: upload
}
```

- Be sure to replace `<your-cloudname-here>` with your **Cloud name** from the Cloudinary Dashboard under **Account Details**
- Note: This section of code uses let and const instead of var, checkout [this tutorial](https://www.youtube.com/watch?v=sjyJBL5fkp8) to learn about let and const as you will be using them throughout this tutorial.



**Explanation:** 
We've just required multer, the package that allows us to add a working file upload button/input to our form. This will allow us to send the file to our server when the form is submitted.

We configured our storage setup so our app knows where to save the image files first before uploading them to Cloudinary.

We configured an image filter with [regular expressions](https://regexr.com/) that only allows files with valid image extensions to be uploaded from the form. e.g., .pdf files won't work, but .jpg's will.

We required cloudinary and configured it to connect to our online account. 

Finally, we exported the now configured cloudinary and upload variables so they can be used in our routes file.

## Adding cloudinary to our routes

- Open `/routes/campgrounds.js` and add this line of code to the top of the file where your other npm packages are being required and save the file: `const { cloudinary, upload } = require('../middleware/cloudinary');`
	- Note: if this syntax is new to you then take a minute after this tutorial to learn about [destructuring assignment](https://www.youtube.com/watch?v=PB_d3uBkQPs)
- Replace `var geocoder = require('geocoder');` in /routes/campgrounds.js with: 

```JS
// require and configure node-geocoder
const NodeGeocoder = require('node-geocoder');
const options = {
  provider: 'google'
};
const geocoder = NodeGeocoder(options);
```

- From the project's root directory in the terminal run `npm i -S node-geocoder && npm uninstall geocoder`
	- Note: We're replacing the geocoder package that I used in my Google maps tutorial with another one that allows us to use async + await, if you don't know what async + await is then take a moment to watch [this tutorial](https://www.youtube.com/watch?v=568g8hxJJp4)
- If you've been following along with all of my tutorials then your /campgrounds POST route code will look something like this:

```JS
//CREATE - add new campground to DB
router.post("/", isLoggedIn, isSafe, function(req, res){
  // get data from form and add to campgrounds array
  var name = req.body.name;
  var image = req.body.image;
  var desc = req.body.description;
  var author = {
      id: req.user._id,
      username: req.user.username
  }
  var cost = req.body.cost;
  geocoder.geocode(req.body.location, function (err, data) {
    var lat = data.results[0].geometry.location.lat;
    var lng = data.results[0].geometry.location.lng;
    var location = data.results[0].formatted_address;
    var newCampground = {name: name, image: image, description: desc, cost: cost, author:author, location: location, lat: lat, lng: lng};
    // Create a new campground and save to DB
    Campground.create(newCampground, function(err, newlyCreated){
        if(err){
            console.log(err);
        } else {
            //redirect back to campgrounds page
            console.log(newlyCreated);
            res.redirect("/campgrounds");
        }
    });
  });
});
```

- This assumes you've completed the google maps tutorial, but if you haven't then see [here](http://slides.com/nax3t/yelpcamp-refactor-google-maps#/)

- We're going to replace the code in this route with some newer syntax and we're also going to add in the code we need to use cloudinary

```JS
//CREATE - add new campground to DB
router.post("/", isLoggedIn, upload.single('image'), async (req, res) => {
  // check if file uploaded otherwise redirect back and flash an error message
  if(!req.file) {
    req.flash('error', 'Please upload an image.');
    return res.redirect('back');
  }
  // try/catch for async + await code
  try {
      // get data from form and add to campgrounds array
      let name = req.body.name;
      let desc = req.body.description;
      let author = {
          id: req.user._id,
          username: req.user.username
      }
      let cost = req.body.cost;
      // upload image to cloudinary and set resulting url to image variable
      let result = await cloudinary.uploader.upload(req.file.path);
      let image = result.secure_url;
      // get map coordinates for location and assign to lat and lng variables
      let geoLocation = await geocoder.geocode(req.body.location);
      let location = geoLocation[0].formattedAddress;
      let lat = geoLocation[0].latitude;
      let lng = geoLocation[0].longitude; 
      // build the newCampground object
      let newCampground = {
        name: name,
        image: image,
        description: desc,
        cost: cost,
        author: author,
        location: location,
        lat: lat,
        lng: lng
      };
      // create campground
      await Campground.create(newCampground);
  } catch (err) {
      req.flash('error', err.message);
  }
  res.redirect('/campgrounds');
});
```

## Update the view
- Open `/views/campgrounds/new.ejs` inside of your code editor
- Change: `<form action="/campgrounds" method="POST">` to: `<form action="/campgrounds" method="POST" enctype="multipart/form-data">`
	- This allows us to upload a file/image with the form
- Replace the following:


```HTML
<div class="form-group">
  <label for="image">Image Url</label>
  <input class="form-control" type="file" name="image" id="image" pattern="^https:\/\/images\.unsplash\.com\/.*" title="Only images from images.unsplash.com allowed. See link below for instructions." placeholder="https://images.unsplash.com/..." required>
</div>
```

with:
 
 
```HTML
<div class="form-group">
  <label for="image">Image</label>
  <input type="file" id="image" name="image" accept="image/*" required>
</div>
```
 - Be sure to save the new.ejs file
 
 
## Try it out
- Fire up your mongod server and node server in your terminal then try to create a new campground with an uploaded image
 
### Notes:
- Users could potentialy upload NSFW pics to your site, if you want to prevent that from happening you'll need to employ one of the [moderation add-ons from Cloudinary](https://cloudinary.com/console/addons)
