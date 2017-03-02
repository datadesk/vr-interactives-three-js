# vr-interactives-three-js
There are a lot of Three.js [examples](http://threejs.org/examples/) and [tutorials](http://learningthreejs.com/) out there, but very few examples of what you can do with real-world data.

In this session you'll learn how we used NASA satellite imagery and elevation data to create a 3-D rendering of the [Gale Crater](http://graphics.latimes.com/mars-gale-crater-vr/) on Mars.

## Requirements and credits
We'll make use of the [WebVR Boilerplate](https://github.com/borismus/webvr-boilerplate), which also uses the [WebVR polyfill](https://github.com/borismus/webvr-polyfill) to provide VR support when the WebVR spec isn't implemented.

We'll also use a [terrain loader](http://blog.thematicmapping.org/2013/10/terrain-building-with-threejs.html), developed by [Bjørn Sandvik](http://blog.thematicmapping.org/).

## Preamble
Before we start, we'll want to start up a small webserver that can serve our page and assets. Open the terminal on your computers and navigate to the directory we'll be using (on the lab computers. If you're on your own rig, [download a copy of this repository](https://github.com/datadesk/vr-interactives-three-js/archive/master.zip)) and enter the following at the prompt.

```bash
$ cd /data/vr_interactives # or wherever you've downloaded this to
$ python -m SimpleHTTPServer
```
![](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/runserver.gif)

*Note: If you're using Python 3, you may have to run [http.server](http://angusjune.github.io/blog/2014/08/16/python-3-dot-x-no-module-named-simplehttpserver/) instead.*

Now you'll be able to go to [http://localhost:8000/three-demo.html](http://localhost:8000/three-demo.html) and navigate to the page we're going to develop. That page will be blank for now.

## Let's get started
Using Three.js can be compared a bit to filmmaking: you have a scene, lighting and a camera. The scene updates a certain number of times per second, otherwise known as "frames per second" (which we'll try to keep as close to 60 as we can, but will drop based on your computer and the complexity of the scene.)

We'll build this in the file [three-demo.html](three-demo.html). Go ahead and open this file in a text editor.

In the empty script tag on line 33, let's start by declaring some constants. We'll use this to set the size of the renderer on the screen, and the "size" of our world.

```javascript
// Width and height of the browser window
var WINDOW_WIDTH = window.innerWidth;
var WINDOW_HEIGHT = window.innerHeight;

// Width and height of the surface we're going to create
var WORLD_WIDTH = 2000;
var WORLD_HEIGHT = 1900;
```

Then, we're ready to create the scene and place the camera:

```javascript
// Where our lights and cameras will go
var scene = new THREE.Scene();

// Keeps track of time
var clock = new THREE.Clock();

// How we will see the scene
var camera = new THREE.PerspectiveCamera(75, WINDOW_WIDTH / WINDOW_HEIGHT, 1, 5000);
```

The [PerspectiveCamera(fieldOfView, aspect, near, far)](http://threejs.org/docs/#Reference/Cameras/PerspectiveCamera) functions similar to a standard photo or video camera you might be familiar with. The first parameter, `75` is the vertical field of view in degrees, followed by the aspect ratio (in this case the width and height of the window), along with the minimum and maximum ranges the camera can "see." Anything outside these ranges will not be rendered in the scene.

We also want to position the camera, and tell it where to look. These settings position the camera slightly above the scene, looking at the mound in the center of the crater.

```javascript
// Position the camera slightly above and in front of the scene
camera.position.set(0, -199, 75);
camera.up = new THREE.Vector3(0,0,1);

// Look at the center of the scene
camera.lookAt(scene.position);
```

Now let's create the renderer. We're going to be creating a [WebGL Renderer](http://threejs.org/docs/#Reference/Renderers/WebGLRenderer), which uses the WebGL API to render our graphics.

```javascript
// Think of the renderer as the engine that drives the scene
var renderer = new THREE.WebGLRenderer({antialias: true});

// Set the pixel ratio of the screen (for high DPI screens)
// This line is actually really important. Otherwise you'll have lots of weird bugs and a generally bad time.
renderer.setPixelRatio(window.devicePixelRatio);

// Set the background of the scene to a orange/red
renderer.setClearColor(0xffd4a6);

// Set renderer to the size of the window
renderer.setSize(WINDOW_WIDTH, WINDOW_HEIGHT);

// Append the renderer to the DOM
document.body.appendChild( renderer.domElement );
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

If you try to load the scene now, you're just going to see a blank screen. That's because while we're initializing the scene, we're not actually rendering it. To do that, we need to write what's called a render loop. 

```javascript
// Render loop
// This should go at the bottom of the script.
function render() {

    // We'll add more up here later

    // Call the render function again
    requestAnimationFrame( render );

    // Update the scene through the manager.
    manager.render(scene, camera);
}

render();
```

Now, if you load the scene, you should see an orange/red window. There's nothing there, but we'll get to that in a bit. 


## Prepping the DEM data
![Digital elevation model of Gale Crater. Brighter values are higher elevations. (NASA)](http://www.trbimg.com/img-562bfe79/turbine/la-mars-dem-map-20151024/600 "Digital elevation model of Gale Crater. Brighter values are higher elevations. (NASA)")

A digital elevation model is a 3D representation of a terrain's surface, and in this case is a greyscale heightmap, where lighter colors represent higher elevations.

The DEM data came as a GeoTIFF file (Gale_HRSC_DEM_50m_overlap.tif). Unfortunately, the TIFF file is huge (30 MB!), and TIFF isn't supported for display by most browsers anyway. We could use the incredibly useful [GDAL](http://www.gdal.org/) to convert it to a PNG image, where the height values are reduced to only 256 shades of grey. This would make our terrain blocky, however.

We can, however, convert it to a format called ENVI, which stores our height values as 16-bit unsigned integers, offering 65,535 height values for each pixel in the heightmap.

**We're not going to do this today**, but for the project we converted the heightmap into a 300x285 ENVI file using the following command. Just remember that we've stored the color values in the heightmap as numbers. 

```bash
$ gdal_translate -scale 600 1905 0 65535 -outsize 300 285 -ot UInt16 -of ENVI Gale_HRSC_DEM_50m.tif Gale_HRSC_DEM_50m.bin
```

If you'd like to read more about how to do this, Bjorn Sandvik has an [excellent explainer](http://blog.mastermaps.com/2013/10/terrain-building-with-threejs-part-1.html). 

You can see the files mentioned above in the 'data' folder.

## The fun part: Creating the planet
Now, we get to create the planet surface from the DEM data. To do this, we set the URL of the data to load and initialize the loader. Let's also initialize a value for the surface.

```javascript
// URL to our DEM resource
var terrainURL = "data/Gale_HRSC_DEM_50m_300x285.bin";

// Utility to load the DEM data
var terrainLoader = new THREE.TerrainLoader();

// We'll need this later
var surface;

// Create the plane geometry
var geometry = new THREE.PlaneGeometry(WORLD_WIDTH, WORLD_HEIGHT, 299, 284);
```

In the line above, we also create a [PlaneGeometry(width, height, widthSegments, heightSegments)](http://threejs.org/docs/#Reference/Extras.Geometries/PlaneGeometry). You'll see four arguments - these are the width and height of the "world" that we defined above, and the number of vertices the plane will have. We set these two values to 299 and 284 - the same dimensions as the DEM data, except that it's zero-indexed.

By the way, you won't see anything on the screen until we start actually rendering the scene. The screenshots in this tutorial are how it would look *if* it was rendering -- which we'll get to soon.

![](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/plain-plane-2.png "Wireframe of the plane loaded into the scene. Each vertex will be adjusted to fit the corresponding height value in the digital elevation model.")

Then we'll do something crazy. Remember how the height data in the DEM file was stored as a series of numbers? We can use the TerrainLoader to iterate over those values, and adjust each corresponding vertex in the plane. Thus we morph a flat plane to take the shape of the terrain in the data. Because at the scale of the scene we're making, the final shape would be pretty boring at its natural values, we exaggerate the height, settling in at a factor that feels comfortable.

```javascript
// The terrainLoader loads the DEM file and defines a function to be called when the file is successfully downloaded.
terrainLoader.load(terrainURL, function(data){

    // Adjust each vertex in the plane to correspond to the height value in the DEM file.
    for (var i = 0, l = geometry.vertices.length; i < l; i++) {
        geometry.vertices[i].z = data[i] / 65535 * 100;
    }

    // Leave this space blank for now we'll come back to it in the next section.

});
```



![](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/surface-wireframe.png?raw=true "Here's what the surface looks like after transforming the height to match the DEM data.")

So now we have a geometry for our surface, but we can't actually load this into the scene yet. Objects in Three.js, called "meshes", require both a geometry and a material. That is, you need to define a shape, and you need to define what that shape is made out of. Yes, it's a box, but what kind of box, is it? Cardboard? Metal? Is it painted?

Let's do this next.

Inside of the callback function, the line I said we'd be coming back to, we want to define a texture loader and set the URL, much in the way that we loaded the DEM data.

```javascript
    var textureLoader = new THREE.TextureLoader();
    var textureURL = "data/Gale_texture_high_4096px.jpg";
```

Then we load the texture similar to how we did the DEM file. This time, we define and set the material in the callback function, using the texture we loaded as a texture map.

```javascript
    // This goes inside the TerrainLoader callback function
    textureLoader.load(textureURL, function(texture) {
        // Lambert is a type non-reflective material
        var material = new THREE.MeshLambertMaterial({
            map: texture
        });

        // Leave this space blank, we're going to continue here in the next section

    });
```

That's fine and all, but what the heck is a texturemap? Why are we doing all this?

Remember the geometry is the shape of an object, and the material is what that object is made out of. Think of a texture map as a paint job, or "skin" on an object. It's basically an image mapped onto an object's surface.

![](http://www.trbimg.com/img-562bfce4/turbine/la-gale-crater-texture-20151024/600 "Color image of the Gale Crater created by a combination of images from the Viking spacecraft and the Mars Reconnaissance Orbiter. (NASA)")

You might notice we have both our geometry and material now, so it's time to add them to the scene, **inside of the textureLoader callback** (so, where the "Leave this space blank" comment is).

```javascript
        // This goes in the TextureLoader callback
        // Create the surface mesh and add it to the scene
        surface = new THREE.Mesh(geometry, material);
        scene.add(surface);
```

It's important to place these in the callbacks, otherwise you might be trying to load a texture onto a shape that hasn't loaded yet, or vice versa.

Now, let's take a look at the scene. There will be a wait while the resources load and process, but eventually you'll see this lovely scene.

![Our scene](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/oops.png?raw=true "Lambert materials won't show up in a scene without lights.")

What happened? We forgot to turn on the lights! Let's do that.

```javascript
// Lights!
var dirLight = new THREE.DirectionalLight( 0xffffff, 0.75);
dirLight.position.set( -1, 1, 1).normalize();

var ambiLight = new THREE.AmbientLight(0x999999);

// Add the lights to the scene
scene.add(ambiLight);
scene.add(dirLight);
```

That's better, right?

Just like a film scene, Three.js scenes need lighting. You can think of a [DirectionalLight(hexColor, intensity)](http://threejs.org/docs/#Reference/Lights/DirectionalLight) as a spotlight that you can specify the direction of and where it's pointing, while [AmbientLight(hexColor)](http://threejs.org/docs/#Reference/Lights/AmbientLight) is more like the sun - it lights all objects in the scene, regardless of where they're positioned.

Standing still in a scene isn't very fun, we can't even look around. To interact with a scene, we'll need to add controls. Controls basically move the camera around the scene according to user inputs. The different parameters we use below are specific to the FlyControls we'll be using. **Make sure to put this code before the `render()` loop:**

```javascript
// WASD-style movement controls
var controls = new THREE.FlyControls(camera);

// Disable automatic forward movement
controls.autoForward = false;

// Click and drag to look around with the mouse
controls.dragToLook = true;

// Movement and roll speeds, adjust these and see what happens!
controls.movementSpeed = 20;
controls.rollSpeed = Math.PI / 12;
```

Reload the page and... nothing happened! To actually move around in the scene, we also need to add and update the controls in the renderer loop. Find the `render()` function from earlier and replace the code inside it like so:

```javascript
    // Render loop
    function render() {

        // Get the difference from when the clock was last updated and update the controls based on that value.
        var delta = clock.getDelta();
        controls.update(delta);

        // Call the render function again
        requestAnimationFrame( render );

        // Update the scene through the manager.
        manager.render(scene, camera);
    }
```

Now when you reload the scene you can look around by clicking and dragging with the mouse, and use the WASD keys to move around the crater.

That's great and all but what if we want to try this on mobile? Instead, let's load controls conditionally, depending on the device we're on. This will load the VRControls if you load the page on your phone, which use the accelerometer to track movement.  

```javascript
// Detect mobile devices in the user agent
var is_mobile= /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);

// Conditionally load VR or Fly controls, based on whether we're on a mobile device
if (is_mobile) {
    var controls = new THREE.VRControls(camera);
} else {
    // WASD-style movement controls
    var controls = new THREE.FlyControls(camera);

    // Disable automatic forward movement
    controls.autoForward = false;

    // Click and drag to look around with the mouse
    controls.dragToLook = true;

    // Movement and roll speeds, adjust these and see what happens!
    controls.movementSpeed = 20;
    controls.rollSpeed = Math.PI / 12;
}
```

Now, if you visit this page on a mobile phone, you'll be able to look around the scene just by moving your device.

## VR Mode

One great thing about the VR manager is it provides an interface to allow users to easily switch in and out of VR mode, so that you can use a cardboard or other VR device. This ability is automatically loaded based on the device that you're viewing the page on, so it'll show up on phones, but not your desktops. We can override this though.

At the very top of the page you'll see a line ([line 20, in fact.](https://github.com/datadesk/vr-interactives-three-js/blob/master/three-demo.html#L20)) that says `FORCE_ENABLE_VR: false`. Change this option to `true`.

![](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/cardboard-icon.png?raw=true "VR mode toggle icon.")

Now, if you reload the page, you'll see a small cardboard icon in the lower right-hand side of the page. Click this, and you'll see the page distort as if you were viewing the page through a cardboard.

Voila!

![](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/vr-mode.png?raw=true "Viewing the scene distorted through VR mode.")


## Other Things you may want to do
- Load in different surface detail and texture sizes based on the device.
- Add in collision detection
- Call out specific points in the crater
- Progressively load terrain and texture


## So what can you do with this?
A couple people have taken the techniques outlined here and integrated them into story pages
- [Yaryna Serkez](https://twitter.com/iarynam) created a [3D map of the Carpathian Mountains](http://texty.org.ua/d/carpathians-3d/).
- This isn't a three.js project, but Joe Fox used the same terrain extrusion techniques to create a map of a [mountain lion's movements in Griffith Park](http://www.latimes.com/projects/la-me-griffith-park-mountain-lion/).
