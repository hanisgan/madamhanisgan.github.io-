<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chubby Face Tilt - Cheerful Cartoon Quiz</title>
    <!-- Tailwind CSS for sleek Dashboard layout -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Dashboard icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Phaser 3 for smooth, bouncy cartoon graphics -->
    <script src="https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.min.js"></script>
    <!-- MediaPipe Face Mesh & Camera tools -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js" crossorigin="anonymous"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fredoka+One&family=Quicksand:wght@500;700;900&display=swap');
        
        body {
            font-family: 'Quicksand', sans-serif;
            background: radial-gradient(circle, #fef9c3 0%, #e0f2fe 100%);
        }
        .cartoon-font {
            font-family: 'Fredoka One', cursive;
        }
        /* Custom scrollbar styling for question list */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
            border-radius: 99px;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 99px;
            border: 2px solid #f1f5f9;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
    </style>
</head>
<body class="min-h-screen text-slate-800 flex flex-col overflow-x-hidden">

    <!-- Top Navigation Bar -->
    <header class="bg-amber-400 border-b-8 border-amber-500 py-4 px-6 flex justify-between items-center shadow-lg z-10">
        <div class="flex items-center space-x-3">
            <div class="bg-white p-2 rounded-2xl text-amber-500 animate-bounce shadow-md border-2 border-amber-500">
                <i class="fa-solid fa-face-laugh-wink text-3xl"></i>
            </div>
            <div>
                <h1 class="text-2xl md:text-3xl cartoon-font tracking-wide text-slate-900 drop-shadow-[0_2px_0_rgba(255,255,255,1)]">TILT-O-RAMA!</h1>
                <p class="text-xs text-slate-700 font-extrabold uppercase tracking-wider">Face-Tracking Cartoon Party Game</p>
            </div>
        </div>
        <div class="flex items-center space-x-3 text-xs md:text-sm font-black bg-white/95 px-5 py-2.5 rounded-full border-4 border-amber-500 shadow-md text-slate-800">
            <span class="flex h-3.5 w-3.5 relative">
                <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
                <span class="relative inline-flex rounded-full h-3.5 w-3.5 bg-emerald-500"></span>
            </span>
            <span id="tracking-status" class="text-emerald-600">Simulator Active</span>
        </div>
    </header>

    <!-- Main Workspace Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 grid grid-cols-1 lg:grid-cols-12 gap-6 items-stretch">
        
        <!-- Left Side: Interactive Game and Virtual Camera Debug Panel (8 cols) -->
        <div class="lg:col-span-8 flex flex-col space-y-6">
            
            <!-- Phaser Game Canvas Mounting Point -->
            <div class="bg-white p-5 rounded-[2.5rem] border-8 border-amber-300 shadow-xl relative overflow-hidden flex flex-col items-center justify-center">
                <div class="absolute top-3 left-6 text-xs font-black text-amber-600 cartoon-font uppercase tracking-widest bg-amber-100 px-3 py-1 rounded-full border-2 border-amber-300">Game Viewport</div>
                <div id="game-container" class="w-full max-w-full rounded-[1.5rem] overflow-hidden shadow-inner border-4 border-slate-700 mt-6"></div>
            </div>

            <!-- Simulator & Debug Control Board -->
            <div class="bg-white p-6 rounded-3xl border-4 border-sky-300 shadow-lg grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                    <h3 class="text-base font-black text-sky-600 cartoon-font uppercase tracking-wider mb-2"><i class="fa-solid fa-gamepad mr-2 text-xl"></i>Virtual Simulator Controls</h3>
                    <p class="text-xs text-slate-500 font-bold mb-4 leading-relaxed">No Webcam? Test right away! Push left/right buttons to mock physical face rotations and trigger answers:</p>
                    <div class="flex space-x-3">
                        <button id="sim-left-btn" class="flex-1 bg-rose-500 hover:bg-rose-400 font-black text-white py-3 px-4 rounded-2xl shadow-md border-b-8 border-rose-700 active:border-b-0 active:translate-y-2 transition-all flex items-center justify-center space-x-2 text-sm uppercase">
                            <i class="fa-solid fa-arrow-left"></i><span>Tilt Left</span>
                        </button>
                        <button id="sim-center-btn" class="bg-slate-200 hover:bg-slate-300 font-black text-slate-700 py-3 px-4 rounded-2xl shadow-md border-b-8 border-slate-400 active:border-b-0 active:translate-y-2 transition-all text-sm">
                            Reset
                        </button>
                        <button id="sim-right-btn" class="flex-1 bg-emerald-500 hover:bg-emerald-400 font-black text-white py-3 px-4 rounded-2xl shadow-md border-b-8 border-emerald-700 active:border-b-0 active:translate-y-2 transition-all flex items-center justify-center space-x-2 text-sm uppercase">
                            <span>Tilt Right</span><i class="fa-solid fa-arrow-right"></i>
                        </button>
                    </div>
                </div>
                
                <div class="flex flex-col justify-between">
                    <div>
                        <h3 class="text-base font-black text-amber-500 cartoon-font uppercase tracking-wider mb-2"><i class="fa-solid fa-sliders mr-2 text-xl"></i>Calibration & Hold Speeds</h3>
                        <div class="space-y-4">
                            <div>
                                <div class="flex justify-between items-center text-xs font-bold text-slate-600 mb-1">
                                    <span>Head Tilt Angle Threshold</span>
                                    <span class="text-amber-500 font-mono font-black text-sm" id="threshold-val">15°</span>
                                </div>
                                <input id="threshold-slider" type="range" min="5" max="35" value="15" class="w-full accent-amber-400 bg-slate-100 rounded-lg appearance-none h-3 cursor-pointer border border-slate-200">
                            </div>
                            
                            <div>
                                <div class="flex justify-between items-center text-xs font-bold text-slate-600 mb-1">
                                    <span>Answer Hold-Time Lock Delay</span>
                                    <span class="text-sky-500 font-mono font-black text-sm" id="delay-val">1.5s</span>
                                </div>
                                <input id="delay-slider" type="range" min="5" max="30" value="15" class="w-full accent-sky-400 bg-slate-100 rounded-lg appearance-none h-3 cursor-pointer border border-slate-200">
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Right Side: Webcam Feed Overlay & Question Customizer (4 cols) -->
        <div class="lg:col-span-4 flex flex-col space-y-6">
            
            <!-- Real Camera tracking panel -->
            <div class="bg-white p-5 rounded-3xl border-4 border-sky-300 shadow-lg flex flex-col items-center relative">
                <div class="w-full flex justify-between items-center mb-3">
                    <h3 class="text-sm font-black text-sky-600 cartoon-font uppercase tracking-wider"><i class="fa-solid fa-camera mr-2 text-lg"></i>Webcam Head Tracking</h3>
                    <button id="toggle-camera-btn" class="bg-emerald-500 hover:bg-emerald-400 text-white text-xs px-4 py-2 rounded-xl font-black shadow-md border-b-4 border-emerald-700 active:border-b-0 active:translate-y-1 transition-all">
                        <i class="fa-solid fa-power-off mr-1.5"></i>Start Feed
                    </button>
                </div>
                
                <!-- Display Canvas with Canvas Mesh Overlays -->
                <div class="relative w-full aspect-[4/3] bg-amber-50 rounded-2xl overflow-hidden border-4 border-sky-200 flex items-center justify-center shadow-inner">
                    <!-- Hidden real raw input element -->
                    <video id="webcam-input" class="hidden" autoplay playsinline></video>
                    <!-- Output canvas drawing frame overlays & wireframe meshes -->
                    <canvas id="mesh-overlay-canvas" class="w-full h-full scale-x-[-1]"></canvas>
                    
                    <!-- Fallback instruction overlay when camera is closed -->
                    <div id="camera-placeholder" class="absolute inset-0 flex flex-col items-center justify-center bg-sky-50/95 p-4 text-center">
                        <div class="bg-white p-4 rounded-full text-sky-400 mb-3 shadow-md border-2 border-sky-200 animate-pulse">
                            <i class="fa-solid fa-video-slash text-4xl"></i>
                        </div>
                        <p class="text-xs text-slate-600 leading-relaxed font-bold">Webcam is currently inactive.<br>Click <span class="text-emerald-500">"Start Feed"</span> above or use the buttons below the game viewport to play!</p>
                    </div>
                </div>

                <!-- Angle Indicator display dashboard -->
                <div class="w-full grid grid-cols-2 gap-3 mt-4 text-center">
                    <div class="bg-sky-50 p-2.5 rounded-2xl border-2 border-sky-100">
                        <div class="text-[10px] text-slate-500 font-extrabold uppercase tracking-widest">Measured Roll</div>
                        <div id="debug-angle" class="text-lg font-black font-mono text-slate-700">0.0°</div>
                    </div>
                    <div class="bg-amber-50 p-2.5 rounded-2xl border-2 border-amber-200">
                        <div class="text-[10px] text-slate-500 font-extrabold uppercase tracking-widest">Decision State</div>
                        <div id="debug-state" class="text-lg font-black cartoon-font text-amber-500">CENTER</div>
                    </div>
                </div>
            </div>

            <!-- Question and Content Configuration Suite -->
            <div class="bg-white p-5 rounded-3xl border-4 border-amber-300 shadow-lg flex flex-col flex-1">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="text-sm font-black text-amber-600 cartoon-font uppercase tracking-wider"><i class="fa-solid fa-list-check mr-2 text-lg"></i>Question Creator</h3>
                    <div class="flex space-x-1.5">
                        <button id="load-preset-btn" class="bg-amber-100 hover:bg-amber-200 text-amber-700 text-xs py-1 px-2.5 rounded-lg font-black border-2 border-amber-300 transition">Presets</button>
                        <button id="reset-quiz-btn" class="bg-rose-50 hover:bg-rose-100 text-rose-500 text-xs py-1 px-2.5 rounded-lg font-black border-2 border-rose-200 transition">Clear</button>
                    </div>
                </div>

                <!-- Dynamic input box to feed the array -->
                <form id="add-question-form" class="space-y-3 bg-sky-50/50 p-3.5 rounded-2xl border-2 border-sky-100 mb-4">
                    <div>
                        <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest mb-1">Question Banner Text</label>
                        <input id="q-input" type="text" placeholder="e.g. Which vehicle travels faster?" class="w-full text-xs bg-white border-2 border-sky-100 rounded-xl p-2.5 text-slate-800 outline-none focus:border-amber-400 font-bold shadow-sm" required>
                    </div>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-[10px] font-black text-rose-500 uppercase tracking-widest mb-1">👈 Left Option (Pink)</label>
                            <input id="optL-input" type="text" placeholder="Rocket 🚀" class="w-full text-xs bg-white border-2 border-rose-100 rounded-xl p-2.5 text-slate-800 outline-none focus:border-rose-400 font-bold shadow-sm" required>
                        </div>
                        <div>
                            <label class="block text-[10px] font-black text-emerald-600 uppercase tracking-widest mb-1">👉 Right Option (Green)</label>
                            <input id="optR-input" type="text" placeholder="Bicycle 🚲" class="w-full text-xs bg-white border-2 border-emerald-100 rounded-xl p-2.5 text-slate-800 outline-none focus:border-emerald-400 font-bold shadow-sm" required>
                        </div>
                    </div>
                    <div>
                        <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest mb-1">Correct Answer Side</label>
                        <select id="correct-side" class="w-full text-xs bg-white border-2 border-sky-100 rounded-xl p-2.5 text-slate-800 outline-none focus:border-amber-400 font-black cursor-pointer shadow-sm">
                            <option value="LEFT">Left Answer Option</option>
                            <option value="RIGHT">Right Answer Option</option>
                        </select>
                    </div>
                    <button type="submit" class="w-full bg-amber-400 hover:bg-amber-350 text-slate-900 font-black text-xs py-2.5 rounded-xl transition shadow border-b-4 border-amber-600 active:border-b-0 active:translate-y-1">
                        <i class="fa-solid fa-plus mr-1"></i> Add to Live Rotation
                    </button>
                </form>

                <!-- Current Question List Container -->
                <div class="flex-1 flex flex-col min-h-[200px]">
                    <div class="text-[10px] font-black text-slate-500 uppercase tracking-widest mb-2">Live Question List (<span id="q-count">0</span>)</div>
                    <div id="question-list-container" class="space-y-2 flex-1 max-h-[260px] overflow-y-auto pr-1">
                        <!-- Questions will inject here dynamically -->
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- Footer Notes -->
    <footer class="bg-amber-100 py-4 text-center text-xs text-slate-500 border-t-4 border-amber-200">
        <p class="font-bold">© 2026 Tilt-O-Rama Creator Engine • Fully Adapted for Laptops and Interactive Kiosked Displays.</p>
    </footer>

    <!-- Web Audio & Core Engine Framework scripts -->
    <script>
        // -------------------------------------------------------------
        // AUDIO SYNTH ENGINE (Procedural Web Audio API Sound FX)
        // -------------------------------------------------------------
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        let audioCtx = null;

        function playSound(type) {
            try {
                if (!audioCtx) audioCtx = new AudioContext();
                const now = audioCtx.currentTime;

                if (type === 'tick') {
                    // Quick bubbly clock tick
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(600, now);
                    osc.frequency.exponentialRampToValueAtTime(100, now + 0.1);
                    gain.gain.setValueAtTime(0.1, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                    osc.start(now);
                    osc.stop(now + 0.1);
                } else if (type === 'correct') {
                    // Uplifting, cheerful dynamic double chime
                    const osc1 = audioCtx.createOscillator();
                    const osc2 = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    
                    osc1.connect(gain);
                    osc2.connect(gain);
                    gain.connect(audioCtx.destination);
                    
                    osc1.type = 'sine';
                    osc1.frequency.setValueAtTime(523.25, now); // C5
                    osc1.frequency.setValueAtTime(659.25, now + 0.08); // E5
                    
                    osc2.type = 'triangle';
                    osc2.frequency.setValueAtTime(783.99, now); // G5
                    osc2.frequency.setValueAtTime(1046.50, now + 0.08); // C6
                    
                    gain.gain.setValueAtTime(0.15, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.4);
                    
                    osc1.start(now);
                    osc2.start(now);
                    osc1.stop(now + 0.42);
                    osc2.stop(now + 0.42);
                } else if (type === 'incorrect') {
                    // Heavy comic sliding drop sound
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(220, now);
                    osc.frequency.exponentialRampToValueAtTime(50, now + 0.4);
                    gain.gain.setValueAtTime(0.15, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.4);
                    osc.start(now);
                    osc.stop(now + 0.42);
                } else if (type === 'pop') {
                    // Little swoosh/pop when locking
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(150, now);
                    osc.frequency.exponentialRampToValueAtTime(800, now + 0.15);
                    gain.gain.setValueAtTime(0.1, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);
                    osc.start(now);
                    osc.stop(now + 0.16);
                }
            } catch (err) {
                console.warn('Audio synthesis warning: ', err);
            }
        }

        // -------------------------------------------------------------
        // STATE STORAGE & DEFAULT QUESTION ROTATIONS
        // -------------------------------------------------------------
        const DEFAULT_QUESTIONS = [
            { id: "q1", question: "Which speedster animal is the fastest?", left: "Cheetah 🐆", right: "Snail 🐌", correct: "LEFT" },
            { id: "q2", question: "What falls from the sky during storms?", left: "Meatballs 🧆", right: "Raindrops 🌧️", correct: "RIGHT" },
            { id: "q3", question: "Which dynamic is hotter?", left: "Icebergs 🧊", right: "Volcanoes 🌋", correct: "RIGHT" },
            { id: "q4", question: "Which is the king of the jungle?", left: "Lion 🦁", right: "Koala 🐨", correct: "LEFT" }
        ];

        let questions = JSON.parse(localStorage.getItem('quiz_questions')) || [...DEFAULT_QUESTIONS];
        let currentQuestionIdx = 0;

        // -------------------------------------------------------------
        // SYNCHRONIZED SHARED CONTROL STATE
        // -------------------------------------------------------------
        let currentGlobalTilt = "CENTER"; // "LEFT", "RIGHT", "CENTER"
        let currentRotationAngle = 0; // Real degrees
        let tiltThreshold = 15; // Set via slider
        let lockinDelay = 1500; // milliseconds. 1.5s hold time.
        
        let appCameraActive = false;
        let mediaPipeFaceMesh = null;
        let mediaPipeCamera = null;

        // Save data to sync state
        function saveQuestions() {
            localStorage.setItem('quiz_questions', JSON.stringify(questions));
            renderQuestionListPanel();
            if (window.phaserGameInstance) {
                window.phaserGameInstance.events.emit('questions-updated');
            }
        }

        // -------------------------------------------------------------
        // FRONTEND PANEL BINDINGS & COMPONENT UPDATES
        // -------------------------------------------------------------
        const questionListContainer = document.getElementById('question-list-container');
        const qCountEl = document.getElementById('q-count');

        function renderQuestionListPanel() {
            qCountEl.innerText = questions.length;
            questionListContainer.innerHTML = '';
            
            questions.forEach((q, index) => {
                const item = document.createElement('div');
                item.className = "flex justify-between items-center bg-slate-50 p-2.5 rounded-xl border-2 border-slate-100 hover:border-amber-300 transition shadow-sm";
                item.innerHTML = `
                    <div class="flex-1 min-w-0 pr-2">
                        <div class="text-xs text-amber-600 font-extrabold tracking-wide">Q${index+1}: ${q.question}</div>
                        <div class="flex items-center space-x-1.5 text-[10px] mt-1 text-slate-500 font-bold">
                            <span class="${q.correct === 'LEFT' ? 'text-rose-500 font-black underline' : ''}">Left: ${q.left}</span>
                            <span>•</span>
                            <span class="${q.correct === 'RIGHT' ? 'text-emerald-600 font-black underline' : ''}">Right: ${q.right}</span>
                        </div>
                    </div>
                    <button onclick="removeQuestion('${q.id}')" class="text-slate-400 hover:text-rose-500 p-1.5 transition text-xs">
                        <i class="fa-solid fa-trash-can"></i>
                    </button>
                `;
                questionListContainer.appendChild(item);
            });
        }

        window.removeQuestion = function(id) {
            questions = questions.filter(q => q.id !== id);
            if (questions.length === 0) {
                questions = [...DEFAULT_QUESTIONS];
            }
            currentQuestionIdx = 0;
            saveQuestions();
        };

        // Form Submission
        document.getElementById('add-question-form').addEventListener('submit', function(e) {
            e.preventDefault();
            const qText = document.getElementById('q-input').value.trim();
            const optL = document.getElementById('optL-input').value.trim();
            const optR = document.getElementById('optR-input').value.trim();
            const side = document.getElementById('correct-side').value;

            const newQ = {
                id: 'custom_' + Date.now(),
                question: qText,
                left: optL,
                right: optR,
                correct: side
            };

            questions.push(newQ);
            saveQuestions();
            
            // Reset input values
            document.getElementById('q-input').value = '';
            document.getElementById('optL-input').value = '';
            document.getElementById('optR-input').value = '';
        });

        // Setup Calibration Listeners
        const thresholdSlider = document.getElementById('threshold-slider');
        const thresholdVal = document.getElementById('threshold-val');
        thresholdSlider.addEventListener('input', (e) => {
            tiltThreshold = parseInt(e.target.value);
            thresholdVal.innerText = `${tiltThreshold}°`;
        });

        const delaySlider = document.getElementById('delay-slider');
        const delayVal = document.getElementById('delay-val');
        delaySlider.addEventListener('input', (e) => {
            const val = parseFloat(e.target.value) / 10;
            lockinDelay = val * 1000;
            delayVal.innerText = `${val.toFixed(1)}s`;
        });

        // Load presets
        document.getElementById('load-preset-btn').addEventListener('click', () => {
            questions = [...DEFAULT_QUESTIONS];
            currentQuestionIdx = 0;
            saveQuestions();
        });

        // Clear All
        document.getElementById('reset-quiz-btn').addEventListener('click', () => {
            questions = [{ id: "q1", question: "Create your own customized questions on the right panel!", left: "Fun Left 🍕", right: "Fun Right 🥞", correct: "LEFT" }];
            currentQuestionIdx = 0;
            saveQuestions();
        });

        // -------------------------------------------------------------
        // MANUAL TILT SIMULATOR BINDINGS
        // -------------------------------------------------------------
        const simLeftBtn = document.getElementById('sim-left-btn');
        const simRightBtn = document.getElementById('sim-right-btn');
        const simCenterBtn = document.getElementById('sim-center-btn');

        simLeftBtn.addEventListener('click', () => {
            currentGlobalTilt = "LEFT";
            // positive rotation represents physical left tilt (counter-clockwise eye slant)
            currentRotationAngle = tiltThreshold + 5;
            updateManualDebugIndicators();
        });
        simRightBtn.addEventListener('click', () => {
            currentGlobalTilt = "RIGHT";
            // negative rotation represents physical right tilt (clockwise eye slant)
            currentRotationAngle = -(tiltThreshold + 5);
            updateManualDebugIndicators();
        });
        simCenterBtn.addEventListener('click', () => {
            currentGlobalTilt = "CENTER";
            currentRotationAngle = 0;
            updateManualDebugIndicators();
        });

        function updateManualDebugIndicators() {
            document.getElementById('debug-angle').innerText = `${currentRotationAngle.toFixed(1)}°`;
            document.getElementById('debug-state').innerText = currentGlobalTilt;
            
            const stateEl = document.getElementById('debug-state');
            if (currentGlobalTilt === 'LEFT') {
                stateEl.className = 'text-lg font-black cartoon-font text-rose-500';
            } else if (currentGlobalTilt === 'RIGHT') {
                stateEl.className = 'text-lg font-black cartoon-font text-emerald-500';
            } else {
                stateEl.className = 'text-lg font-black cartoon-font text-amber-500';
            }
        }

        // -------------------------------------------------------------
        // WEBCAM & MEDIAPIPE FACE MESH IMPLEMENTATION
        // -------------------------------------------------------------
        const videoInput = document.getElementById('webcam-input');
        const overlayCanvas = document.getElementById('mesh-overlay-canvas');
        const canvasCtx = overlayCanvas.getContext('2d');
        const toggleCamBtn = document.getElementById('toggle-camera-btn');
        const cameraPlaceholder = document.getElementById('camera-placeholder');

        function drawFaceMeshOverlay(results) {
            // Resize canvas to match the processing input frame aspect ratios
            if (overlayCanvas.width !== videoInput.videoWidth || overlayCanvas.height !== videoInput.videoHeight) {
                overlayCanvas.width = videoInput.videoWidth || 640;
                overlayCanvas.height = videoInput.videoHeight || 480;
            }

            canvasCtx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);
            
            // Draw camera frame directly to the overlay
            canvasCtx.drawImage(results.image, 0, 0, overlayCanvas.width, overlayCanvas.height);

            if (results.multiFaceLandmarks && results.multiFaceLandmarks.length > 0) {
                const landmarks = results.multiFaceLandmarks[0];

                // Landmark ID points for face tracking:
                // Left Eye outer corner: 33, Right Eye outer corner: 263
                const leftEye = landmarks[33];
                const rightEye = landmarks[263];

                // Calculate tilt angle based on difference between eyes relative to frame width/height
                const dx = (rightEye.x - leftEye.x) * overlayCanvas.width;
                const dy = (rightEye.y - leftEye.y) * overlayCanvas.height;
                
                // Convert angle from eye tilt to roll degrees
                const angleRad = Math.atan2(dy, dx);
                let degrees = angleRad * (180 / Math.PI);

                // Filter out micro-movements
                if (Math.abs(degrees) < 2) degrees = 0;

                // Sync current game variables directly
                currentRotationAngle = degrees; 

                // FIX FOR ROTATION INPUT MAPPING:
                // Tilting head to your RIGHT (clockwise) slants the eyes such that the right-most eye is lower, creating negative degrees.
                // Tilting head to your LEFT (counter-clockwise) slants eyes such that left-most eye is lower, creating positive degrees.
                if (currentRotationAngle < -tiltThreshold) {
                    currentGlobalTilt = "RIGHT";
                } else if (currentRotationAngle > tiltThreshold) {
                    currentGlobalTilt = "LEFT";
                } else {
                    currentGlobalTilt = "CENTER";
                }

                updateManualDebugIndicators();

                // Draw cartoonish wireframe lines over face
                const outerFaceIndices = [10, 338, 297, 332, 284, 251, 389, 356, 454, 323, 361, 288, 397, 365, 379, 378, 400, 377, 152, 148, 176, 149, 150, 136, 172, 58, 132, 93, 234, 127, 162, 21, 54, 103, 67, 109];
                
                // Set cute outline aesthetics
                canvasCtx.strokeStyle = currentGlobalTilt !== 'CENTER' ? '#f43f5e' : '#0284c7';
                canvasCtx.lineWidth = 4;
                canvasCtx.beginPath();
                
                outerFaceIndices.forEach((idx, i) => {
                    const lm = landmarks[idx];
                    const pxX = lm.x * overlayCanvas.width;
                    const pxY = lm.y * overlayCanvas.height;
                    if (i === 0) canvasCtx.moveTo(pxX, pxY);
                    else canvasCtx.lineTo(pxX, pxY);
                });
                canvasCtx.closePath();
                canvasCtx.stroke();

                // Draw glowing points on the eyes
                const drawGlowPoint = (lm, color) => {
                    canvasCtx.beginPath();
                    canvasCtx.arc(lm.x * overlayCanvas.width, lm.y * overlayCanvas.height, 8, 0, 2 * Math.PI);
                    canvasCtx.fillStyle = color;
                    canvasCtx.shadowColor = color;
                    canvasCtx.shadowBlur = 12;
                    canvasCtx.fill();
                    canvasCtx.shadowBlur = 0; // Reset
                };

                drawGlowPoint(landmarks[33], '#f43f5e'); // Rose glow Left Eye
                drawGlowPoint(landmarks[263], '#10b981'); // Emerald glow Right Eye
                drawGlowPoint(landmarks[1], '#f59e0b'); // Amber Nose dot
            }
        }

        function initFaceMeshTracking() {
            mediaPipeFaceMesh = new FaceMesh({
                locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`
            });

            mediaPipeFaceMesh.setOptions({
                maxNumFaces: 1,
                refineLandmarks: false,
                minDetectionConfidence: 0.5,
                minTrackingConfidence: 0.5
            });

            mediaPipeFaceMesh.onResults(drawFaceMeshOverlay);

            mediaPipeCamera = new Camera(videoInput, {
                onFrame: async () => {
                    if (appCameraActive) {
                        await mediaPipeFaceMesh.send({ image: videoInput });
                    }
                },
                width: 640,
                height: 480
            });
        }

        toggleCamBtn.addEventListener('click', async () => {
            if (!appCameraActive) {
                // Initialize tracking engine on first launch
                if (!mediaPipeFaceMesh) {
                    toggleCamBtn.innerText = "Connecting...";
                    initFaceMeshTracking();
                }

                try {
                    // Activate device input stream
                    await mediaPipeCamera.start();
                    appCameraActive = true;
                    toggleCamBtn.innerHTML = '<i class="fa-solid fa-power-off mr-1.5"></i>Stop Feed';
                    toggleCamBtn.className = "bg-rose-500 hover:bg-rose-400 text-white text-xs px-4 py-2 rounded-xl font-black shadow-md border-b-4 border-rose-700 active:border-b-0 active:translate-y-1 transition-all";
                    cameraPlaceholder.classList.add('hidden');
                    document.getElementById('tracking-status').innerText = "Camera Active";
                    document.getElementById('tracking-status').className = "text-emerald-600 font-bold";
                } catch (err) {
                    console.error("Camera access failed:", err);
                    toggleCamBtn.innerText = "Error!";
                    setTimeout(() => {
                        toggleCamBtn.innerHTML = '<i class="fa-solid fa-power-off mr-1.5"></i>Start Feed';
                        toggleCamBtn.className = "bg-emerald-500 hover:bg-emerald-400 text-white text-xs px-4 py-2 rounded-xl font-black shadow-md border-b-4 border-emerald-700 active:border-b-0 active:translate-y-1 transition-all";
                    }, 2000);
                }
            } else {
                // Turn Off Tracking
                await mediaPipeCamera.stop();
                appCameraActive = false;
                toggleCamBtn.innerHTML = '<i class="fa-solid fa-power-off mr-1.5"></i>Start Feed';
                toggleCamBtn.className = "bg-emerald-500 hover:bg-emerald-400 text-white text-xs px-4 py-2 rounded-xl font-black shadow-md border-b-4 border-emerald-700 active:border-b-0 active:translate-y-1 transition-all";
                cameraPlaceholder.classList.remove('hidden');
                document.getElementById('tracking-status').innerText = "Simulator Active";
                
                // Clear the canvas
                canvasCtx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);
                currentGlobalTilt = "CENTER";
                currentRotationAngle = 0;
                updateManualDebugIndicators();
            }
        });

        // -------------------------------------------------------------
        // PHASER 3 INTERACTIVE GAME ENGINE
        // -------------------------------------------------------------
        const phaserConfig = {
            type: Phaser.AUTO,
            parent: 'game-container',
            width: 800,
            height: 550,
            backgroundColor: '#bae6fd', // Cheerful sky blue game panel background
            physics: {
                default: 'arcade',
                arcade: { gravity: { y: 0 } }
            },
            scene: {
                preload: preloadScene,
                create: createScene,
                update: updateScene
            }
        };

        // Initialize Phaser View Frame
        const phaserGame = new Phaser.Game(phaserConfig);
        window.phaserGameInstance = phaserGame; // Expose globally for event bus syncing

        // Global variables inside the Phaser Scene Scope
        let currentSceneCtx;
        let pQuestionText;
        let pScoreText;
        let leftCardContainer, rightCardContainer;
        let cardBgLeft, cardBgRight;
        let optionTextLeft, optionTextRight;
        let progressRingLeft, progressRingRight;
        
        // Dynamic game flow trackers
        let gameScore = 0;
        let countdownProgress = 0; // Goes from 0 to lockinDelay
        let lastLoggedTilt = "CENTER";
        let localConfettiParticles;
        let stateIndicatorDot;
        let questionFeedbackBanner;

        function preloadScene() {
            currentSceneCtx = this;
            
            // Generate nice flat gradient/vector textures procedurally to ensure zero broken assets
            createDynamicGraphics(this);
        }

        // Helper to synthesize smooth vector graphics and map them to standard texture strings
        function createDynamicGraphics(scene) {
            // 1. Colorful Rounded Button Background for Options Cards
            const leftCardG = scene.make.graphics({ x: 0, y: 0, add: false });
            leftCardG.fillStyle(0xf43f5e, 1.0); // Bright bubblegum rose
            leftCardG.fillRoundedRect(4, 4, 272, 212, 32);
            leftCardG.lineStyle(8, 0xffffff, 1.0); // Thick premium white outline
            leftCardG.strokeRoundedRect(4, 4, 272, 212, 32);
            leftCardG.generateTexture('card_texture_left', 280, 220);

            const rightCardG = scene.make.graphics({ x: 0, y: 0, add: false });
            rightCardG.fillStyle(0x10b981, 1.0); // Cheerful emerald
            rightCardG.fillRoundedRect(4, 4, 272, 212, 32);
            rightCardG.lineStyle(8, 0xffffff, 1.0);
            rightCardG.strokeRoundedRect(4, 4, 272, 212, 32);
            rightCardG.generateTexture('card_texture_right', 280, 220);

            // 2. White Confetti Particle Dot
            const particleG = scene.make.graphics({ x: 0, y: 0, add: false });
            particleG.fillStyle(0xffffff, 1.0);
            particleG.fillCircle(8, 8, 8);
            particleG.generateTexture('particle_bubble', 16, 16);
            
            // 3. Status/Header Board
            const bannerG = scene.make.graphics({ x: 0, y: 0, add: false });
            bannerG.fillStyle(0xffffff, 0.95);
            bannerG.fillRoundedRect(4, 4, 752, 112, 24);
            bannerG.lineStyle(6, 0xfbbf24, 1.0); // Amber border
            bannerG.strokeRoundedRect(4, 4, 752, 112, 24);
            bannerG.generateTexture('top_banner', 760, 120);
        }

        function createScene() {
            // Draw a cartoon sky with fluffy vector backgrounds
            const skyBg = this.add.graphics();
            skyBg.fillGradientStyle(0xe0f2fe, 0xe0f2fe, 0xbae6fd, 0xbae6fd, 1);
            skyBg.fillRect(0, 0, 800, 550);

            // Draw cartoonish green landscape hills at bottom
            const hills = this.add.graphics();
            hills.fillStyle(0x86efac, 1.0);
            hills.fillEllipse(200, 560, 600, 150);
            hills.fillEllipse(600, 580, 700, 180);

            // Add Cheerful Floating Clouds (drawn procedurally)
            drawProceduralCloud(this, 120, 150, 0.6);
            drawProceduralCloud(this, 680, 120, 0.8);
            drawProceduralCloud(this, 400, 220, 0.4);

            // Top Status Panel (Question Banner)
            this.add.image(400, 80, 'top_banner');

            // Question Text with Fredoka-like rendering
            pQuestionText = this.add.text(400, 80, '', {
                fontFamily: 'Fredoka One, cursive',
                fontSize: '24px',
                fill: '#1e293b',
                align: 'center',
                wordWrap: { width: 700 }
            }).setOrigin(0.5);

            // Interactive Live Status Dot (at top header center)
            stateIndicatorDot = this.add.circle(400, 130, 8, 0xfbbf24);
            this.tweens.add({
                targets: stateIndicatorDot,
                scale: 1.2,
                duration: 600,
                yoyo: true,
                repeat: -1
            });

            // Score Counter Shield in bottom center
            const scoreLabel = this.add.text(400, 480, 'SCORE: 0', {
                fontFamily: 'Fredoka One, cursive',
                fontSize: '32px',
                fill: '#1e293b',
                stroke: '#ffffff',
                strokeThickness: 8
            }).setOrigin(0.5);
            pScoreText = scoreLabel;

            // -------------------------------------------------------------
            // OPTION A (LEFT) CONTAINER CARD
            // -------------------------------------------------------------
            leftCardContainer = this.add.container(220, 280);
            cardBgLeft = this.add.image(0, 0, 'card_texture_left');
            
            optionTextLeft = this.add.text(0, 0, '', {
                fontFamily: 'Fredoka One, cursive',
                fontSize: '28px',
                fill: '#ffffff',
                align: 'center',
                wordWrap: { width: 220 }
            }).setOrigin(0.5);

            // Dynamic Progress Indicator Bar for Lock-in Hold Timer
            progressRingLeft = this.add.graphics();

            // Assemble left stack
            leftCardContainer.add([cardBgLeft, optionTextLeft, progressRingLeft]);

            // -------------------------------------------------------------
            // OPTION B (RIGHT) CONTAINER CARD
            // -------------------------------------------------------------
            rightCardContainer = this.add.container(580, 280);
            cardBgRight = this.add.image(0, 0, 'card_texture_right');

            optionTextRight = this.add.text(0, 0, '', {
                fontFamily: 'Fredoka One, cursive',
                fontSize: '28px',
                fill: '#ffffff',
                align: 'center',
                wordWrap: { width: 220 }
            }).setOrigin(0.5);

            progressRingRight = this.add.graphics();

            // Assemble right stack
            rightCardContainer.add([cardBgRight, optionTextRight, progressRingRight]);

            // -------------------------------------------------------------
            // FEEDBACK BANNER OVERLAYS
            // -------------------------------------------------------------
            questionFeedbackBanner = this.add.text(400, 280, '', {
                fontFamily: 'Fredoka One, cursive',
                fontSize: '64px',
                fill: '#ffffff',
                stroke: '#1e293b',
                strokeThickness: 12
            }).setOrigin(0.5).setAlpha(0).setAngle(-5);

            // Setup Confetti Particle Emitters for celebration
            localConfettiParticles = this.add.particles(0, 0, 'particle_bubble', {
                speed: { min: 150, max: 300 },
                angle: { min: 0, max: 360 },
                scale: { start: 1, end: 0 },
                blendMode: 'SCREEN',
                lifespan: 1200,
                gravityY: 100,
                emitting: false
            });

            // Listen to data synchronization events from dashboard
            this.events.on('questions-updated', () => {
                loadCurrentQuestionData(this);
            });

            // Initial question fetch
            loadCurrentQuestionData(this);
        }

        function drawProceduralCloud(scene, x, y, scale) {
            const cloudG = scene.add.graphics();
            cloudG.fillStyle(0xffffff, 0.85);
            cloudG.fillCircle(x, y, 40 * scale);
            cloudG.fillCircle(x - 30 * scale, y + 10 * scale, 30 * scale);
            cloudG.fillCircle(x + 30 * scale, y + 10 * scale, 30 * scale);
            cloudG.fillCircle(x + 60 * scale, y + 15 * scale, 20 * scale);
            cloudG.fillCircle(x - 60 * scale, y + 15 * scale, 20 * scale);
            cloudG.fillRect(x - 60 * scale, y + 5 * scale, 120 * scale, 30 * scale);
        }

        function loadCurrentQuestionData(scene) {
            if (questions.length === 0) return;
            
            // Enforce array wrapping boundaries
            if (currentQuestionIdx >= questions.length) {
                currentQuestionIdx = 0;
            }

            const currentQ = questions[currentQuestionIdx];

            // Set question title & choices
            pQuestionText.setText(currentQ.question);
            optionTextLeft.setText(currentQ.left);
            optionTextRight.setText(currentQ.right);

            // Reset UI states
            countdownProgress = 0;
            leftCardContainer.setScale(1);
            rightCardContainer.setScale(1);
            
            // Fresh entering scale animation
            scene.tweens.add({
                targets: [leftCardContainer, rightCardContainer],
                scale: { start: 0.5, to: 1.0 },
                duration: 600,
                ease: 'Back.easeOut'
            });
        }

        // -------------------------------------------------------------
        // GAMEPLAY UPDATE LOOP
        // -------------------------------------------------------------
        function updateScene(time, delta) {
            if (questions.length === 0) return;

            // Sync visual indicator colors with decision state
            if (currentGlobalTilt === "LEFT") {
                stateIndicatorDot.setFillStyle(0xf43f5e); // Rose
            } else if (currentGlobalTilt === "RIGHT") {
                stateIndicatorDot.setFillStyle(0x10b981); // Emerald
            } else {
                stateIndicatorDot.setFillStyle(0xfbbf24); // Amber Center
            }

            // 1. Dynamic Card Juice & Scale Animations based on face tilt
            if (currentGlobalTilt === "LEFT") {
                // Smooth interpolation zoom
                leftCardContainer.scaleX = Phaser.Math.Linear(leftCardContainer.scaleX, 1.15, 0.15);
                leftCardContainer.scaleY = Phaser.Math.Linear(leftCardContainer.scaleY, 1.15, 0.15);
                leftCardContainer.setAngle(-3); // Gentle cocky tilt

                rightCardContainer.scaleX = Phaser.Math.Linear(rightCardContainer.scaleX, 0.85, 0.15);
                rightCardContainer.scaleY = Phaser.Math.Linear(rightCardContainer.scaleY, 0.85, 0.15);
                rightCardContainer.setAngle(0);

                // Accumulate hold progress
                countdownProgress += delta;
                if (Math.floor(countdownProgress / 200) > Math.floor((countdownProgress - delta) / 200)) {
                    playSound('tick');
                }
            } else if (currentGlobalTilt === "RIGHT") {
                rightCardContainer.scaleX = Phaser.Math.Linear(rightCardContainer.scaleX, 1.15, 0.15);
                rightCardContainer.scaleY = Phaser.Math.Linear(rightCardContainer.scaleY, 1.15, 0.15);
                rightCardContainer.setAngle(3);

                leftCardContainer.scaleX = Phaser.Math.Linear(leftCardContainer.scaleX, 0.85, 0.15);
                leftCardContainer.scaleY = Phaser.Math.Linear(leftCardContainer.scaleY, 0.85, 0.15);
                leftCardContainer.setAngle(0);

                countdownProgress += delta;
                if (Math.floor(countdownProgress / 200) > Math.floor((countdownProgress - delta) / 200)) {
                    playSound('tick');
                }
            } else {
                // Transition gracefully to neutral baseline
                leftCardContainer.scaleX = Phaser.Math.Linear(leftCardContainer.scaleX, 1.0, 0.1);
                leftCardContainer.scaleY = Phaser.Math.Linear(leftCardContainer.scaleY, 1.0, 0.1);
                leftCardContainer.setAngle(0);

                rightCardContainer.scaleX = Phaser.Math.Linear(rightCardContainer.scaleX, 1.0, 0.1);
                rightCardContainer.scaleY = Phaser.Math.Linear(rightCardContainer.scaleY, 1.0, 0.1);
                rightCardContainer.setAngle(0);

                countdownProgress = 0;
            }

            // Cap the progress accumulation
            if (countdownProgress > lockinDelay) {
                countdownProgress = lockinDelay;
                triggerAnswerLockin(this, currentGlobalTilt);
            }

            // 2. Render dynamic holding ring graphics inside the card boundaries
            drawCircularProgressIndicator();
        }

        function drawCircularProgressIndicator() {
            // Draw progress bar inside the Left Card
            progressRingLeft.clear();
            if (currentGlobalTilt === "LEFT" && countdownProgress > 0) {
                const percent = countdownProgress / lockinDelay;
                progressRingLeft.lineStyle(10, 0xffffff, 0.8);
                progressRingLeft.strokeCircle(0, 0, 110);
                progressRingLeft.lineStyle(12, 0xffeb3b, 1.0); // Gold loading stroke
                progressRingLeft.beginPath();
                progressRingLeft.arc(0, 0, 110, Phaser.Math.DegToRad(-90), Phaser.Math.DegToRad(-90 + (360 * percent)), false);
                progressRingLeft.stroke();
            }

            // Draw progress bar inside the Right Card
            progressRingRight.clear();
            if (currentGlobalTilt === "RIGHT" && countdownProgress > 0) {
                const percent = countdownProgress / lockinDelay;
                progressRingRight.lineStyle(10, 0xffffff, 0.8);
                progressRingRight.strokeCircle(0, 0, 110);
                progressRingRight.lineStyle(12, 0xffeb3b, 1.0);
                progressRingRight.beginPath();
                progressRingRight.arc(0, 0, 110, Phaser.Math.DegToRad(-90), Phaser.Math.DegToRad(-90 + (360 * percent)), false);
                progressRingRight.stroke();
            }
        }

        // -------------------------------------------------------------
        // LOCK IN AND SCORE GAME MECHANICS
        // -------------------------------------------------------------
        let scoringInputLockout = false;

        function triggerAnswerLockin(scene, chosenSide) {
            if (scoringInputLockout) return;
            scoringInputLockout = true;
            
            playSound('pop');

            const currentQ = questions[currentQuestionIdx];
            const isCorrect = (chosenSide === currentQ.correct);

            // Disable rendering state progress instantly
            countdownProgress = 0;

            // Play colorful splash overlay
            if (isCorrect) {
                gameScore += 10;
                pScoreText.setText(`SCORE: ${gameScore}`);
                playSound('correct');

                // Burst dynamic confetti around winning side
                const burstX = chosenSide === 'LEFT' ? 220 : 580;
                localConfettiParticles.emitParticleAt(burstX, 280, 40);

                // Show cheerful popup text
                questionFeedbackBanner.setText("FANTASTIC! ✨")
                                        .setFill('#10b981')
                                        .setStroke('#ffffff', 10);
            } else {
                playSound('incorrect');
                questionFeedbackBanner.setText("OOF! 💥")
                                        .setFill('#ef4444')
                                        .setStroke('#ffffff', 10);
                
                // Shake incorrect card
                const targetCard = (chosenSide === 'LEFT') ? leftCardContainer : rightCardContainer;
                scene.tweens.add({
                    targets: targetCard,
                    x: targetCard.x + 15,
                    duration: 50,
                    yoyo: true,
                    repeat: 5,
                    onComplete: () => {
                        // Reset card location bounds
                        targetCard.x = chosenSide === 'LEFT' ? 220 : 580;
                    }
                });
            }

            // Pop feed feedback text
            scene.tweens.add({
                targets: questionFeedbackBanner,
                scale: { start: 0.2, to: 1.2 },
                alpha: { start: 1, to: 1 },
                duration: 400,
                ease: 'Back.easeOut',
                completeDelay: 1000,
                onComplete: () => {
                    scene.tweens.add({
                        targets: questionFeedbackBanner,
                        alpha: 0,
                        scale: 0.5,
                        duration: 300,
                        onComplete: () => {
                            // Cycle indices and release lock
                            currentQuestionIdx = (currentQuestionIdx + 1) % questions.length;
                            loadCurrentQuestionData(scene);
                            scoringInputLockout = false;
                        }
                    });
                }
            });
        }

        // Initialize Panel Rendering
        renderQuestionListPanel();
    </script>
</body>
</html># madamhanisgan.github.io-
