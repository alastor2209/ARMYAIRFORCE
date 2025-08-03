// script.js
import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
import { getFirestore, doc, setDoc, onSnapshot, collection, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
import * as THREE from "https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js";

// Global variables provided by the environment
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

// Firebase instances
let app, db, auth;
let user;
let isAuthReady = false;

// Pre-defined units and colors
const unitsAndColors = {
    'หน่วยรบพิเศษ': '#ff0000',
    'หน่วยลาดตระเวน': '#00ff00',
    'หน่วยสนับสนุน': '#0000ff',
    'หน่วยข่าวกรอง': '#ffff00',
};
const ranks = ['พลทหาร', 'จ่าสิบตรี', 'ร้อยเอก', 'พันเอก'];
let userRank = 'พลทหาร';
let userUnit = 'หน่วยรบพิเศษ';
let userColor = unitsAndColors[userUnit];

// THREE.js instances
let scene, camera, renderer, terrainMesh;

// UI Elements
const loadingScreen = document.getElementById('loading-screen');
const appContainer = document.getElementById('app-container');
const canvas = document.getElementById('map-canvas');
const userIdDisplay = document.getElementById('user-id');
const rankSelect = document.getElementById('user-rank-select');
const unitSelect = document.getElementById('user-unit-select');
const markerColorDisplay = document.getElementById('marker-color-display');
const markerList = document.getElementById('marker-list');
const refreshButton = document.getElementById('refresh-button');
const alertBox = document.getElementById('alert-box');

// Helper function to show custom alerts
function showAlert(message) {
    alertBox.classList.remove('hidden');
    alertBox.querySelector('p').textContent = message;
}

// Initialize Firebase and THREE.js
window.onload = async () => {
    // Populate dropdowns
    ranks.forEach(rank => {
        const option = document.createElement('option');
        option.value = rank;
        option.textContent = rank;
        rankSelect.appendChild(option);
    });
    Object.keys(unitsAndColors).forEach(unit => {
        const option = document.createElement('option');
        option.value = unit;
        option.textContent = unit;
        unitSelect.appendChild(option);
    });
    unitSelect.value = userUnit;
    markerColorDisplay.style.backgroundColor = userColor;

    // --- Firebase Initialization ---
    app = initializeApp(firebaseConfig);
    db = getFirestore(app);
    auth = getAuth(app);

    try {
        if (initialAuthToken) {
            await signInWithCustomToken(auth, initialAuthToken);
        } else {
            await signInAnonymously(auth);
        }

        onAuthStateChanged(auth, async (currentUser) => {
            if (currentUser) {
                user = currentUser;
                isAuthReady = true;
                userIdDisplay.textContent = user.uid;

                // Fetch or create user data
                const userDocRef = doc(db, 'artifacts', appId, 'users', user.uid);
                const docSnap = await getDoc(userDocRef);
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    userRank = data.rank || 'พลทหาร';
                    userUnit = data.unit || 'หน่วยรบพิเศษ';
                    userColor = data.color || unitsAndColors[userUnit];
                } else {
                    await setDoc(userDocRef, {
                        rank: userRank,
                        unit: userUnit,
                        userId: user.uid,
                        color: userColor
                    });
                }
                rankSelect.value = userRank;
                unitSelect.value = userUnit;
                markerColorDisplay.style.backgroundColor = userColor;

                // Set up real-time marker listener after auth is ready
                setupMarkerListener();
                // Hide loading screen and show app
                loadingScreen.classList.add('hidden');
                appContainer.classList.remove('hidden');
            } else {
                console.error('Firebase Auth state is not ready.');
            }
        });
    } catch (error) {
        console.error('Error during Firebase initialization or sign-in:', error);
        showAlert('เกิดข้อผิดพลาดในการเชื่อมต่อ กรุณาลองใหม่อีกครั้ง');
    }

    // --- THREE.js Scene Setup ---
    scene = new THREE.Scene();
    scene.background = new THREE.Color('#1f2937');
    camera = new THREE.PerspectiveCamera(75, canvas.offsetWidth / canvas.offsetHeight, 0.1, 1000);
    renderer = new THREE.WebGLRenderer({ antialias: true, canvas: canvas });
    renderer.setSize(canvas.offsetWidth, canvas.offsetHeight);
    renderer.shadowMap.enabled = true;

    // Lighting
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);
    const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight.position.set(50, 50, 50);
    directionalLight.castShadow = true;
    scene.add(directionalLight);

    // Terrain placeholder
    const terrainGeometry = new THREE.PlaneGeometry(100, 100, 50, 50);
    const terrainMaterial = new THREE.MeshPhongMaterial({ color: '#34d399', side: THREE.DoubleSide });
    terrainMesh = new THREE.Mesh(terrainGeometry, terrainMaterial);
    terrainMesh.rotation.x = -Math.PI / 2;
    terrainMesh.receiveShadow = true;
    scene.add(terrainMesh);

    // Camera position
    camera.position.set(0, 50, 50);
    camera.lookAt(0, 0, 0);

    // Resize handler
    window.addEventListener('resize', () => {
        camera.aspect = canvas.offsetWidth / canvas.offsetHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(canvas.offsetWidth, canvas.offsetHeight);
    });

    // Start animation loop
    const animate = () => {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    };
    animate();
};

