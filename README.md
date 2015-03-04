# TunguskaGauge

Embed dynamic gauges into your Meteor app.

# Install

```JavaScript
meteor add tunguska:gauge
```

# Demo

tbd

# Basic Usage

## Instantiate a new gauge

```JavaScript
var someGauge = new TunguskaGauge(options);
```

### Options

The instantiation `options` allow you to define a gauge's style. In fact gauge theme packages can be used to provide 
a variety of pre-built gauge designs. Gauges may be fully rendered, use images as pointers, or a combination.
An individual gauge may have more than one pointer (e.g. a clock could have three: hours, minutes and seconds).

The full `options` object with sample values is as follows. Note that most options are, well, optional:

```JavaScript
{
  id: 'DOM-id',                       // The DOM id of a container for the gauge (e.g. a div)
  theme: 'basic',                     // The name of a predefined theme to base this gauge off
  radius: 0.95,                       // A value between 0 and 1 proportional to the size of the radius within the container
  range: {                            // Start of the range object
    min: 0,                           // The minimum value of the gauge
    max: 150,                         // The maximum value of the gauge
    lowStop: -3,                      // Where to stop the pointer if a value smaller than range.min is supplied
    highStop: 153,                    // Where to stop the pointer if a value larger than range.max is supplied
    startAngle: -135,                 // Where the pointer starts in degrees (vertically up is 0)
    sweep: 270                        // How many degrees to sweep for the full range (+ sweeps clockwise, - sweeps anticlockwise)
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

### Update or read a gauge

```
someGauge.set(newValue);
```

Where newValue is a simple pointer value, or an array of pointer values. You can also read a gauge to get its current value:

```
var myValue = someGauge.get();
```

Where myValue will be set to a simple value or an array of values.

### Easing

The easing data is in TunguskaGauge.easing

It is an object of named cubic Bezier control arrays, each being an array of four waypoints (t,v):
[[t0,v0],[t1,v1],[t2,v2],[t3,v3]] which form the control points of a cubic Beziér Curve.
Under normal circumstances, the first point will be [0,0] and the fourth will be [1,1].

```JavaScript
TunguskaGauge.easing = {
  linear: [
    [0, 0],
    [0.33, 0.33],
    [0.67, 0.67],
    [1, 1]
  ],
  bounce: [
    [0, 0],
    [0.73, 1],
    [1, 1.3],
    [1, 1]
  ],
  instant: [
    [0, 1],
    [0, 1],
    [0, 1],
    [1, 1]
  ],
  easeIn: [
    [0, 0],
    [0.42, 0],
    [1, 1],
    [1, 1]
  ],
  easeOut: [
    [0, 0],
    [0, 0],
    [0.58, 1],
    [1, 1]
  ],
  easeInOut: [
    [0, 0],
    [0.42, 0],
    [0.58, 1],
    [1, 1]
  ]
};
```

## Themes

Themes are in TunguskaGauge.themes

A basic theme is built in:

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

## Examples

## Todo

