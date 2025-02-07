<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ultimate Image Editor</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      margin: 0;
      padding: 0;
      background: linear-gradient(135deg, #1e3c72, #2a5298); /* Modern gradient background */
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .container {
      background: rgba(255, 255, 255, 0.1); /* Semi-transparent white background */
      padding: 30px;
      border-radius: 15px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
      backdrop-filter: blur(10px); /* Frosted glass effect */
      max-width: 800px;
      width: 100%;
      text-align: center;
    }
    h1 {
      color: white;
      margin-bottom: 20px;
      font-size: 2.5rem;
    }
    input[type="file"] {
      margin: 20px 0;
      padding: 10px;
      background: rgba(255, 255, 255, 0.2);
      border: none;
      border-radius: 5px;
      color: white;
      cursor: pointer;
    }
    .image-container {
      margin: 20px 0;
      position: relative;
    }
    img {
      max-width: 100%;
      max-height: 400px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    }
    .tools {
      margin-top: 20px;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
    }
    button {
      padding: 10px 20px;
      border: none;
      border-radius: 5px;
      background: #007bff;
      color: #fff;
      cursor: pointer;
      transition: background 0.3s ease;
    }
    button:hover {
      background: #0056b3;
    }
    button:disabled {
      background: #ccc;
      cursor: not-allowed;
    }
    .slider-container {
      margin: 10px 0;
    }
    .slider-container label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
      color: white;
    }
    .text-overlay {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-size: 24px;
      font-weight: bold;
      background: rgba(0, 0, 0, 0.5); /* Semi-transparent background for text */
      padding: 5px 10px;
      border-radius: 5px;
      cursor: move;
    }
    .feedback {
      position: absolute;
      top: 10px;
      right: 10px;
      color: white;
      font-size: 14px;
    }
    .color-picker {
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div class="feedback">
    Feedback: <a href="mailto:sangboih32@gmail.com" style="color: white;">sangboih32@gmail.com</a>
  </div>
  <div class="container">
    <h1>Ultimate Image Editor</h1>
    <input type="file" id="imageInput" accept="image/*">
    <div class="image-container">
      <img id="previewImage" src="#" alt="Preview" style="display: none;">
      <div id="textOverlay" class="text-overlay" contenteditable="true" style="display: none;">Your Text Here</div>
    </div>
    <div class="tools">
      <button onclick="removeBackground()">Remove Background</button>
      <button onclick="applyGreenScreenBackground()">Green Screen Background</button>
      <button onclick="applyFilter('grayscale')">Grayscale</button>
      <button onclick="applyFilter('sepia')">Sepia</button>
      <button onclick="applyFilter('blur')">Blur</button>
      <button onclick="applyFilter('invert')">Invert</button>
      <button onclick="resetFilters()">Reset Filters</button>
      <button onclick="addText()">Add Text</button>
      <button onclick="restartEditor()">Restart</button>
      <button onclick="undo()" id="undoButton" disabled>Undo</button>
      <button onclick="redo()" id="redoButton" disabled>Redo</button>
      <button onclick="downloadImage()">Download</button>
    </div>
    <div class="slider-container">
      <label for="brightness">Brightness</label>
      <input type="range" id="brightness" min="0" max="200" value="100" oninput="applyColorGrading()">
    </div>
    <div class="slider-container">
      <label for="contrast">Contrast</label>
      <input type="range" id="contrast" min="0" max="200" value="100" oninput="applyColorGrading()">
    </div>
    <div class="slider-container">
      <label for="saturation">Saturation</label>
      <input type="range" id="saturation" min="0" max="200" value="100" oninput="applyColorGrading()">
    </div>
    <div class="slider-container">
      <label for="hue">Hue</label>
      <input type="range" id="hue" min="0" max="360" value="0" oninput="applyColorGrading()">
    </div>
    <div class="color-picker">
      <label for="textColor">Text Color</label>
      <input type="color" id="textColor" onchange="changeTextColor()">
    </div>
  </div>

  <script>
    const API_KEY = 'YqkU25zjeRBYVQibmYG5E51V'; // Replace with your API key
    let imageFile;
    let history = []; // Stores image states for undo/redo
    let currentStep = -1; // Tracks the current step in history

    // Load image preview
    document.getElementById('imageInput').addEventListener('change', function(event) {
      const file = event.target.files[0];
      if (file) {
        imageFile = file;
        const reader = new FileReader();
        reader.onload = function(e) {
          const previewImage = document.getElementById('previewImage');
          previewImage.src = e.target.result;
          previewImage.style.display = 'block';
          saveState(previewImage.src); // Save initial state
        };
        reader.readAsDataURL(file);
      }
    });

    // Save current image state to history
    function saveState(imageSrc) {
      history = history.slice(0, currentStep + 1); // Remove redo steps
      history.push(imageSrc);
      currentStep++;
      updateUndoRedoButtons();
    }

    // Undo functionality
    function undo() {
      if (currentStep > 0) {
        currentStep--;
        document.getElementById('previewImage').src = history[currentStep];
        updateUndoRedoButtons();
      }
    }

    // Redo functionality
    function redo() {
      if (currentStep < history.length - 1) {
        currentStep++;
        document.getElementById('previewImage').src = history[currentStep];
        updateUndoRedoButtons();
      }
    }

    // Enable/disable undo and redo buttons
    function updateUndoRedoButtons() {
      document.getElementById('undoButton').disabled = currentStep <= 0;
      document.getElementById('redoButton').disabled = currentStep >= history.length - 1;
    }

    // Remove background using Remove.bg API
    async function removeBackground() {
      if (!imageFile) {
        alert('Please upload an image first.');
        return;
      }

      const formData = new FormData();
      formData.append('image_file', imageFile);

      try {
        const response = await fetch('https://api.remove.bg/v1.0/removebg', {
          method: 'POST',
          headers: {
            'X-Api-Key': API_KEY,
          },
          body: formData,
        });

        if (!response.ok) {
          throw new Error('Failed to remove background.');
        }

        const blob = await response.blob();
        const url = URL.createObjectURL(blob);
        document.getElementById('previewImage').src = url;
        saveState(url); // Save state after background removal
      } catch (error) {
        console.error('Error:', error);
        alert('Failed to remove background. Please try again.');
      }
    }

    // Apply green screen background (simplified)
    function applyGreenScreenBackground() {
      const previewImage = document.getElementById('previewImage');
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');

      canvas.width = previewImage.width;
      canvas.height = previewImage.height;

      // Draw green background
      ctx.fillStyle = '#00ff00'; // Green color
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Draw the image on top
      ctx.drawImage(previewImage, 0, 0, canvas.width, canvas.height);

      // Update the preview image
      const newImageUrl = canvas.toDataURL('image/png');
      previewImage.src = newImageUrl;
      saveState(newImageUrl); // Save state after applying green screen
    }

    // Apply color grading (brightness, contrast, saturation, hue)
    function applyColorGrading() {
      const previewImage = document.getElementById('previewImage');
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');

      canvas.width = previewImage.width;
      canvas.height = previewImage.height;

      // Draw the image
      ctx.drawImage(previewImage, 0, 0, canvas.width, canvas.height);

      // Apply filters
      const brightness = document.getElementById('brightness').value;
      const contrast = document.getElementById('contrast').value;
      const saturation = document.getElementById('saturation').value;
      const hue = document.getElementById('hue').value;

      ctx.filter = `
        brightness(${brightness}%)
        contrast(${contrast}%)
        saturate(${saturation}%)
        hue-rotate(${hue}deg)
      `;

      // Redraw the image with filters
      ctx.drawImage(previewImage, 0, 0, canvas.width, canvas.height);

      // Update the preview image
      const newImageUrl = canvas.toDataURL('image/png');
      previewImage.src = newImageUrl;
      saveState(newImageUrl); // Save state after applying filters
    }

    // Apply predefined filters
    function applyFilter(filter) {
      const previewImage = document.getElementById('previewImage');
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');

      canvas.width = previewImage.width;
      canvas.height = previewImage.height;

      // Draw the image
      ctx.drawImage(previewImage, 0, 0, canvas.width, canvas.height);

      // Apply filter
      if (filter === 'grayscale') {
        ctx.filter = 'grayscale(100%)';
      } else if (filter === 'sepia') {
        ctx.filter = 'sepia(100%)';
      } else if (filter === 'blur') {
        ctx.filter = 'blur(5px)';
      } else if (filter === 'invert') {
        ctx.filter = 'invert(100%)';
      }

      // Redraw the image with filter
      ctx.drawImage(previewImage, 0, 0, canvas.width, canvas.height);

      // Update the preview image
      const newImageUrl = canvas.toDataURL('image/png');
      previewImage.src = newImageUrl;
      saveState(newImageUrl); // Save state after applying filter
    }

    // Reset all filters
    function resetFilters() {
      document.getElementById('brightness').value = 100;
      document.getElementById('contrast').value = 100;
      document.getElementById('saturation').value = 100;
      document.getElementById('hue').value = 0;
      applyColorGrading();
    }

    // Add text overlay
    function addText() {
      const textOverlay = document.getElementById('textOverlay');
      textOverlay.style.display = 'block';
    }

    // Change text color
    function changeTextColor() {
      const textOverlay = document.getElementById('textOverlay');
      const color = document.getElementById('textColor').value;
      textOverlay.style.color = color;
    }

    // Restart the editor
    function restartEditor() {
      const previewImage = document.getElementById('previewImage');
      previewImage.src = '#';
      previewImage.style.display = 'none';
      document.getElementById('textOverlay').style.display = 'none';
      history = [];
      currentStep = -1;
      updateUndoRedoButtons();
    }

    // Download edited image
    function downloadImage() {
      const previewImage = document.getElementById('previewImage');
      if (!previewImage.src || previewImage.src === '#') {
        alert('No image to download.');
        return;
      }

      // Create a temporary link element
      const link = document.createElement('a');
      link.href = previewImage.src;
      link.download = 'edited-image.png'; // Default filename
      document.body.appendChild(link);
      link.click(); // Trigger download
      document.body.removeChild(link); // Clean up
    }

    // Make text overlay draggable
    let isDragging = false;
    let offsetX, offsetY;

    document.getElementById('textOverlay').addEventListener('mousedown', function(e) {
      isDragging = true;
      offsetX = e.offsetX;
      offsetY = e.offsetY;
    });

    document.addEventListener('mousemove', function(e) {
      if (isDragging) {
        const textOverlay = document.getElementById('textOverlay');
        textOverlay.style.left = `${e.clientX - offsetX}px`;
        textOverlay.style.top = `${e.clientY - offsetY}px`;
      }
    });

    document.addEventListener('mouseup', function() {
      isDragging = false;
    });
  </script>
</body>
</html>
