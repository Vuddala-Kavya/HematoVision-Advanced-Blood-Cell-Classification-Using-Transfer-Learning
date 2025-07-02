project exceutable files

Coding part
Project Name : Hematovision

App.py page 
from flask import Flask, render_template, request, redirect, url_for 
import os 
from werkzeug.utils import secure_filename 
from tensorflow.keras.models import load_model 
from utils import preprocess_image, CLASS_NAMES 
import numpy as np 
 
app = Flask(__name__) 
app.config['UPLOAD_FOLDER'] = 'static/uploads' 
 
# Load the trained model 
model = load_model('model/hemato_model.h5') 
 
@app.route('/') 
def home(): 
    return render_template('index.html') 
 
@app.route('/predict', methods=['POST']) 
def predict(): 
    if 'file' not in request.files: 
        return redirect(request.url) 
    file = request.files['file'] 
    if file.filename == '': 
        return redirect(request.url)
     if file: 
        filename = secure_filename(file.filename) 
        file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename) 
        file.save(file_path) 
         
        img = preprocess_image(file_path) 
        prediction = model.predict(img) 
        class_index = np.argmax(prediction) 
        confidence = round(100 * np.max(prediction), 2) 
        label = CLASS_NAMES[class_index] 
         
        return render_template('result.html',  
                               label=label,  
                               confidence=confidence, 
                               user_image=file_path) 
    return redirect(url_for('index')) 
 
@app.route('/about') 
def about(): 
    return render_template('about.html') 
 
if __name__ == '__main__': 
    app.run(debug=True) 
 
Utils.py page 
 
from tensorflow.keras.preprocessing import image 
import numpy as np 
 
# Define the class labels 
CLASS_NAMES = ['Eosinophil', 'Lymphocyte', 'Monocyte', 'Neutrophil'] 
 
# Function to preprocess image 
def preprocess_image(img_path): 
    img = image.load_img(img_path, target_size=(224, 224)) 
    img_array = image.img_to_array(img) 
    img_array = img_array / 255.0  # Normalize 
    img_array = np.expand_dims(img_array, axis=0)  # Add batch dimension 
    return img_array
    Requirements.txt Page 
Flask==3.0.0 
tensorflow==2.15.0 
numpy 
pillow 
werkzeug 
 
Index.html page 
<!DOCTYPE html> 
<html lang="en"> 
<head> 
    <meta charset="UTF-8"> 
    <title>HematoVision</title> 
    <style> 
        body { 
            font-family: Arial, sans-serif; 
            background-color: #f5f5f5; 
            text-align: center; 
            margin: 0; 
            padding: 0; 
        } 
        .container { 
            max-width: 600px; 
            margin: 80px auto; 
            padding: 30px; 
            background-color: #fff; 
            box-shadow: 0 4px 8px rgba(0,0,0,0.1); 
            border-radius: 10px; 
        } 
        .header { 
            background-color: #d9534f; 
            color: white; 
            padding: 15px; 
            border-radius: 10px 10px 0 0; 
            font-size: 24px; 
            font-weight: bold; 
        } 
        .section { 
            margin: 30px 0; 
        } 
        h2 { 
            color: #d9534f; 
        } 
        p { 
            font-size: 16px; 
            color: #333; 
        } 
        input[type="file"] { 
            margin: 15px 0; 
        } 
        input[type="submit"] { 
            background-color: #d9534f; 
            color: white; 
            padding: 10px 20px; 
            border: none; 
            border-radius: 5px; 
            cursor: pointer; 
            font-weight: bold; 
        } 
        input[type="submit"]:hover { 
            background-color: #c9302c; 
        } 
    </style> 
</head> 
<body> 
    <div class="container"> 
        <div class="header">Welcome to the HematoVision</div> 
 
        <div class="section"> 
            <h2>About Blood Cells</h2> 
            <p>Blood cells are vital components of our body, playing essential roles in 
immunity, oxygen transport, and clotting. Understanding different types of blood 
cells is crucial for diagnosing various medical conditions.</p> 
        </div> 
 
        <div class="section"> 
            <h2>Predict Blood Cell Type</h2> 
            <form action="/predict" method="post" enctype="multipart/form-data"> 
                <input type="file" name="file" required> 
                <br> 
                <input type="submit" value="Predict"> 
            </form> 
        </div> 
    </div> 
</body> 
</html> 
 
result.html page 
 
<!DOCTYPE html> 
<html lang="en"> 
<head> 
    <meta charset="UTF-8"> 
    <title>Prediction Result</title> 
    <style> 
        body { 
            font-family: Arial, sans-serif; 
            background-color: #f5f5f5; 
            text-align: center; 
            margin: 0; 
            padding: 0; 
        } 
        .container { 
            max-width: 700px; 
            margin: 60px auto; 
            padding: 30px; 
            background-color: #fff; 
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1); 
            border-radius: 12px; 
        } 
        .header { 
            background-color: #d9534f; 
            color: white; 
            padding: 18px; 
            font-size: 26px; 
            font-weight: bold; 
            border-radius: 10px 10px 0 0; 
        } 
        .result { 
            margin: 30px 0; 
            font-size: 18px; 
        } 
        .result b { 
            color: #333; 
        } 
        .image-preview { 
            margin: 20px 0; 
        } 
        img { 
            width: 300px; 
            height: auto; 
            border-radius: 10px; 
        } 
        .button { 
            display: inline-block; 
            padding: 12px 25px; 
            font-size: 16px; 
            font-weight: bold; 
            color: white; 
            background-color: #d9534f; 
            border: none; 
            border-radius: 6px; 
            text-decoration: none; 
            margin-top: 20px; 
        } 
        .button:hover { 
            background-color: #c9302c; 
        } 
    </style> 
</head> 
<body> 
    <div class="container"> 
        <div class="header">Prediction Result</div> 
 
        <div class="result"> 
            <p><b>Predicted Class:</b> {{ label }}</p> 
            <p><b>Confidence:</b> {{ confidence }}%</p> 
        </div> 
 
<div class="image-preview"> 
<img src="{{ url_for('static', filename='uploads/' + user_image.split('/')[-1]) 
}}" alt="Uploaded Image"> 
</div> 
<a class="button" href="/">Upload Another Image</a> 
</div> 
</body> 
</html> 
