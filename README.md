# TunguskaGauge

Embed reactive, dynamic gauges into your Meteor app.

Uses canvas, so if you're on IE8 or below, you're out of luck.

# Install

```JavaScript
meteor add tunguska:gauge
```

# Demo

http://tunguska-gauge-demo.meteor.com

![Demo Gauges](docs/gauges.png)

# Basic Usage

## Instantiate a new gauge

```JavaScript
var someGauge = new TunguskaGauge(options);
```

### Options

The instantiation `options` allow you to define a gauge's style. In fact gauge theme packages can be used to provide
a variety of pre-built gauge designs. Gauges may be fully rendered, use images as pointers, or a combination.
An individual gauge may have more than one pointer (e.g. a clock could have three: hours, minutes and seconds).

The full `options` object with sample values is as follows. Note that most options are, well, optional. Note that the
terms `range units` and `radius units` are used for certain values.

A `range unit` is a number between `range.min` and `range.max` inclusive.

A `radius unit` is a number between 0 and 1 which will internally be multiplied by the radius size of the containing div. The
radius size is calculated as half of the smaller of the height and width. That is the radius of the largest circle which
can be drawn within the container. For example, a div which is sized as 200 x 150 (px) has a radius size of 75px.
TunguskaGauge does not currently support non-circular gauges (e.g. ellipses).

```JavaScript
{
  id: 'DOM-id',                       // The DOM id of a container for the gauge (e.g. a div)
  theme: 'basic',                     // The name of a predefined theme to base this gauge off
  radius: 0.95,                       // A value in radius units (0..1)
  range: {                            // Start of the range object
    min: 0,                           // The minimum value of the gauge
    max: 150,                         // The maximum value of the gauge
    lowStop: -3,                      // Where to stop the pointer if a value smaller than range.min is supplied
    highStop: 153,                    // Where to stop the pointer if a value larger than range.max is supplied
    startAngle: -135,                 // Where the pointer starts in degrees (vertically up is 0)
    sweep: 270,                       // How many degrees to sweep for the full range (+ sweeps clockwise, - sweeps anticlockwise)
    colorBand: [{                     // Start of colour bands
      startAt: 0.95,                  // A narrow, green band from 0 to 75 range units
      endAt: 0.99,
      from: 0,
      to: 75,
      color: '#0d0'
    },{
      startAt: 0.90,                  // A wider, amber band from 75 to 90
      endAt: 0.99,
      from: 75,
      to: 90,
      color: '#ed0'
    }, {                              //
      startAt: 0.85,                  // An even wider, red band from 90 to 100
      endAt: 0.99,
      from: 90,
      to: 100,
      color: '#d00'
    }]
    },
  },
  background: {                       // Start of the background object
    image: 'image-file.png',          // Use an image for the background
    left: -50,                        // Reposition the background to line up with the pointer (px)
    top: -50                          // Reposition the background to line up with the pointer (px)
  },
  foreground: {                       // Start of the foreground object
    image: 'image-file.png',          // Use an image for the foreground
    left: -50,                        // Reposition the foreground to line up with the pointer (px)
    top: -50                          // Reposition the foreground to line up with the pointer (px)
  },
  digital: {                          // Start of digital object (for text representation of pointers)
    top: 75,                          // Where to place the top of the text block (px)
    left: 0,                          // Where to place the anchor point of the text
    font: '12px monospace',           // The font to use
    color: '#0f0',                    // The colour to use
    callback: function(pV) {          // A callback if anything special needs doing
      code here;                      //  to convert a pointer value (pV) to text
    }                                 //  (e.g. convert a Date object for a clock)
  },
  outer: {                            // Start of outer object (draws a border round the gauge)
    lineWidth: 1,                     // Thickness of border line (px)
    color: 'white',                   // Colour of line
    alpha: 0.5,                       // Opacity (0 - fully transparent .. 1 - fully opaque)
    radius: 1                         // Radius value(0..1) proportional to the size of the container
  },
  callback: {                         // Define general pointer value conversion functionality
    pointer: function(pV) {           //   Callback to convert a pointer value (pV) to a usable range value
      code here;                      //   (e.g. convert a Date object to hours, minutes and seconds for a clock)
    },                                //
    wrap: true                        // If the pointer should wrap around from max to min when max is exceeded
  },                                  //   (e.g. a clock hands shouldn't wind backwards at midday/midnight)
  tick: {                             // Start of tick definitions
    minor: {                          // Start of minor (small) ticks
      lineWidth: 1,                   // Thickness (px) of tick marks
      startAt: 0.95,                  // Start radius (0..1)
      endAt: 0.99,                    // End radius (0..1)
      interval: 5,                    // The spacing around the gauge in range units
      color: 'white',                 // The colour
      alpha: 1,                       // The opacity
      first: 5,                       // The first tick in range units
      last: 145                       // The last tick in range units
    },
    major: {                          // Start of major (large) ticks
      lineWidth: 2,                   // Thickness (px) of tick marks
      startAt: 0.9,                   // Start radius (0..1)
      endAt: 0.99,                    // End radius (0..1)
      interval: 25,                   // The spacing around the gauge in range units
      color: 'white',                 // The colour
      legend: {                       // Start of legend definition
        color: 'white',               // Legend colour
        font: '12px sans-serif',      // Legend font
        radius: 0.75,                 // Distance from centre (0..1)
        callback: function(n) {       // Callback if any special conversion needed: n is major tick value
          code here;                  //
        }
      },
      alpha: 1,                       // Opacity (0..1) of major ticks
      first: 0,                       // First tick in range units
      last: 150                       // Last tick in range units
    }
  },
  events: {
    onPointerStart: function(g, v) {},// Define event callbacks for when the pointer is about to sweep,
    onPointerSweep: function(g, v) {},// is sweeping,
    onPointerStop: function(g, v) {}  // or has finished sweeping. "g" is the gauge theme, "v" the pointer values
  },
  render: true,                       // Whether to render the gauge automatically
  pointer:                            // See below
}
```

