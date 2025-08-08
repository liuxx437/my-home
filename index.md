<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Piano Practice Counter</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-900 text-white flex flex-col items-center justify-center min-h-screen p-4">
    <div class="text-center">
        <h1 class="text-2xl sm:text-3xl font-bold mb-4">Perfect Practice Reps</h1>
        <div class="relative w-full max-w-2xl bg-gray-800 rounded-2xl p-8 shadow-2xl transition-all duration-300 transform scale-100 hover:scale-105">
            <!-- Counter Display -->
            <div id="counter-display" class="text-[12rem] sm:text-[15rem] font-extrabold text-white text-center tracking-tight transition-colors duration-300">
                0
            </div>
            <!-- Status Message Area -->
            <div id="status-message" class="text-xl text-gray-400 mt-4 transition-opacity duration-300 opacity-100">
                Press 'Start Counter' to begin.
            </div>
        </div>
        
        <!-- Controls -->
        <div class="flex flex-col sm:flex-row justify-center space-y-4 sm:space-y-0 sm:space-x-4 mt-8 w-full max-w-xl">
            <button id="start-button" class="flex-1 w-full sm:w-auto px-6 py-3 bg-green-600 hover:bg-green-700 text-white font-bold text-lg rounded-full shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 disabled:bg-gray-600 disabled:cursor-not-allowed focus:outline-none focus:ring-4 focus:ring-green-500 focus:ring-opacity-50">
                Start Counter
            </button>
            <button id="stop-button" class="flex-1 w-full sm:w-auto px-6 py-3 bg-red-600 hover:bg-red-700 text-white font-bold text-lg rounded-full shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 disabled:bg-gray-600 disabled:cursor-not-allowed focus:outline-none focus:ring-4 focus:ring-red-500 focus:ring-opacity-50" disabled>
                Stop Counter
            </button>
        </div>
        
        <!-- Error/Permission Message Box -->
        <div id="error-box" class="fixed inset-0 bg-black bg-opacity-75 hidden items-center justify-center p-4 z-50">
            <div class="bg-gray-800 text-white rounded-xl p-8 max-w-md w-full shadow-2xl text-center">
                <h3 class="text-2xl font-bold mb-4">Microphone Access Needed</h3>
                <p class="text-gray-300 mb-6">
                    This app requires microphone permission to listen for voice commands. Please allow access in the pop-up to continue.
                </p>
                <button id="close-error-box" class="px-6 py-2 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-full transition-all duration-300">
                    Got it!
                </button>
            </div>
        </div>
    </div>

    <script>
        // Check for Web Speech API compatibility
        if (!('webkitSpeechRecognition' in window)) {
            document.getElementById('status-message').textContent = 'Your browser does not support the Web Speech API. Please use a modern browser like Chrome.';
            document.getElementById('start-button').disabled = true;
            document.getElementById('stop-button').disabled = true;
        }

        let counter = 0;
        let isListening = false;
        let recognition;
        let speechRecognitionIsRunning = false;

        // DOM Elements
        const counterDisplay = document.getElementById('counter-display');
        const statusMessage = document.getElementById('status-message');
        const startButton = document.getElementById('start-button');
        const stopButton = document.getElementById('stop-button');
        const errorBox = document.getElementById('error-box');
        const closeErrorBoxButton = document.getElementById('close-error-box');

        // Function to update the counter display
        const updateCounterDisplay = () => {
            counterDisplay.textContent = counter;
        };

        // Function to set the status message
        const setStatus = (message) => {
            statusMessage.textContent = message;
        };

        // Function to initialize and start the speech recognition
        const startRecognition = () => {
            if (speechRecognitionIsRunning) {
                return;
            }

            try {
                // Initialize the recognition object
                recognition = new webkitSpeechRecognition();
                recognition.continuous = true;
                recognition.interimResults = false;
                recognition.lang = 'en-US';

                // Event handler for when a result is received
                recognition.onresult = (event) => {
                    const last = event.results.length - 1;
                    const transcript = event.results[last][0].transcript.trim().toLowerCase();

                    // Check for the stop command
                    if (transcript.includes('stop counter')) {
                        stopCounter();
                    } else {
                        // Increment the counter for any other phrase
                        counter++;
                        updateCounterDisplay();
                        setStatus(`Recognized: "${transcript}". Counter: ${counter}`);
                    }
                };

                // Event handler for errors
                recognition.onerror = (event) => {
                    console.error('Speech recognition error:', event.error);
                    if (event.error === 'not-allowed') {
                        errorBox.style.display = 'flex';
                    }
                    setStatus(`Error: ${event.error}. Please try again.`);
                    isListening = false;
                    speechRecognitionIsRunning = false;
                    startButton.disabled = false;
                    stopButton.disabled = true;
                };

                // Event handler for when the recognition service ends
                // We restart it automatically for continuous listening
                recognition.onend = () => {
                    if (isListening) {
                        setStatus('Listening again...');
                        recognition.start();
                    } else {
                        setStatus('Counter stopped.');
                        speechRecognitionIsRunning = false;
                        startButton.disabled = false;
                        stopButton.disabled = true;
                    }
                };
                
                recognition.start();
                isListening = true;
                speechRecognitionIsRunning = true;
                setStatus('Listening for commands...');
                startButton.disabled = true;
                stopButton.disabled = false;

            } catch (e) {
                console.error('An error occurred during recognition setup:', e);
                setStatus('Could not start the counter. Please check microphone permissions.');
            }
        };

        // Function to stop the counter
        const stopCounter = () => {
            if (isListening && recognition) {
                isListening = false;
                recognition.stop();
                setStatus('Counter stopped. Total reps: ' + counter);
                startButton.disabled = false;
                stopButton.disabled = true;
            }
        };

        // Event listener for the start button
        startButton.addEventListener('click', () => {
            if (!isListening) {
                counter = 0;
                updateCounterDisplay();
                startRecognition();
            }
        });

        // Event listener for the stop button
        stopButton.addEventListener('click', () => {
            stopCounter();
        });

        // Event listener for the close error box button
        closeErrorBoxButton.addEventListener('click', () => {
            errorBox.style.display = 'none';
        });

        // Voice command for starting the counter
        // We'll use a separate recognition instance just for the 'start' command,
        // and then start the main continuous one. This prevents a potential race condition.
        const startCommandRecognition = new webkitSpeechRecognition();
        startCommandRecognition.continuous = false;
        startCommandRecognition.interimResults = false;
        startCommandRecognition.lang = 'en-US';

        startCommandRecognition.onresult = (event) => {
            const transcript = event.results[0][0].transcript.trim().toLowerCase();
            if (transcript.includes('start counter')) {
                if (!isListening) {
                    counter = 0;
                    updateCounterDisplay();
                    startRecognition();
                }
            }
        };

        // Start listening for the 'start' command on page load
        startCommandRecognition.start();
        startCommandRecognition.onend = () => {
            if (!isListening) {
                // Restart listening for the start command if we're not yet running
                startCommandRecognition.start();
            }
        };

    </script>
</body>
</html>
