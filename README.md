# JUEGO
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Pasillo Interactivo</title>
<style>
  body { margin: 0; overflow: hidden; }
  canvas { display: block; }
  #infoPanel {
    position: absolute;
    top: 20px;
    left: 20px;
    background: rgba(255,255,255,0.9);
    padding: 15px;
    border-radius: 8px;
    display: none;
    font-family: Arial, sans-serif;
  }
  #infoPanel h2 { margin-top: 0; }
  #openBtn { margin-top: 10px; padding: 5px 10px; }
</style>
</head>
<body>
<div id="infoPanel">
  <h2 id="roomTitle"></h2>
  <p id="roomDesc"></p>
  <button id="openBtn">Cerrar y Avanzar</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/controls/OrbitControls.js"></script>

<script>
let scene, camera, renderer, controls;
let doors = [];
let currentDoor = 0;
let moving = false;

// Datos de habitaciones
const rooms = [
  {title: "Videojuegos", desc: "Bienvenido a la habitación de videojuegos"},
  {title: "Básquet", desc: "Aquí hablamos de básquet"},
  {title: "Animales: Perros", desc: "¡Esta habitación es sobre perros!"},
  {title: "Videojuegos Extra", desc: "Otra sección de videojuegos"},
  {title: "Animales Extra", desc: "Otra sección de animales"}
];

// Inicializar escena
init();
animate();

function init() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xaaaaaa);

  camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  camera.position.set(0, 1.6, 5);

  renderer = new THREE.WebGLRenderer({antialias: true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enablePan = false;
  controls.enableZoom = false;
  controls.minDistance = 1;
  controls.maxDistance = 10;

  // Piso y techo
  const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 3),
    new THREE.MeshStandardMaterial({color: 0x808080})
  );
  floor.rotation.x = -Math.PI/2;
  floor.position.y = 0;
  scene.add(floor);

  const ceiling = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 3),
    new THREE.MeshStandardMaterial({color: 0x999999})
  );
  ceiling.rotation.x = Math.PI/2;
  ceiling.position.y = 3;
  scene.add(ceiling);

  // Paredes
  const wallMat = new THREE.MeshStandardMaterial({color: 0xffffff});
  const wallLeft = new THREE.Mesh(new THREE.PlaneGeometry(20,3), wallMat);
  wallLeft.position.set(0,1.5,-1.5);
  wallLeft.rotation.y = 0;
  scene.add(wallLeft);

  const wallRight = new THREE.Mesh(new THREE.PlaneGeometry(20,3), wallMat);
  wallRight.position.set(0,1.5,1.5);
  wallRight.rotation.y = Math.PI;
  scene.add(wallRight);

  // Puertas
  for(let i=0; i<5; i++){
    const door = new THREE.Mesh(
      new THREE.BoxGeometry(1,2,0.2),
      new THREE.MeshStandardMaterial({color: 0x3333ff})
    );
    door.position.set(i*4 - 8, 1, 0);
    scene.add(door);
    doors.push(door);
  }

  // Luz
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(0,5,5);
  scene.add(light);

  window.addEventListener('resize', onWindowResize, false);
  renderer.domElement.addEventListener('click', onClick, false);

  document.getElementById('openBtn').addEventListener('click', nextDoor);
}

function onWindowResize() {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

// Función para detectar clic en puertas
function onClick(event){
  if(moving) return;

  const mouse = new THREE.Vector2();
  mouse.x = (event.clientX/window.innerWidth)*2-1;
  mouse.y = -(event.clientY/window.innerHeight)*2+1;

  const raycaster = new THREE.Raycaster();
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(doors);

  if(intersects.length > 0 && intersects[0].object === doors[currentDoor]){
    openDoor(currentDoor);
  }
}

// Abrir puerta
function openDoor(index){
  document.getElementById('roomTitle').innerText = rooms[index].title;
  document.getElementById('roomDesc').innerText = rooms[index].desc;
  document.getElementById('infoPanel').style.display = 'block';
}

// Avanzar a la siguiente puerta
function nextDoor(){
  document.getElementById('infoPanel').style.display = 'none';
  if(currentDoor < doors.length -1){
    moving = true;
    const targetPos = doors[currentDoor+1].position.clone();
    const startPos = camera.position.clone();
    let elapsed = 0;
    const duration = 25; // segundos

    function move(delta){
      elapsed += delta;
      const t = Math.min(elapsed/duration,1);
      camera.position.lerpVectors(startPos, targetPos.clone().add(new THREE.Vector3(0,0,5)), t);
      if(t < 1){
        requestAnimationFrame(()=>move(0.016));
      } else {
        moving = false;
        currentDoor++;
      }
    }
    move(0.016);
  }
}

function animate(){
  requestAnimationFrame(animate);
  renderer.render(scene,camera);
}
</script>
</body>
</html>