The pointer object can be a simple object: {} or an array of objects: [{},{},...,{}]

Use a simple object for a single pointer. Use an array of objects for multiple pointers.

The basic pointer object is as follows:

```JavaScript//
pointer: {                            // Start of pointer definition
  image: {                            // Pointer is an image. Note that image pointers trump rendered pointers
    name: 'pointer-image.png',        // The image pointer file name
    xOffset: 32,                      // Where the pointer centre is in the image
    yOffset: 15                       //
  },
  shadow: {                           // Shadow is an image.
    name: 'shadow-image.png',         // The shadow pointer file name
    xOffset: 32,                      // Where the shadow centre is in the image
    yOffset: 15                       //
  },
  points: [                           // Pointer is rendered
    [-0.1, -0.05],                    //   (x,y) Co-ordinates of pointer relative to centre (0,0)
    [0.7, 0],                         //
    [-0.1, 0.05]                      // Note that final point will close the shape
  ],
  lineWidth: 1,                       // Thickness of pointer outline (rendered only)
  color: "white",                     // Colour of pointer outline (rendered only)
  fillColor: "white",                 // Colour of pointer infill (rendered only)
  alpha: 1,                           // Opacity of pointer (0..1) (rendered only)
  shadowBlur: 1,                      // Amount of shadow blur (px) to apply (rendered only)
  shadowColor: "#000"                 // Shadow colour (rendered only)
  shadowX: 1,                         // Offset of shadow from pointer in px
  shadowY: 1,                         //   (also applies to image shadows)
  dynamics: {                         // How to move the pointer
    duration: 100,                    //   Move in 100ms
    easing: 'bounce'                  //   Use 'bounce' easing
  }
}
```
### Gauge Methods

#### Update pointer

`someGauge.set(newValue);`

Where newValue is a simple pointer value, or an array of pointer values.

#### Read current pointer

`var myValue = someGauge.get();`

Where myValue will be set to a simple value or an array of values.

#### Get reference to current gauge theme

`var myTheme = someGauge.getTheme();`

Permits direct getting/setting of theme, e.g. to change the duration of pointer[2]:

`myTheme.pointer[2].dynamics.duration = 1000;`

#### Redraw gauge

`someGauge.redraw(value);`

May also be used to initially draw the gauge if `render: false` was set in the instantiation options. Note that this also sets the `render`
property to true. If specified, `value` sets the pointer value to use (a simple value or an array of values).

### Easing

The easing data is in `TunguskaGauge.easing`

It is an object of named easing functions. These take one parameter (t), which takes
a value between 0 and 1, representing the "distance" along the easing duration time.
Functions return a number between 0 and 1 indicating the distance travelled by the
gauge pointer at that time. Note that some functions may exceed 1 (e.g. "bounce") or
may never return 0 (e.g. "instant").

