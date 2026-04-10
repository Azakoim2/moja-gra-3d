<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Moja Gra Wyścigowa 3D</title>
    <style>
        body { 
            margin: 0; 
            overflow: hidden; /* Ukrywa paski przewijania */
            background-color: #87CEEB; /* Kolor nieba */
        }
        canvas { 
            display: block; 
        }
        #instrukcja { 
            position: absolute; 
            top: 20px; 
            width: 100%; 
            text-align: center; 
            color: white; 
            font-family: sans-serif; 
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
            pointer-events: none; 
        }
    </style>
</head>
<body>
    <div id="instrukcja">Sterowanie: Strzałki lub W, A, S, D</div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // 1. Ustawienie Sceny, Kamery i Renderera
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // 2. Dodanie Światła
        const light = new THREE.DirectionalLight(0xffffff, 1);
        light.position.set(10, 20, 10);
        scene.add(light);
        scene.add(new THREE.AmbientLight(0x404040)); // Rozproszone światło

        // 3. Tworzenie "Samochodu" (niebieski prostopadłościan)
        const carGeometry = new THREE.BoxGeometry(2, 1, 4);
        const carMaterial = new THREE.MeshPhongMaterial({ color: 0x0000ff });
        const car = new THREE.Mesh(carGeometry, carMaterial);
        car.position.y = 0.5; // Uniesienie nad ziemię
        scene.add(car);

        // 4. Tworzenie Toru (szara płaszczyzna)
        const groundGeometry = new THREE.PlaneGeometry(200, 200);
        const groundMaterial = new THREE.MeshPhongMaterial({ color: 0x555555 });
        const ground = new THREE.Mesh(groundGeometry, groundMaterial);
        ground.rotation.x = -Math.PI / 2; // Położenie płasko
        scene.add(ground);

        // 5. Logika Sterowania
        const keys = { w: false, a: false, s: false, d: false, ArrowUp: false, ArrowDown: false, ArrowLeft: false, ArrowRight: false };
        let speed = 0;
        const maxSpeed = 0.8;
        const acceleration = 0.02;
        const steering = 0.04;

        window.addEventListener('keydown', (e) => keys[e.key] = true);
        window.addEventListener('keyup', (e) => keys[e.key] = false);

        // Przesunięcie kamery względem samochodu (widok z trzeciej osoby)
        const cameraOffset = new THREE.Vector3(0, 5, -10);

        // 6. Główna Pętla Gry
        function animate() {
            requestAnimationFrame(animate);

            // Przyspieszanie i hamowanie
            if (keys.ArrowUp || keys.w) speed += acceleration;
            else if (keys.ArrowDown || keys.s) speed -= acceleration;
            else speed *= 0.95; // Naturalne zwalnianie (tarcie)

            // Ograniczenie maksymalnej prędkości
            speed = Math.max(-maxSpeed / 2, Math.min(maxSpeed, speed));

            // Skręcanie (tylko gdy samochód jedzie)
            if (Math.abs(speed) > 0.01) {
                if (keys.ArrowLeft || keys.a) car.rotation.y += steering * Math.sign(speed);
                if (keys.ArrowRight || keys.d) car.rotation.y -= steering * Math.sign(speed);
            }

            // Ruch samochodu do przodu/tyłu
            car.translateZ(speed);

            // Płynne podążanie kamery za samochodem
            const relativeCameraOffset = cameraOffset.clone().applyMatrix4(car.matrixWorld);
            camera.position.lerp(relativeCameraOffset, 0.1);
            camera.lookAt(car.position);

            renderer.render(scene, camera);
        }

        animate(); // Uruchomienie pętli

        // Responsywność okna
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
