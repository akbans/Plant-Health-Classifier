<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plant Health Classifier</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 0;
            background: url('backgnd.png') no-repeat center center fixed;
            background-size: cover;
            color: #fff;
            text-align: center;
        }

        h1 {
            margin-top: 20px;
            font-size: 2.5rem;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
        }

        .button-container, .sample-container {
            margin: 20px auto;
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
            gap: 15px;
        }

        button, img {
            cursor: pointer;
            border: none;
            border-radius: 8px;
            padding: 10px 20px;
            font-size: 1rem;
            background-color: #4caf50;
            color: #fff;
            box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
            transition: transform 0.2s, box-shadow 0.2s;
        }

        button:hover, img:hover {
            transform: scale(1.05);
            box-shadow: 0px 6px 10px rgba(0, 0, 0, 0.4);
        }

        img {
            width: 100px;
            height: 100px;
            object-fit: cover;
        }

        #webcam-container {
            margin: 20px auto;
        }

        #label-container div {
            font-size: 1.2rem;
            margin: 5px;
            text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.6);
        }
    </style>
</head>
<body>
    <h1>Plant Health Classifier</h1>
    <div class="button-container">
        <button type="button" onclick="init()">Start Webcam</button>
    </div>
    <div id="webcam-container"></div>
    <div id="label-container"></div>
    <h2>Sample Images</h2>
    <div class="sample-container">
        <img src="healthy1.jpg" alt="Healthy Sample 1" onclick="loadSampleImage('healthy1.jpg')">
        <img src="healthy2.jpg" alt="Healthy Sample 2" onclick="loadSampleImage('healthy2.jpg')">
        <img src="healthy3.jpg" alt="Healthy Sample 3" onclick="loadSampleImage('healthy3.jpg')">
        <img src="rust1.jpg" alt="Rust Sample 1" onclick="loadSampleImage('rust1.jpg')">
        <img src="rust2.jpg" alt="Rust Sample 2" onclick="loadSampleImage('rust2.jpg')">
        <img src="rust3.jpg" alt="Rust Sample 3" onclick="loadSampleImage('rust3.jpg')">
        <img src="mildew1.jpg" alt="Mildew Sample 1" onclick="loadSampleImage('mildew1.jpg')">
        <img src="mildew2.jpg" alt="Mildew Sample 2" onclick="loadSampleImage('mildew2.jpg')">
        <img src="mildew3.jpg" alt="Mildew Sample 3" onclick="loadSampleImage('mildew3.jpg')">
    </div>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
    <script>
        const URL = "https://teachablemachine.withgoogle.com/models/n9PWK3ckT/";
        let model, webcam, labelContainer, maxPredictions;

        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();

            const flip = true; 
            webcam = new tmImage.Webcam(200, 200, flip); 
            await webcam.setup();
            await webcam.play();
            window.requestAnimationFrame(loop);

            document.getElementById("webcam-container").appendChild(webcam.canvas);
            labelContainer = document.getElementById("label-container");
            labelContainer.innerHTML = "";
            for (let i = 0; i < maxPredictions; i++) {
                labelContainer.appendChild(document.createElement("div"));
            }
        }

        async function loop() {
            webcam.update();
            await predict(webcam.canvas);
            setTimeout(loop, 100); // Increased delay for stable percentages
        }

        async function predict(input) {
            const prediction = await model.predict(input);
            for (let i = 0; i < maxPredictions; i++) {
                const classPrediction = prediction[i].className + ": " + (prediction[i].probability * 100).toFixed(0) + "%";
                labelContainer.childNodes[i].innerHTML = classPrediction;
            }
        }

        async function loadSampleImage(imagePath) {
            const img = new Image();
            img.src = imagePath;
            img.onload = async () => {
                await predict(img);
            };
        }
    </script>
</body>
</html>