```JavaScript
TunguskaGauge.easing = {
  /*
   *  Mainly from https://gist.github.com/gre/1650294
   */
  // no easing, no acceleration
  linear: function(t) {
    return t
  },
  // accelerating from zero velocity
  easeInQuad: function(t) {
    return t * t
  },
  // decelerating to zero velocity
  easeOutQuad: function(t) {
    return t * (2 - t)
  },
  // acceleration until halfway, then deceleration
  easeInOutQuad: function(t) {
    return t < .5 ? 2 * t * t : -1 + (4 - 2 * t) * t
  },
  // accelerating from zero velocity
  easeInCubic: function(t) {
    return t * t * t
  },
  // decelerating to zero velocity
  easeOutCubic: function(t) {
    return (--t) * t * t + 1
  },
  // acceleration until halfway, then deceleration
  easeInOutCubic: function(t) {
    return t < .5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1
  },
  // accelerating from zero velocity
  easeInQuart: function(t) {
    return t * t * t * t
  },
  // decelerating to zero velocity
  easeOutQuart: function(t) {
    return 1 - (--t) * t * t * t
  },
  // acceleration until halfway, then deceleration
  easeInOutQuart: function(t) {
    return t < .5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t
  },
  // accelerating from zero velocity
  easeInQuint: function(t) {
    return t * t * t * t * t
  },
  // decelerating to zero velocity
  easeOutQuint: function(t) {
    return 1 + (--t) * t * t * t * t
  },
  // acceleration until halfway, then deceleration

  easeInOutQuint: function(t) {
    return t < .5 ? 16 * t * t * t * t * t : 1 + 16 * (--t) * t * t * t * t
  },
  // bounce effect
  bounce: function(t) {
    var p = 0.3;
    return Math.pow(2, -10 * t) * Math.sin((t - p / 4) * (2 * Math.PI) / p) + 1;
  },
  // no delay
  instant: function(t) {
    return 1;
  },
  // alternative names
  easeIn: function(t) {
    return this.easeInCubic(t);
  },
  easeOut: function(t) {
    return this.easeOutCubic(t);
  },
  easeInOut: function(t) {
    return this.easeInOutCubic(t);
  },
};
```

## Themes

Themes are in `TunguskaGauge.themes`

A theme provides a named initial set of options for a new TunguskaGauge instance. A basic theme is built in (called "basic"), which can be superseded if you have another theme package.

```JavaScript
TunguskaGauge.themes = {
  basic: {
    radius: 0.85,
    range: {
      min: 0,
      max: 100,
      startAngle: -135,
      sweep: 225,
      colorBand: [{
        startAt: 0.95,
        endAt: 0.99,
        from: 0,
        to: 75,
        color: '#090'
      }, {
        startAt: 0.90,
        endAt: 0.99,
        from: 75,
        to: 90,
        color: '#e80'
      }, {
        startAt: 0.85,
        endAt: 0.99,
        from: 90,
        to: 100,
        color: '#d00'
      }]
    },
    outer: {
      lineWidth: 1,
      color: 'black',
      alpha: 0.5,
      radius: 1
    },
    pointer: {
      points: [
        [-0.1, -0.05],
        [0.95, 0],
        [-0.1, 0.05]
      ],
      lineWidth: 1,
      color: 'black',
      alpha: 1,
      fillColor: 'red',
      shadowX: 2,
      shadowY: 2,
      shadowBlur: 5,
      shadowColor: '#000',
      dynamics: {
        duration: 150,
        easing: 'easeIn'
      }
    },
    tick: {
      minor: {
        lineWidth: 1,
        startAt: 0.90,
        endAt: 0.96,
        interval: 2,
        color: 'black',
        alpha: 1,
        first: 0,
        last: 100
      },
      major: {
        lineWidth: 2,
        startAt: 0.86,
        endAt: 0.96,
        interval: 10,
        color: 'black',
        legend: {
          color: '#669',
          font: '12px sans-serif',
          radius: 0.72
        },
        alpha: 1,
        first: 0,
        last: 100
      }
    },
    digital: {
      top: 40,
      left: 0,
      font: '20px monospace',
      color: '#66a'
    }
  }
};
```

Normally, you would specify a theme early on in the options object passed to the `new TunguskaGauge` command. For example:

```JavaScript
var anotherGauge = new TunguskaGauge({
  id: 'gauge-id',
  theme: 'steampunk',                             // Use steampunk theme
  background: {                                   // Then override its background image
    image: '/public/images/steampunk99.png'
  }
});
```

## ThemePacks

TunguskaGauge tries to find a named theme in the global TunguskaGaugeThemePack object. If it cannot be found (e.g. a theme pack has
not been installed, or the named theme is not in the theme pack) it will try the default themes. If all else fails it will try the
overall default theme (currently "basic").

## Examples

### Basic

Set of five demo gauges (the first has random numbers served from meteor.com as a pub/sub): http://tunguska-gauge-demo.meteor.com

GIT repo: https://github.com/robfallows/tunguska-gauge-demo

## Todo

- ~~Tests~~
- Themeroller.
- Improve Annotation.
- ~~Include requestAnimationFrame polyfill.~~
- ~~Better handling of options overrides.~~
- ~~Better easing: bespoke functions, rather than cubic Bezier interpolation~~
- ~~More demo gauges.~~
- Non-linear scales (vu-meters, anyone?)
- Changelog

## Tests

Package testing courtesy [practicalmeteor:munit](https://atmospherejs.com/practicalmeteor/munit).

![Test Results](docs/tests.png)

## Contributors

@techieyann

## Licence

The MIT License (MIT)

Copyright (c) 2015 Rob Fallows

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

