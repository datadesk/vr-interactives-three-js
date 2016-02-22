# vr-interactives-three-js
There are a lot of Three.js [examples](http://threejs.org/examples/) and [tutorials](http://learningthreejs.com/) out there, but very few examples of what you can do with real-world data. 

In this session you'll learn how we used NASA satellite imagery and elevation data to create a 3-D rendering of the [Gale Crater](http://graphics.latimes.com/mars-gale-crater-vr/) on Mars.  

## Requirements
We'll make use of the [WebVR Boilerplate](https://github.com/borismus/webvr-boilerplate), which also uses the [WebVR polyfill](https://github.com/borismus/webvr-polyfill) to provide VR support when the WebVR spec isn't implemented.  

We'll also use a [terrain loader](http://blog.thematicmapping.org/2013/10/terrain-building-with-threejs.html), developed by Bjørn Sandvik. 

## Let's get started
Using Three.js can be compared a bit to filmmaking: you have a scene, lighting and a camera. The scene updates a certain number of times per second, otherwise known as "frames per second" (which we'll try to keep as close to 60 as we can, but will drop based on your computer and the complexity of the scene.)

We'll build this in the file [three-demo.html](three-demo.html).