// Function to set up the Firestore real-time listener
function setupMarkerListener() {
    const markersCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'markers');
    onSnapshot(markersCollectionRef, (snapshot) => {
        markerList.innerHTML = ''; // Clear old list items
        
        // Remove old markers from scene
        scene.children = scene.children.filter(obj => obj.userData.isMarker !== true);

        // Add new markers to scene and UI
        snapshot.docs.forEach(doc => {
            const markerData = doc.data();
            const markerGeometry = new THREE.SphereGeometry(1, 32, 32);
            const markerMaterial = new THREE.MeshPhongMaterial({ color: markerData.color });
            const markerMesh = new THREE.Mesh(markerGeometry, markerMaterial);
            markerMesh.position.set(markerData.x, 0.5, markerData.z);
            markerMesh.userData.isMarker = true;
            scene.add(markerMesh);

            // Add to UI list
            const listItem = document.createElement('li');
            listItem.className = 'flex items-center space-x-2 p-2 rounded-md bg-gray-800';
            listItem.innerHTML = `
                <div class="w-4 h-4 rounded-full" style="background-color: ${markerData.color};"></div>
                <div class="flex-1">
                    <p class="text-sm font-medium">${markerData.unit} - ${markerData.rank}</p>
                    <p class="text-xs text-gray-400">พิกัด: X:${markerData.x.toFixed(2)}, Z:${markerData.z.toFixed(2)}</p>
                </div>
            `;
            markerList.appendChild(listItem);
        });
    });
}

// Event listener for canvas clicks to add a marker
canvas.addEventListener('mousedown', async (event) => {
    if (!user) {
        showAlert('โปรดลงชื่อเข้าใช้ก่อนทำการมาร์คจุด');
        return;
    }

    const mouse = new THREE.Vector2();
    const rect = canvas.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(mouse, camera);

    const intersects = raycaster.intersectObjects(scene.children, true);

    if (intersects.length > 0) {
        const intersection = intersects[0];
        const position = intersection.point;

        try {
            const markersCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'markers');
            await setDoc(doc(markersCollectionRef), {
                x: position.x,
                y: 0.5,
                z: position.z,
                userId: user.uid,
                unit: userUnit,
                rank: userRank,
                color: userColor,
                timestamp: new Date().toISOString()
            });
        } catch (error) {
            console.error('Error adding marker to Firestore:', error);
        }
    }
});

// Event listener for unit selection change
unitSelect.addEventListener('change', async (e) => {
    const newUnit = e.target.value;
    const newColor = unitsAndColors[newUnit];
    userUnit = newUnit;
    userColor = newColor;
    markerColorDisplay.style.backgroundColor = newColor;
    if (user && db) {
        try {
            const userDocRef = doc(db, 'artifacts', appId, 'users', user.uid);
            await setDoc(userDocRef, { rank: userRank, unit: newUnit, color: newColor, userId: user.uid });
        } catch (error) {
            console.error('Error updating user unit:', error);
        }
    }
});

// Event listener for rank selection change
rankSelect.addEventListener('change', async (e) => {
    const newRank = e.target.value;
    userRank = newRank;
    if (user && db) {
        try {
            const userDocRef = doc(db, 'artifacts', appId, 'users', user.uid);
            await setDoc(userDocRef, { rank: newRank, unit: userUnit, color: userColor, userId: user.uid });
        } catch (error) {
            console.error('Error updating user rank:', error);
        }
    }
});

// Event listener for refresh button
refreshButton.addEventListener('click', () => {
    // Firestore's onSnapshot listener handles real-time updates automatically,
    // so a manual refresh button isn't strictly necessary for new markers.
    // However, it can be useful for debugging or re-syncing if needed.
    console.log('Refreshing map manually...');
    setupMarkerListener();
});
