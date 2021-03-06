# Website Optimization Project

This is my project for the Udacity [Front-End Web Developer Nanodegree](https://www.udacity.com/course/front-end-web-developer-nanodegree--nd001). The goal of this project was to optimize webpages by making them faster.

You can find the original github repository at [https://github.com/udacity/frontend-nanodegree-mobile-portfolio](https://github.com/udacity/frontend-nanodegree-mobile-portfolio).

## Basic Project Setup - Windows version

* The unminified version is in the **src** folder.
* The minified version is in the **dist** folder.
* Enter your project folder from the terminal and run a local webserver with **python -m http.server 8080**
* Use [ngrok](https://ngrok.com/) to start a tunneling service.
* Ngrok will provide you with a custom address.
* Use this address to check your page against [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)
* Test the FPS metrics against the Chrome Dev Timeline Tool.

## Learning about Gulp!

The most fun part of this project was to learn about Taskrunners like Grunt and Gulp. Before this project I knew nothing about them and it took me about a weekend to write my own gulpfile.js. For this project I have used the following plugins:

* [gulp-uglify](https://www.npmjs.com/package/gulp-uglify)
* [gulp-clean-css](https://www.npmjs.com/package/gulp-clean-css)
* [gulp-htmlmin](https://www.npmjs.com/package/gulp-htmlmin)
* [gulp-imagemin](https://www.npmjs.com/package/gulp-imagemin)

I used the following resources to learn about Gulp:

* [Gulp for Beginners by css-tricks](https://css-tricks.com/gulp-for-beginners/)
* [Udacity Screencast - Setting up a gulp workflow](https://plus.google.com/u/0/events/cv9skua854h0rr1qf9b6pisl87g?authkey=COLTgKmx35_NZw)
* [Learning Gulp - 11 Lessons on Gulp by levelup Tutorials](https://leveluptutorials.com/tutorials/learning-gulp)
* [Udacity Forum WriteUp on basic Gulp](https://discussions.udacity.com/t/gulp-and-setting-up-a-gulp-workflow-intermediate/24359/3)

## Part I: Optimizing index.html

In the first part of this project we had to optimize index.html to reach a [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) Score of 90 or higher for desktop and mobile.

I made the following changes to reach my goal:

* added a **media="print"** tag for the print.css stylesheet, so that it wouldn't block rendering.
* After numerous searches on the udacity forum I used the [Web Font Loader](https://github.com/typekit/webfontloader) to load the Google Font asynchronously.
* Set **analytics.js** and **perfmatters.js** to asynchronous so that DOM construction is not paused and moved Google Analytics script to bottom of body.
* Inline critical CSS only, so it can be delivered to the client as soon as possible to optimize the render time.
* I used gulp to uglify JavaScript, minify CSS, minify HTML and to optimize the images.
* **pizzeria.jpg** was a special case, because its actual size was much to big. I had to resize it and optimize for the web with adobe photoshop.

## Part II: The hunt for 60 FPS

In the second part of this project we had to achieve 60 FPS when scrolling in pizza.html. Before I started the average time to generate the last 10 frames was between 20ms and 30ms. After taking all the steps described below I was able to chunk it down, to between 0.1ms and 0.2ms, and reaching constant 60fps while scrolling.

**starting at line 495 on main.js**
```javaScript
// querySelectorAll is slow, better use getElementsByClassName
var items = document.getElementsByClassName('mover');
```

**starting at line 497 on main.js**
```javaScript
// no need to declare phase everytime the loop starts, so declare variable beforehand
var phase;
```

**starting at line 499 on main.js**
```javaScript
// no need to search for document.body.scrollTop in every iteration, because it will never change
// after testing the average scripting time of the last 10 frames, I found out, that this step was a huge win.
// Scripting Time went down from 15 to 0.5 - not bad!
var cachedScrollTop = document.body.scrollTop / 1250;
```

**starting at line 503 on main.js**
```javaScript
// declaration outside of for loop will add to performance
var dist;
```

**starting at line 503 on main.js or welcome to the tricky part**
```javaScript
for (var i = 0; i < items.length; i++) {
    phase = Math.sin(cachedScrollTop + (i % 5));
    dist = items[i].basicLeft + 100 * phase - 1024 + 'px';
    // transform and translate were the missing link because they are avoiding layout and paint.
    items[i].style.transform = 'translateX(' + dist + ')';
}
```

**starting at line 524 on main.js**
```javaScript
// Generates the sliding pizzas when the page loads.
document.addEventListener('DOMContentLoaded', function() {
    var cols = 8;
    // s describes the height of a row
    var s = 256;
    // How many rows are needed? Window.innerHeight divided by the space between pizzas
    var rows = Math.ceil(window.innerHeight / s);
    // in the original code, there are always 200 pizzas flying around, on my screen most of them are not shown.
    // situation gets even worth on smaller screens or mobile.
    // to fix the problem we need changing numbers of pizzas according to the height of the device
    var pizzaNum = cols * rows;
    for (var i = 0; i < pizzaNum; i++) {
        var elem = document.createElement('img');
        elem.className = 'mover';
        elem.src = "images/pizza.png";
        elem.style.height = "100px";
        elem.style.width = "73.333px";
        elem.basicLeft = (i % cols) * s;
        elem.style.top = (Math.floor(i / cols) * s) + 'px';
        // Updated querySelector to getElementById for efficiency.
        document.getElementById("movingPizzas1").appendChild(elem);
    }
```

### Part III Resizing the pizzas

In the last part of this project we had to optimize the time, when a pizza is resized. Initially this time is measured around 140ms. After optimizing the code I was able to shrink it down to around 0.35ms.

The following code was added and changed:

**starting at line 422 on main.js**
```javaScript
// resizePizzas(size) is called when the slider in the "Our Pizzas" section of the website moves.
var resizePizzas = function(size) {
    window.performance.mark("mark_start_resize"); // User Timing API function
// Convert size to percentage number and change value for the size of the pizza above the slider.
    var pizzaSizeLabel = "";
    switch (size) {
        case "1":
            newWidth = 25;
            pizzaSizeLabel = "Small";
            break;
        case "2":
            newWidth = 33.3;
            pizzaSizeLabel = "Medium";
            break;
        case "3":
            newWidth = 50;
            pizzaSizeLabel = "Large";
            break;
        default:
            console.log("bug in resizePizzas switch");
    }
    document.getElementById("pizzaSize").innerHTML = pizzaSizeLabel;
    var rngPizza = document.getElementsByClassName("randomPizzaContainer");
    for (var i = 0; i < rngPizza.length; i++) {
        rngPizza[i].style.width = newWidth + "%";
    }
var dist;
```



