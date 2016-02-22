# vr-interactives-three-js
There are a lot of Three.js [examples](http://threejs.org/examples/) and [tutorials](http://learningthreejs.com/) out there, but very few examples of what you can do with real-world data. 

In this session you'll learn how we used NASA satellite imagery and elevation data to create a 3-D rendering of the [Gale Crater](http://graphics.latimes.com/mars-gale-crater-vr/) on Mars.  

## Requirements
We'll make use of the [WebVR Boilerplate](https://github.com/borismus/webvr-boilerplate), which also uses the [WebVR polyfill](https://github.com/borismus/webvr-polyfill) to provide VR support when the WebVR spec isn't implemented.  

We'll also use a [terrain loader](http://blog.thematicmapping.org/2013/10/terrain-building-with-threejs.html), developed by Bjørn Sandvik. 

## Let's get started
Using Three.js can be compared a bit to filmmaking: you have a scene, lighting and a camera. The scene updates a certain number of times per second, otherwise known as "frames per second" (which we'll try to keep as close to 60 as we can, but will drop based on your computer and the complexity of the scene.)

We'll build this in the file [three-demo.html](three-demo.html).

In the empty script tag on line 27, let's start by declaring some constants. We'll use this to set the size of the renderer on the screen, and the "size" of our world. 

```javascript
var WINDOW_WIDTH = window.innerWidth,
    WINDOW_HEIGHT = window.innerHeight,
    WORLD_WIDTH = 2000,
    WORLD_HEIGHT = 1900;
```

Then, we're ready to create the scene and place the camera:

```javascript
var container = document.getElementById("webgl"),
    scene = new THREE.Scene(),
    camera = new THREE.PerspectiveCamera(75, WINDOW_WIDTH / WINDOW_HEIGHT, 1, 5000);
```

The [Perspective Camera](http://threejs.org/docs/#Reference/Cameras/PerspectiveCamera) functions similar to a standard photo or video camera you might be familiar with. The first parameter, `75` is the field of view, followed by the aspect ratio (in this case the width and height of the window), along with the minimum and maximum ranges the camera can "see." Anything outside these ranges will not be rendered in the scene. 

On that note, let's create the renderer. We're going to be creating a [WebGL Renderer](http://threejs.org/docs/#Reference/Renderers/WebGLRenderer), which uses the WebGL API to render our graphics. It has way better performance than the [Canvas Renderer](http://threejs.org/docs/#Reference/Renderers/CanvasRenderer) and you'll want to use it whenever possible. 

```javascript
    var renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setClearColor(0xffd4a6);
    renderer.setSize(WINDOW_WIDTH, WINDOW_HEIGHT);
    container.appendChild( renderer.domElement );
```

The "clear color" of the renderer is set to a dusty orange/red, and the size is set to the size of the window. As the last step, the renderer is appended to the DOM. 

Now, we need to add a couple of helpers that allow VR-style stereo effects to our renderer, along with a manager to allow us to switch between VR and non-VR contexts.

```javascript
    // Apply VR stereo rendering to renderer
    var effect = new THREE.VREffect(renderer);
    effect.setSize(WINDOW_WIDTH, WINDOW_HEIGHT);

    // Create a VR manager helper to enter and exit VR mode
    var manager = new WebVRManager(renderer, effect);
```
