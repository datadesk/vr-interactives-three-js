# vr-interactives-three-js
There are a lot of Three.js [examples](http://threejs.org/examples/) and [tutorials](http://learningthreejs.com/) out there, but very few examples of what you can do with real-world data.

In this session you'll learn how we used NASA satellite imagery and elevation data to create a 3-D rendering of the [Gale Crater](http://graphics.latimes.com/mars-gale-crater-vr/) on Mars.

## Requirements
We'll make use of the [WebVR Boilerplate](https://github.com/borismus/webvr-boilerplate), which also uses the [WebVR polyfill](https://github.com/borismus/webvr-polyfill) to provide VR support when the WebVR spec isn't implemented.

We'll also use a [terrain loader](http://blog.thematicmapping.org/2013/10/terrain-building-with-threejs.html), developed by [Bjørn Sandvik](http://blog.thematicmapping.org/).

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

We also want to position the camera, and tell it where to look.

```javascript
camera.position.set(0, -199, 75);
camera.up = new THREE.Vector3(0,0,1);
camera.lookAt(scene.position);
```

Now let's create the renderer. We're going to be creating a [WebGL Renderer](http://threejs.org/docs/#Reference/Renderers/WebGLRenderer), which uses the WebGL API to render our graphics. It has way better performance than the [Canvas Renderer](http://threejs.org/docs/#Reference/Renderers/CanvasRenderer) and you'll want to use it whenever possible.

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

## Prepping the DEM data
![Digital elevation model of Gale Crater. Brighter values are higher elevations. (NASA)](http://www.trbimg.com/img-562bfe79/turbine/la-mars-dem-map-20151024/600 "Digital elevation model of Gale Crater. Brighter values are higher elevations. (NASA)")

A digital elevation model is a 3D representation of a terrain's surface, and in this case is a greyscale heightmap, where lighter colors represent higher elevations.

The DEM data came as a GeoTIFF file (Gale_HRSC_DEM_50m_overlap.tif). Unfortunately, the TIFF file is huge, and TIFF isn't supported by most browsers anyway. We could use the incredibly useful [GDAL](http://www.gdal.org/) to convert it to a PNG image, where the height values are reduced to only 256 shades of grey. This would make our terrain blocky, however.

We can, however, convert it to a format called ENVI, which can store our height values as 16-bit unsigned integers, offering 65,535 height values for each pixel in the heightmap.

We're not going to do this today, but we converted the heightmap into a 300x285 ENVI file using the following command. Just remember that we've stored the color values in the heightmap as numbers.

```bash
$ gdal_translate -scale 600 1905 0 65535 -outsize 300 285 -ot UInt16 -of ENVI Gale_HRSC_DEM_50m.tif Gale_HRSC_DEM_50m.bin
```

## The fun part: Creating the planet
Now, we get to create the planet surface from the DEM data. To do this, we set the URL of the data to load and initialize the loader. Let's also initialize a value for the surface.

```javascript
var terrainURL = "data/Gale_HRSC_DEM_50m_300x285.bin";
var terrainLoader = new THREE.TerrainLoader();
var surface;
```

Then load in the data using the loader. In the callback, once the data file is loaded, we're first going to create a [plane](http://threejs.org/docs/#Reference/Extras.Geometries/PlaneGeometry). You'll see four arguments - these are the width and height of the "world" that we defined above, and the number of vertices the plane will have. We set these two values to 299 and 284 - the same dimensions as the DEM data, except that it's zero-indexed.

Then we'll do something crazy. Remember how the height data in the DEM file was stored as a series of numbers? We can use the TerrainLoader to iterate over those values, and adjust each corresponding vertex in the plane. Thus we morph a flat plane to take the shape of the terrain in the data. Because at the scale of the scene we're making, the final shape would be pretty boring at its natural values, we exaggerate the height, settling in at a factor that feels comfortable.


```javascript
// The terrainLoader loads the DEM file and defines a function to be called when the file is successfully downloaded.
terrainLoader.load(terrainURL, function(data){
    // Create the plane geometry
    var geometry = new THREE.PlaneGeometry(WORLD_WIDTH, WORLD_HEIGHT, 299, 284);

    // Adjust each vertex in the plane to correspond to the height value in the DEM file.
    for (var i = 0, l = geometry.vertices.length; i < l; i++) {
        geometry.vertices[i].z = data[i] / 65535 * 100;
    }

    // Leave this space blank for now we'll come back to it in the next section.

});
```

So now we have a geometry for our surface, but we can't actually load this into the scene yet. Objects in Three.js, called "meshes", require both a geometry and a material. That is, you need to define a shape, and you need to define what that shape is made out of. Yes, it's a box, but what kind of box, is it? Cardboard? Metal? Is it painted?

Let's do this next.

Inside of the callback function, the line I said we'd be coming back to, we want to define a texture loader and set the URL, much in the way that we loaded the DEM data.

```javascript
    var textureLoader = new THREE.TextureLoader()
    var textureURL = "data/Gale_texture_high_4096px.jpg";
```

Then we load the texture similar to how we did the DEM file. This time, we define and set the material in the callback function, using the texture we loaded as a texture map.

```javascript
    textureLoader.load(textureURL, function(texture) {
        var material = new THREE.MeshLambertMaterial({
            color: 0xffffff,
            map: texture
        });

        // Leave this space blank, we're going to continue here in the next section

    });
```

That's fine and all, but what the heck is a texturemap? Why are we doing all this?

Remember the geometry is the shape of an object, and the material is what that object is made out of? Well think of a texture map as a paint job, or "skin" on an object. It's basically an image mapped onto an object's surface.

You might notice we have both our geometry and material now, so it's time to add them to the scene, inside of the textureLoader callback.

```javascript
        surface = new THREE.Mesh(geometry, material);
        scene.add(surface);
```

It's important to place these in the callbacks, otherwise you might be trying to load a texture onto a shape that hasn't loaded yet, or vice versa.

So we have a scene, a camera and objects in that scene. Let's render it! We do this by creating a rendering loop. Right now, since we don't have controls defined yet, this is very simple.

```javascript

// Render loop
function render() {
    // Render the scene through the manager.
    requestAnimationFrame( render );
    manager.render(scene, camera);
}

render();

```

Now load the scene. There will be a wait while the resources load and process, but eventually you'll see this lovely scene.

![Our scene](https://github.com/datadesk/vr-interactives-three-js/blob/master/img/oops.png?raw=true)

What happened? We forgot to turn on the lights!

```javascript
// Lights!
var dirLight = new THREE.DirectionalLight( 0xffffff, 0.75);
dirLight.position.set( -1, 1, 1).normalize();

var ambiLight = new THREE.AmbientLight(0x999999);

scene.add(ambiLight);
scene.add(dirLight);
```


## Other Things you may want to do
- Load in different surface detail and texture sizes based on the device.
- Add in collision detection