# Wolfenstein 5k Running on Modern Browsers

[Play the demo](https://parkertomatoes.github.io/wolf5k/wolf5k.html)

## Background

This is a patch to make Lee Semel's [Wolfenstein 5k](http://www.wolf5k.com/) demo, which rocked my socks off back in high school, run on modern browsers.

![screenshot][screenshot.png]

If you're seeing this for the first time in 2018, it's... not impressive. Now, there are Javascript ports of say, [Team Fortress 2](https://github.com/toji/webgl-source), or the [actual Wolfenstein 3d](https://archive.org/details/msdos_Wolfenstein_3D_1992), running in a DOS emulator. But in 2002, Javascript wasn't for games. It was awful and slow, and every game made without Flash was some board game rendered with HTML tables. I thought it was rad that Wolfenstein 5K could render individual pixels at all, nevermind a working raycaster with sprites, textures, and shading. All code golfed into 5k, with so few dependencies it runs in _Netscape Navigator_.

But sadly, it hasn't worked in a decade because all major browsers have dropped XBM image support. The demo used this mechanism to dump pixels into an img element, so now the game looks like a broken image link, even though it's running in the background. I thought it would be fun for a Saturday project to get the demo working again.

## The Fix

My goal was to avoid hacking up the original source, since the code itself is a feature of the demo. Instead, this code was tacked on to the end. It intercepts assignments to "img.src" each frame, parses the XBM image passed to it, renders it to an HTML5 canvas, and copies that image back to the img tag. It's not particularly optimized, but it runs fine on my PC. I'm no good at code golfing, so I'm not even going to try to minify it.

```javascript
var screen = document.images[0];
var canvas = document.createElement('canvas');
canvas.width = screen.width;
canvas.height = screen.height;
const context = canvas.getContext('2d');
var xbmHeader = /\s*#define\s*[^\s]+\s*(\d+)\s*#define\s*[^\s*]+\s*(\d+)\s*static\s*char\s*[^\s\[]+\[\]\s*=\s*{/;
Object.defineProperty(screen, 'src', { 
  set: function(src) {
    if (!/javascript:\d+;im;/.test(src)) {
      screen.setAttribute('src', src);
      return;
    }
    const header = im.match(xbmHeader);
    if (header === null) {
      throw new Error('not an XBM: ' + src);
    }
    const pixels = im
      .substring(header.lastIndex)
      .match(/0x[\da-fA-F]{2,2}/g)
      .map(x => parseInt(x, 16));
    const cvData = context.createImageData(header[1], header[2]);
    let cvDataIndex = 0;
    for (const pixel of pixels) {
      for (let bitIndex=0; bitIndex<8; bitIndex++) {
        const bit = pixel & (1 << bitIndex) ? 255 : 0;
        cvData.data[cvDataIndex++] = bit;
        cvData.data[cvDataIndex++] = bit;
        cvData.data[cvDataIndex++] = bit;
        cvData.data[cvDataIndex++] = 255;
      }
    }
    context.putImageData(cvData, 0, 0);
    screen.setAttribute('src', canvas.toDataURL());
  }
});
```

