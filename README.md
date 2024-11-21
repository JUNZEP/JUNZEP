<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Image Descriptor</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        :root {
            --primary-color: #3498db;
            --secondary-color: #2ecc71;
            --background-color: #f4f6f9;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            background-color: var(--background-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            line-height: 1.6;
        }

        .container {
            background-color: white;
            border-radius: 20px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 900px;
            padding: 40px;
            display: flex;
            flex-direction: column;
        }

        .workflow-visualization {
            display: flex;
            justify-content: space-between;
            margin-bottom: 30px;
            background-color: #f8f9fa;
            padding: 20px;
            border-radius: 15px;
        }

        .workflow-step {
            display: flex;
            flex-direction: column;
            align-items: center;
            text-align: center;
            width: 22%;
        }

        .workflow-step i {
            font-size: 3rem;
            color: var(--primary-color);
            margin-bottom: 15px;
        }

        .upload-section {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 30px;
        }

        #imageUpload {
            display: none;
        }

        .upload-label {
            display: flex;
            align-items: center;
            padding: 15px 30px;
            background-color: var(--primary-color);
            color: white;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .upload-label:hover {
            transform: scale(1.05);
            background-color: #2980b9;
        }

        .image-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .image-item {
            position: relative;
            border-radius: 15px;
            overflow: hidden;
            box-shadow: 0 8px 15px rgba(0,0,0,0.1);
            transition: transform 0.3s ease;
        }

        .image-item:hover {
            transform: scale(1.05);
        }

        .image-item img {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }

        .description-form {
            background-color: #f8f9fa;
            padding: 30px;
            border-radius: 15px;
            margin-top: 30px;
        }

        .form-group {
            margin-bottom: 20px;
        }

        .form-group label {
            display: block;
            margin-bottom: 10px;
            font-weight: bold;
        }

        .form-group input, 
        .form-group textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        .action-buttons {
            display: flex;
            justify-content: space-between;
            gap: 15px;
        }

        .btn {
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 12px 25px;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .btn-generate {
            background-color: var(--secondary-color);
            color: white;
        }

        .btn-download {
            background-color: var(--primary-color);
            color: white;
        }

        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid var(--primary-color);
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            display: none;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        @media (max-width: 768px) {
            .workflow-visualization {
                flex-direction: column;
            }

            .workflow-step {
                width: 100%;
                margin-bottom: 20px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Workflow Visualization -->
        <div class="workflow-visualization">
            <div class="workflow-step">
                <i class="fas fa-cloud-upload-alt"></i>
                <h3>Upload Image</h3>
                <p>Select multiple images to analyze</p>
            </div>
            <div class="workflow-step">
                <i class="fas fa-magic"></i>
                <h3>AI Analysis</h3>
                <p>Gemini API processes images</p>
            </div>
            <div class="workflow-step">
                <i class="fas fa-file-alt"></i>
                <h3>Generate Description</h3>
                <p>Create detailed image insights</p>
            </div>
            <div class="workflow-step">
                <i class="fas fa-download"></i>
                <h3>Download Report</h3>
                <p>Export image analysis</p>
            </div>
        </div>

        <!-- Image Upload Section -->
        <div class="upload-section">
            <input type="file" id="imageUpload" multiple accept="image/*">
            <label for="imageUpload" class="upload-label">
                <i class="fas fa-cloud-upload-alt"></i> Upload Images
            </label>
            <div id="imageGrid" class="image-grid"></div>
        </div>

        <!-- Description Form -->
        <div class="description-form" id="descriptionForm" style="display:none;">
            <div class="form-group">
                <label>File Details</label>
                <input type="text" id="fileDetails" readonly>
            </div>
            
            <div class="form-group">
                <label>AI Generated Description</label>
                <textarea id="aiDescription" rows="4" readonly></textarea>
            </div>

            <div class="form-group">
                <label>Keywords</label>
                <input type="text" id="keywords">
            </div>

            <div class="action-buttons">
                <button class="btn btn-generate" id="generateBtn">
                    <i class="fas fa-magic"></i> Generate Description
                </button>
                <button class="btn btn-download" id="downloadBtn" disabled>
                    <i class="fas fa-download"></i> Download Report
                </button>
            </div>
            
            <div class="loader" id="loader"></div>
        </div>
    </div>

    <script>
