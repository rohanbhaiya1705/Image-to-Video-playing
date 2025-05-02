# Image-to-Video-playing
Image to Video playing
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image to Video Player</title>
    <style>
        .container {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }

        .upload-section {
            margin-bottom: 20px;
        }

        .player-container {
            position: relative;
            background: #000;
            margin: 20px 0;
        }

        #displayImage {
            max-width: 100%;
            display: none;
        }

        .controls {
            background: rgba(0, 0, 0, 0.7);
            padding: 10px;
            position: absolute;
            bottom: 0;
            width: 100%;
            box-sizing: border-box;
        }

        .progress-bar {
            height: 5px;
            background: #444;
            margin-bottom: 10px;
            cursor: pointer;
        }

        .progress {
            height: 100%;
            background: #00ff00;
            width: 0%;
        }

        .time-display {
            color: white;
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
        }

        button {
            padding: 5px 15px;
            cursor: pointer;
        }

        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="upload-section">
            <input type="file" id="imageInput" accept="image/*">
            <input type="number" id="durationInput" placeholder="Duration in seconds" min="1">
            <button onclick="initializePlayer()">Generate Video</button>
        </div>

        <div class="player-container hidden" id="playerContainer">
            <img id="displayImage" alt="Uploaded content">
            <div class="controls">
                <div class="progress-bar" onclick="seek(event)">
                    <div class="progress" id="progressBar"></div>
                </div>
                <div class="time-display">
                    <span id="currentTime">0:00</span>
                    <span id="duration">0:00</span>
                </div>
                <button id="playPauseBtn" onclick="togglePlay()">Play</button>
            </div>
        </div>
    </div>

    <script>
        let isPlaying = false;
        let startTime = 0;
        let pausedTime = 0;
        let totalDuration = 0;
        let imageElement = document.getElementById('displayImage');
        let playerContainer = document.getElementById('playerContainer');

        function initializePlayer() {
            const fileInput = document.getElementById('imageInput');
            const durationInput = document.getElementById('durationInput');
            
            if (!fileInput.files[0] || !durationInput.value) {
                alert('Please upload an image and enter a duration');
                return;
            }

            totalDuration = parseInt(durationInput.value) * 1000;
            document.getElementById('duration').textContent = formatTime(totalDuration / 1000);

            const reader = new FileReader();
            reader.onload = function(e) {
                imageElement.src = e.target.result;
                imageElement.style.display = 'block';
                playerContainer.classList.remove('hidden');
            };
            reader.readAsDataURL(fileInput.files[0]);
        }

        function togglePlay() {
            const btn = document.getElementById('playPauseBtn');
            if (isPlaying) {
                pausedTime += Date.now() - startTime;
                btn.textContent = 'Play';
            } else {
                startTime = Date.now();
                btn.textContent = 'Pause';
                requestAnimationFrame(updateProgress);
            }
            isPlaying = !isPlaying;
        }

        function updateProgress() {
            if (!isPlaying) return;

            const currentTime = Date.now() - startTime + pausedTime;
            const progress = (currentTime / totalDuration) * 100;

            if (currentTime >= totalDuration) {
                isPlaying = false;
                document.getElementById('playPauseBtn').textContent = 'Play';
                pausedTime = 0;
                progressBar.style.width = '100%';
                document.getElementById('currentTime').textContent = formatTime(totalDuration / 1000);
                return;
            }

            document.getElementById('progressBar').style.width = `${progress}%`;
            document.getElementById('currentTime').textContent = formatTime(currentTime / 1000);

            requestAnimationFrame(updateProgress);
        }

        function seek(event) {
            if (!totalDuration) return;
            
            const rect = event.target.getBoundingClientRect();
            const percentage = (event.clientX - rect.left) / rect.width;
            pausedTime = totalDuration * percentage;
            
            document.getElementById('progressBar').style.width = `${percentage * 100}%`;
            document.getElementById('currentTime').textContent = formatTime((totalDuration * percentage) / 1000);
            
            if (isPlaying) {
                startTime = Date.now();
            }
        }

        function formatTime(seconds) {
            const hours = Math.floor(seconds / 3600);
            const minutes = Math.floor((seconds % 3600) / 60);
            const secs = Math.floor(seconds % 60);
            return [hours, minutes, secs]
                .map(v => v < 10 ? "0" + v : v)
                .join(':')
                .replace(/^00:/, '');
        }
    </script>
</body>
</html>
