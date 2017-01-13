<article id="post-249172" class="instapaper_body h-entry e-content">
I was stopped at an intersection the other day. It was raining. The road on the
other side sloped upwards, so I could see the stopped cars on the other side of 
the road kind of stadium-seating style. I could see all their windshield wipers 
going all at the same time, all out-of-sync with each other. Plus a few of them 
had seemingly kinda broken ones that flapped at awkward times and angles.

What does that have to do with web design and development? Nothing really,
other than that I took the scene as inspiration to create something, and it 
ended up being an interesting hodgepodge of "tricks
".

See the Pen [Cars with Weird Windshield Wipers][1] by Chris Coyier (
[@chriscoyier][2]) on [CodePen][3].

 [1]: https://codepen.io/chriscoyier/pen/woxRBW/
 [2]: http://codepen.io/chriscoyier
 [3]: http://codepen.io/

### It's SVG

When you need a little shape like a car, [nothing beats The Noun Project][4]

 [4]: https://thenounproject.com/search/?q=car<figure id="post-249211" class="
align-none media-249211
">

![][5]</figure>
I actually used the little Mac app they have and dragged the car I liked onto
Adobe Illustrator. Then drew two little lines on the windshield for the wipers. 
Literal, straight,`<line>`s. 

 [5]: img/noun-proj.png

### Repeating the SVG

I planned on showing a whole grid of cars. I could have just popped the SVG
into the HTML 20 times. But that isn't very efficient in that it lacks control. 
I figured programmatically looping was the way to go. Pug (the HTML preprocessor
) offers simple loops, so I went for that. At first, I did:

    - svg = '<svg viewBox="0 0 59 45.9" class="car"> ... </svg>' while cars < 20 - cars++ != svg

Figuring I could target "rows" of cars by using `:nth-child` selectors. For
example, if I wanted to select just the 10th-15th cars, I could do like
`.car:nth-child(n+11):nth-child(-n+15)`. In the end, it was easier to target a
whole "row" of cars grouped together, so they could scale all together. So:

    - cols = 0 - rows = 0 - svg = '<svg viewBox="0 0 59 45.9" class="car"> ... </svg>' while rows < 4 - rows++ div.car-row - cols = 0 while cols < 5 - cols++ != svg

### Sizing

Each car has a particular aspect ratio. Notice the viewBox of the SVG. I
figured it would be best to size them according to that aspect ratio. I set the 
aspect ratio in pixels, as variables, then I could use a multiplier to scale 
them. For example, here I'm doubling their "size
":

    :root { --carWidth: 59px; --carHeight: 46px; } .car { width: calc(var(--carWidth) * 2); height: calc(var(--carHeight) * 2); }

Before I decided to break up the "rows" of cars with divs, I was able to force
the floated cars into rows by limiting the width of the body with a multiple of 
the width of a car.

### Animating the Wipers

The animation of the wiper is clearly a rotation transform. Normally I'd worry
about that in SVG, as transforms on SVG elements are
[notoriously inconsistent across browsers][6]. That's why I reached for GSAP,
which normalizes that.

 [6]: http://css-tricks.stfi.re/svg-animation-on-css-transforms/

My first thought was to set up a timeline. Timelines in GSAP have a `yoyo`
parameter that make good sense for the back-and-forth style motion of a 
windshield wiper. We'll use rotation, anchored at the bottom right, where the 
wipers pivot.

    var wipers = document.querySelectorAll(".wiper"); var tl = new TimelineMax({ repeat: -1, yoyo: true }); tl.to(wipers, 0.6, { rotation: 90, transformOrigin: "bottom right", ease: Expo.easeOut, });

### Randomizing

A helper function to spit out pseudo random numbers:

    function getRandomInt(min, max) { return Math.floor(Math.random() * (max - min + 1) + min); }

Now we can add randomization like delays and how far the rotation actually goes
:

    var tl = new TimelineMax({ repeat: -1, yoyo: true, delay: getRandomInt(1, 4) }); tl.to(wipers, 0.6, { rotation: function() { return getRandomInt(80, 140); }, transformOrigin: "bottom right", ease: Expo.easeOut, });

This works pretty well, it's just that each wiper then has a *set* timeline
that it follows, it doesn't randomize each iteration. We can get a little closer
by looping over each wiper and applying a unique timeline to each:

    wipers.forEach(function(el, i) { var tl = new TimelineMax({ repeat: -1, yoyo: true, delay: getRandomInt(1, 4) }); tl.to(el, 0.6, { rotation: function() { return getRandomInt(80, 140); }, transformOrigin: "bottom right", ease: Expo.easeOut, }); });

### Callback randomization

To make each iteration randomly rotate, I think it might be easier to not
actually use a timeline, but just call a single animation method over and over 
as a callback. That way each time we call it, it can be randomized. So rather 
than`TimelineMax()`, we'll use `TweenLite` and abstract it into our own
function.

    function doWiperAnimation(el) { TweenLite.to(el, 0.5, { delay: getRandomInt(0.1, 0.3), rotation: function() { return getRandomInt(0, 140); }, transformOrigin: "bottom right", ease: Power0.easeNone, onComplete: function() { doWiperAnimation(el); } }); }

Note how the `onComplete` callback calls itself. Animation loop! We just need
to kick it off once:

    wipers.forEach(function(el, i) { doWiperAnimation(el); });

There are no limits to how weird you wanna get with what you randomize. Here's
how you might even randomize which easing you pick:

    var easings = [ "SlowMo.ease.config(0.7, 0.7, false)", "Power0.easeNone", "Power2.easeOut" ]; ... ease: easings[Math.floor(Math.random()*easings.length)]