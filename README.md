<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>LSB Steganography Tool</title>
  <style>
    * { box-sizing: border-box; }

    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 20px;
      background: #f2f2f2;
    }

    .container {
      max-width: 800px;
      margin: auto;
      background: #fff;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }

    h2, h3, h4 {
      color: #2c3e50;
      text-align: center;
    }

    label {
      display: block;
      font-weight: 600;
      margin-top: 20px;
    }

    textarea, input[type="file"], button {
      width: 100%;
      padding: 10px;
      font-size: 16px;
      margin-top: 6px;
      margin-bottom: 15px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }

    button {
      background-color: #3498db;
      color: white;
      border: none;
      cursor: pointer;
    }

    button:hover {
      background-color: #2980b9;
    }

    #downloadLink {
      display: inline-block;
      background: #2ecc71;
      color: #fff;
      padding: 10px 20px;
      border-radius: 6px;
      text-decoration: none;
      margin-top: 10px;
    }

    #downloadLink:hover {
      background-color: #27ae60;
    }

    #decodedMessage {
      min-height: 40px;
      border: 1px solid #ccc;
      padding: 10px;
      background: #f9f9f9;
      border-radius: 6px;
    }

    canvas {
      display: none;
    }

    .spinner {
      display: none;
      justify-content: center;
      align-items: center;
      margin: 20px 0;
    }

    .spinner svg {
      width: 40px;
      height: 40px;
      animation: spin 1s linear infinite;
    }

    @keyframes spin {
      100% {
        transform: rotate(360deg);
      }
    }

    @media (max-width: 600px) {
      textarea, input[type="file"], button {
        font-size: 15px;
      }
    }
  </style>
  
</head>
<body>
  <div class="container">
    <h2>LSB Steganography Tool</h2>

    <h3>1. Encode a Secret Message</h3>
    <label>Select Image (PNG/JPG):</label>
    <input type="file" id="imageInput">

    <label>Enter Secret Message:</label>
    <textarea id="secretText" rows="4" placeholder="Type your message here..."></textarea>

    <button onclick="encode()">Encode Message</button>
    <div class="spinner" id="encodeSpinner">
      <svg viewBox="0 0 50 50"><circle cx="25" cy="25" r="20" fill="none" stroke="#3498db" stroke-width="5"/></svg>
    </div>
    <a id="downloadLink" style="display:none;">Download Encoded Image</a>

    <hr>

    <h3>2. Decode Message from Image</h3>
    <label>Select Encoded Image:</label>
    <input type="file" id="decodeImageInput">

    <button onclick="decode()">Decode Message</button>
    <div class="spinner" id="decodeSpinner">
      <svg viewBox="0 0 50 50"><circle cx="25" cy="25" r="20" fill="none" stroke="#e67e22" stroke-width="5"/></svg>
    </div>

    <h4>Decoded Message:</h4>
    <div id="decodedMessage"></div>

    <canvas id="canvas"></canvas>
  </div>

  <script>
    function textToBinary(text) {
      return text.split('').map(char =>
        char.charCodeAt(0).toString(2).padStart(8, '0')
      ).join('') + '1111111111111110'; // End marker
    }

    function binaryToText(binary) {
      let chars = [];
      for (let i = 0; i < binary.length; i += 8) {
        const byte = binary.slice(i, i + 8);
        if (byte === '11111110') break;
        chars.push(String.fromCharCode(parseInt(byte, 2)));
      }
      return chars.join('');
    }

    function showSpinner(id) {
      document.getElementById(id).style.display = 'flex';
    }

    function hideSpinner(id) {
      document.getElementById(id).style.display = 'none';
    }

    function encode() {
      showSpinner('encodeSpinner');

      const fileInput = document.getElementById('imageInput');
      const canvas = document.getElementById('canvas');
      const ctx = canvas.getContext('2d');
      const secretText = document.getElementById('secretText').value;
      const reader = new FileReader();

      if (!fileInput.files.length) {
        hideSpinner('encodeSpinner');
        return alert("Please select an image.");
      }

      reader.onload = function(e) {
        const img = new Image();
        img.onload = function() {
          canvas.width = img.width;
          canvas.height = img.height;
          ctx.drawImage(img, 0, 0);
          const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
          const binary = textToBinary(secretText);
          let index = 0;

          for (let i = 0; i < imageData.data.length && index < binary.length; i += 4) {
            for (let j = 0; j < 3 && index < binary.length; j++) {
              imageData.data[i + j] = (imageData.data[i + j] & ~1) | parseInt(binary[index]);
              index++;
            }
          }

          ctx.putImageData(imageData, 0, 0);
          const link = document.getElementById('downloadLink');
          link.href = canvas.toDataURL();
          link.download = 'encoded_image.png';
          link.style.display = 'inline-block';
          hideSpinner('encodeSpinner');
        };
        img.src = e.target.result;
      };

      reader.readAsDataURL(fileInput.files[0]);
    }

    function decode() {
      showSpinner('decodeSpinner');

      const fileInput = document.getElementById('decodeImageInput');
      const canvas = document.getElementById('canvas');
      const ctx = canvas.getContext('2d');
      const reader = new FileReader();

      if (!fileInput.files.length) {
        hideSpinner('decodeSpinner');
        return alert("Please select an encoded image.");
      }

      reader.onload = function(e) {
        const img = new Image();
        img.onload = function() {
          canvas.width = img.width;
          canvas.height = img.height;
          ctx.drawImage(img, 0, 0);
          const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
          let binary = "";

          for (let i = 0; i < imageData.data.length; i += 4) {
            for (let j = 0; j < 3; j++) {
              binary += (imageData.data[i + j] & 1).toString();
              if (binary.slice(-16) === '1111111111111110') {
                document.getElementById('decodedMessage').innerText = binaryToText(binary);
                hideSpinner('decodeSpinner');
                return;
              }
            }
          }

          document.getElementById('decodedMessage').innerText = "No hidden message found.";
          hideSpinner('decodeSpinner');
        };
        img.src = e.target.result;
      };

      reader.readAsDataURL(fileInput.files[0]);
    }
  </script>
</body>
</html>
