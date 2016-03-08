<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Building VR interactives with Three.js</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes" />
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
    <style>
    html, body { margin:0; padding:0; overflow:hidden;}
    </style>
</head>
<body>
    <script>
    WebVRConfig = {
      // Forces availability of VR mode.
      FORCE_ENABLE_VR: true,
    };
    </script>
    <!-- Assets we'll need - Three.js, controls, and the webVR manager and polyfill -->
    <script src="js/es6-promise.min.js"></script>
    <script src="js/three.min.js"></script>
    <script src="js/three.flycontrols.js"></script>
    <script src="js/three.terrainloader.js"></script>
    <script src="js/VRControls.js"></script>
    <script src="js/VREffect.js"></script>
    <script src="js/webvr-polyfill.js"></script>
    <script src="js/webvr-manager.js"></script>

    <script>
    // Width and height of the browser window
    var WINDOW_WIDTH = window.innerWidth;
    var WINDOW_HEIGHT = window.innerHeight;

    // Width and height of the surface we're going to create
    var WORLD_WIDTH = 2000;
    var WORLD_HEIGHT = 1900;

    // Where our lights and cameras will go
    var scene = new THREE.Scene();

    // Keeps track of time
    var clock = new THREE.Clock();

    // How we will see the scene
    var camera = new THREE.PerspectiveCamera(75, WINDOW_WIDTH / WINDOW_HEIGHT, 1, 5000);

    // Position the camera slightly above and in front of the scene
    camera.position.set(0, -199, 75);
    camera.up = new THREE.Vector3(0,0,1);

    // Look at the center of the scene
    camera.lookAt(scene.position);

    // Think of the renderer as the engine that drives the scene
    var renderer = new THREE.WebGLRenderer({antialias: true});

    // Set the pixel ratio of the screen (for high DPI screens)
    renderer.setPixelRatio(window.devicePixelRatio);

    // Set the background of the scene to a orange/red
    renderer.setClearColor(0xffd4a6);

    // Set renderer to the size of the window
    renderer.setSize(WINDOW_WIDTH, WINDOW_HEIGHT);

    // Append the renderer to the DOM
    document.body.appendChild( renderer.domElement );

    // Apply VR stereo rendering to renderer
    var effect = new THREE.VREffect(renderer);
    effect.setSize(WINDOW_WIDTH, WINDOW_HEIGHT);

    var manager = new WebVRManager(renderer, effect);

    // URL to our DEM resource
    var terrainURL = "data/Gale_HRSC_DEM_50m_300x285.bin";

    // Utility to load the DEM data
    var terrainLoader = new THREE.TerrainLoader();

    // We'll need this later
    var surface;

    // Create the plane geometry
    var geometry = new THREE.PlaneGeometry(WORLD_WIDTH, WORLD_HEIGHT, 299, 284);

    // The terrainLoader loads the DEM file and defines a function to be called when the file is successfully downloaded.
    terrainLoader.load(terrainURL, function(data){

        // Adjust each vertex in the plane to correspond to the height value in the DEM file.
        for (var i = 0, l = geometry.vertices.length; i < l; i++) {
            geometry.vertices[i].z = data[i] / 65535 * 100;
        }

        var textureLoader = new THREE.TextureLoader();
        var textureURL = "data/Gale_texture_high_4096px.jpg";

        // This goes inside the TerrainLoader callback function
        textureLoader.load(textureURL, function(texture) {
            var material = new THREE.MeshLambertMaterial({
                map: texture,
            });

            // This goes in the TextureLoader callback
            // Create the surface mesh and add it to the scene
            surface = new THREE.Mesh(geometry, material);
            scene.add(surface);
        });
    });

    // Lights!
    var dirLight = new THREE.DirectionalLight( 0xffffff, 0.75);
    dirLight.position.set( -1, 1, 1).normalize();

    var ambiLight = new THREE.AmbientLight(0x999999);

    // Add the lights to the scene
    scene.add(ambiLight);
    scene.add(dirLight);

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

    // Render loop
    // This should go at the bottom of the script.
    function render() {

        // Get the difference from when the clock was last updated and update the controls based on that value.
        var delta = clock.getDelta();
        controls.update(delta);

        // Update the scene through the manager.
        manager.render(scene, camera);

        // Call the render function again
        requestAnimationFrame( render );

    }

    render();

    </script>

</body>
</html>