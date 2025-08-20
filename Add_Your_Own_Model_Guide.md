
# 🛠️ Add Your Own Model Guide

IntelliStack is designed as a flexible framework. You can easily plug in your own deep learning model and use the existing pipeline (frontend → Node.js → Python → Node.js → frontend).  

This guide walks you through the process.

---

## 📌 Step 1: Prepare Your Model
- Train and save your model in your preferred framework (PyTorch, TensorFlow, Keras, etc.).  
- Export in a supported format:  
  - **PyTorch:** `.pth` or `.pt`  
  - **TensorFlow/Keras:** `.h5` or `SavedModel/` directory  
  - **ONNX:** `.onnx` (optional, if you want framework independence)  

Example (PyTorch):  
```python
torch.save(model.state_dict(), "my_model.pth")
```

---

## 📌 Step 2: Modify `infer.py`
Open `model/infer.py`. This is where inference happens.  

1. **Import your framework and model**  
   ```python
   import torch
   from torchvision import transforms
   from my_model import CustomNet   # your model class
   ```

2. **Load the model**  
   ```python
   model = CustomNet()
   model.load_state_dict(torch.load("my_model.pth", map_location="cpu"))
   model.eval()
   ```

3. **Adjust preprocessing**  
   Change the image preprocessing to match your model’s input requirements. Example:  
   ```python
   preprocess = transforms.Compose([
       transforms.Resize((128, 128)),
       transforms.ToTensor(),
       transforms.Normalize(mean=[0.5], std=[0.5])
   ])
   ```

4. **Run inference and format output**  
   ```python
   with torch.no_grad():
       outputs = model(input_tensor)
       _, predicted = outputs.max(1)
       confidence = torch.nn.functional.softmax(outputs, dim=1)[0][predicted].item()
       label = str(predicted.item())  # or map to class names
   ```

5. **Return JSON**  
   Make sure to stick to this format so Node.js can understand it:  
   ```python
   result = {
       "label": label,
       "confidence": confidence,
       "framework": "PyTorch",
       "timestamp": datetime.now().isoformat()
   }
   print(json.dumps(result))
   ```

---

## 📌 Step 3: (Optional) Add a Model Selector
If you want to support **multiple models**, modify `infer.py` to take a `--model` argument.  

Example:
```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--model", default="resnet", help="Choose model: resnet, mobilenet, custom")
args = parser.parse_args()

if args.model == "resnet":
    # load resnet
elif args.model == "mobilenet":
    # load mobilenet
elif args.model == "custom":
    # load your model
```

Then, update Node.js (`frameController.js`) to call:  
```js
spawn("python", ["model/infer.py", "--model", "custom", tempFilePath]);
```

---

## 📌 Step 4: Test It
1. Start the server:
   ```bash
   npm start
   ```
2. Open the demo page:
   ```
   http://localhost:3000
   ```
3. Send frames and check if predictions from your model show up in real-time.

---

## 📌 Tips & Best Practices
- Always preprocess images exactly as you did during training.  
- Keep JSON output consistent (`label`, `confidence`, `framework`, `timestamp`).  
- Use **GPU acceleration** if available by moving your model to CUDA.  
- For large models, consider loading once and keeping it in memory instead of reloading for every frame.  

---

✅ That’s it! You now have IntelliStack running with your custom model.  
