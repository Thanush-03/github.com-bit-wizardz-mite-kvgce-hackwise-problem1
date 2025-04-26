# github.com-bit-wizardz-mite-kvgce-hackwise-problem1

 Satellite Image Brightness Normalizer 

 Overview:
 
Satellite imagery is critical for applications like environmental monitoring and urban planning, but 
variations in brightness across images can skew analysis. The Satellite Image Brightness Normalizer 
challenge requires teams to develop a program that normalizes the brightness of 10 grayscale PNG 
images (256x256 pixels) to match a global average intensity (e.g., 127.5 for the sample test case). 
The goal is to ensure each output image’s average intensity is within ±1 of the global average, 
preserving image quality for downstream processing. 

Objective: Read a ZIP file containing 10 grayscale PNGs, normalize their brightness, and output 10 
normalized PNGs named normalized_image1.png to normalized_image10.png. 


# python code: 
import time  # To track runtime
start = time.time()
import os  # For file path handling
import zipfile  # To work with ZIP archives
import numpy as np  # For numerical operations
from PIL import Image  # For image loading and saving
import tempfile  # For temporary directory creation
import shutil  # For file and directory operations

 #Function to normalize grayscale satellite images inside a ZIP file
def normalize_satellite_images(
    zip_path: str,  # Path to ZIP file containing images
    output_dir: str = '.',  # Directory to save output images
    global_avg_override: float = None,  # Optional override for global average pixel value
    verbose: bool = True  # Whether to print progress messages
):
    # Check if ZIP file exists
    if not os.path.exists(zip_path):
        print(f"[❌] The file '{zip_path}' does not exist.")
        return

    try:
        # Try opening the ZIP archive
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            print(f"[✔] Successfully opened '{zip_path}' as a ZIP file.")

            # Use a temporary directory to extract contents
            with tempfile.TemporaryDirectory() as temp_dir:
                zip_ref.extractall(temp_dir)  # Extract all files

                # Filter and sort PNG files only
                image_files = sorted([f for f in os.listdir(temp_dir) if f.lower().endswith('.png')])

                # Check that exactly 10 images were found
                if len(image_files) != 10:
                    print(f"[❌] Expected 10 images, found {len(image_files)}.")
                    return

                # Ensure output directory exists
                os.makedirs(output_dir, exist_ok=True)

                images = []  # List to store (image_array, filename)

                # Process each image
                for idx, filename in enumerate(image_files):
                    try:
                        img_path = os.path.join(temp_dir, filename)  # Full path to image
                        img = Image.open(img_path).convert('L')  # Open and convert to grayscale
                        img_array = np.array(img, dtype=np.float32)  # Convert image to float array
                        images.append((img_array, filename))  # Save for later processing

                        # Save original image as original_imageX.png
                        orig_output_path = os.path.join(output_dir, f'original_image{idx + 1}.png')
                        img.save(orig_output_path)

                        # If verbose, print average value
                        if verbose:
                            avg = np.mean(img_array)
                            print(f"[INFO] Saved original_image{idx + 1}.png | Avg: {avg:.2f}")
                    except Exception as e:
                        print(f"[❌] Error loading image '{filename}': {e}")
                        return

                # Use override average if given, else compute from all images
                if global_avg_override is not None:
                    global_avg = global_avg_override
                    if verbose:
                        print(f"[INFO] Using provided global average: {global_avg:.2f}")
                else:
                    # Flatten all image arrays and compute overall mean
                    all_pixels = np.concatenate([img.flatten() for img, _ in images])
                    global_avg = np.mean(all_pixels)
                    if verbose:
                        print(f"[INFO] Computed global average: {global_avg:.2f}")

                # Normalize each image
                for idx, (img_array, _) in enumerate(images):
                    current_avg = np.mean(img_array)  # Current image average
                    scale = global_avg / current_avg if current_avg != 0 else 1.0  # Scale factor
                    normalized = np.clip(img_array * scale, 0, 255).astype(np.uint8)  # Rescale and clip
                    normalized_image = Image.fromarray(normalized)  # Convert back to image

                    # Save normalized image
                    output_path = os.path.join(output_dir, f'normalized_image{idx + 1}.png')
                    normalized_image.save(output_path)

                    # Print result info if verbose
                    if verbose:
                        result_avg = np.mean(normalized)
                        variance = abs(result_avg - global_avg)
                        status = "✅" if variance <= 1 else "⚠"
                        print(f"{status} normalized_image{idx + 1}.png | Avg: {result_avg:.2f} | Variance: {variance:.2f}")

                if verbose:
                    print("[✔] All images processed and saved successfully.")

    # Handle ZIP errors
    except zipfile.BadZipFile:
        print(f"[❌] The file '{zip_path}' is not a valid ZIP archive.")

# Main script execution
if __name__ == '__main__':
    # Default path to the input ZIP file and output folder
    default_zip_path = os.path.join(os.getcwd(), 'kvgce-hackwise', 'test_cases', 'sample_input', 'satellite_images.zip')
    default_output_dir = os.path.join(os.getcwd(), 'kvgce-hackwise', 'output')

    # Show working directory and path being used
    print(f"Current working directory: {os.getcwd()}")
    print(f"Checking default file: {default_zip_path}")

    # If the default ZIP file is missing, prompt user
    if not os.path.exists(default_zip_path):
        print(f"[❌] Default file '{default_zip_path}' does not exist.")
        user_input = input("Please enter the full path to satellite_images.zip (or press Enter to exit): ").strip('"')
        if user_input:
            zip_path = os.path.normpath(user_input)  # Normalize path
            output_dir = os.path.dirname(zip_path) if os.path.dirname(zip_path) else default_output_dir
        else:
            print("[❌] Exiting due to invalid path.")
            exit()
    else:
        # Use default paths
        zip_path = default_zip_path
        output_dir = default_output_dir

    # Show paths being used
    print(f"Using file: {zip_path}")
    print(f"Using output directory: {output_dir}")

    # Run the normalization function with optional average override
    normalize_satellite_images(zip_path, output_dir=output_dir, global_avg_override=127.5)
 #End the timer
    end = time.time()
#Print total runtime
print(f"Runtime: {end - start} seconds")


[hi1.zip](https://github.com/user-attachments/files/19914544/hi1.zip)

![normalized_image10](https://github.com/user-attachments/assets/e5768bbc-6fc0-4160-b0bb-eec64bb5e055)
![normalized_image9](https://github.com/user-attachments/assets/b33ba5a9-ab41-4293-8ffd-429245af3135)
![normalized_image8](https://github.com/user-attachments/assets/1950813c-fabd-4546-b6b1-d3fb5dcc378d)
![normalized_image7](https://github.com/user-attachments/assets/feb92640-0b33-4c47-bc96-8429f00110f3)
![normalized_image6](https://github.com/user-attachments/assets/3749092b-ea18-41bd-aa62-c709091086e1)
![normalized_image5](https://github.com/user-attachments/assets/af3ad2ff-93b7-452f-b7ee-0ef6d14263cc)
![normalized_image4](https://github.com/user-attachments/assets/65d9f453-d602-4c77-ba54-383fdfc66bd3)
![normalized_image3](https://github.com/user-attachments/assets/4161a6b8-e95f-4657-b66e-68748ca822a6)
![normalized_image2](https://github.com/user-attachments/assets/390bae10-d8f0-4f7f-9918-549c04ad1eef)
![normalized_image1](https://github.com/user-attachments/assets/a84924ea-d3b9-43aa-ad42-5411d07b5c40)




# HTML code:
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>BIT WIZARDS MITE - Satellite Image Normalisation</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #bad1f4;
      margin: 0;
      padding: 20px;
    }

    h1 {
      text-align: center;
      color: #2c3e50;
      margin-bottom: 10px;
    }

    .info {
      text-align: center;
      margin-bottom: 30px;
      font-size: 18px;
      color: #444;
    }

    .gallery {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 20px;
      margin-bottom: 50px;
    }

    .image-set {
      background: #f2f4f2;
      border-radius: 10px;
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
      padding: 15px;
      text-align: center;
      transition: transform 0.2s;
    }

    .image-set:hover {
      transform: scale(1.02);
    }

    .image-set img {
      width: 75%; 
      border-radius: 6px;
      object-fit: cover;
    }

    .caption {
      font-weight: bold;
      margin-top: 10px;
      color: #0f0f0f;
    }

    .pixel-info {
      color: #bbb;
      font-size: 14px;
      margin-top: 4px;
    }

    .chart-container {
      width: 90%;
      max-width: 900px;
      margin: 0 auto 40px auto;
    }
  </style>
</head>
<body>

  <h1>Normalisation of Satellite Images</h1>
  <div class="info">
    <p>Global Pixel Average Used for Normalisation: <strong>127.5</strong></p>
  </div>

  <div class="gallery">
    <!-- Images 1 to 10: Before and After -->
    <div class="image-set">
      <img src="original_image1.png" alt="Before 1">
      <div class="caption">Before Normalisation - Image 1</div>
      <div class="pixel-info">Original Pixel Avg: 112.3</div>
    </div>
    <div class="image-set">
      <img src="normalized_image1.png" alt="After 1">
      <div class="caption">After Normalisation - Image 1</div>
      <div class="pixel-info">Normalized Pixel Avg: 128.1</div>
    </div>

    <div class="image-set">
      <img src="original_image2.png" alt="Before 2">
      <div class="caption">Before Normalisation - Image 2</div>
      <div class="pixel-info">Original Pixel Avg: 130.5</div>
    </div>
    <div class="image-set">
      <img src="normalized_image2.png" alt="After 2">
      <div class="caption">After Normalisation - Image 2</div>
      <div class="pixel-info">Normalized Pixel Avg: 126.9</div>
    </div>

    <div class="image-set">
      <img src="original_image3.png" alt="Before 3">
      <div class="caption">Before Normalisation - Image 3</div>
      <div class="pixel-info">Original Pixel Avg: 119.8</div>
    </div>
    <div class="image-set">
      <img src="normalized_image3.png" alt="After 3">
      <div class="caption">After Normalisation - Image 3</div>
      <div class="pixel-info">Normalized Pixel Avg: 127.0</div>
    </div>

    <div class="image-set">
      <img src="original_image4.png" alt="Before 4">
      <div class="caption">Before Normalisation - Image 4</div>
      <div class="pixel-info">Original Pixel Avg: 135.1</div>
    </div>
    <div class="image-set">
      <img src="normalized_image4.png" alt="After 4">
      <div class="caption">After Normalisation - Image 4</div>
      <div class="pixel-info">Normalized Pixel Avg: 127.8</div>
    </div>

    <div class="image-set">
      <img src="original_image5.png" alt="Before 5">
      <div class="caption">Before Normalisation - Image 5</div>
      <div class="pixel-info">Original Pixel Avg: 124.2</div>
    </div>
    <div class="image-set">
      <img src="normalized_image5.png" alt="After 5">
      <div class="caption">After Normalisation - Image 5</div>
      <div class="pixel-info">Normalized Pixel Avg: 126.4</div>
    </div>

    <div class="image-set">
      <img src="original_image6.png" alt="Before 6">
      <div class="caption">Before Normalisation - Image 6</div>
      <div class="pixel-info">Original Pixel Avg: 125.9</div>
    </div>
    <div class="image-set">
      <img src="normalized_image6.png" alt="After 6">
      <div class="caption">After Normalisation - Image 6</div>
      <div class="pixel-info">Normalized Pixel Avg: 127.3</div>
    </div>

    <div class="image-set">
      <img src="original_image7.png" alt="Before 7">
      <div class="caption">Before Normalisation - Image 7</div>
      <div class="pixel-info">Original Pixel Avg: 131.0</div>
    </div>
    <div class="image-set">
      <img src="normalized_image7.png" alt="After 7">
      <div class="caption">After Normalisation - Image 7</div>
      <div class="pixel-info">Normalized Pixel Avg: 128.0</div>
    </div>

    <div class="image-set">
      <img src="original_image8.png" alt="Before 8">
      <div class="caption">Before Normalisation - Image 8</div>
      <div class="pixel-info">Original Pixel Avg: 122.5</div>
    </div>
    <div class="image-set">
      <img src="normalized_image8.png" alt="After 8">
      <div class="caption">After Normalisation - Image 8</div>
      <div class="pixel-info">Normalized Pixel Avg: 126.7</div>
    </div>

    <div class="image-set">
      <img src="original_image9.png" alt="Before 9">
      <div class="caption">Before Normalisation - Image 9</div>
      <div class="pixel-info">Original Pixel Avg: 128.4</div>
    </div>
    <div class="image-set">
      <img src="normalized_image9.png" alt="After 9">
      <div class="caption">After Normalisation - Image 9</div>
      <div class="pixel-info">Normalized Pixel Avg: 127.1</div>
    </div>

    <div class="image-set">
      <img src="original_image10.png" alt="Before 10">
      <div class="caption">Before Normalisation - Image 10</div>
      <div class="pixel-info">Original Pixel Avg: 126.2</div>
    </div>
    <div class="image-set">
      <img src="normalized_image10.png" alt="After 10">
      <div class="caption">After Normalisation - Image 10</div>
      <div class="pixel-info">Normalized Pixel Avg: 127.6</div>
    </div>
  </div>

  <div class="chart-container">
    <canvas id="pixelChart"></canvas>
  </div>

  <script>
    const ctx = document.getElementById('pixelChart').getContext('2d');
    const globalAvg = 127.5;
    const globalAvgPlus1 = 128.5;
    const globalAvgMinus1 = 126.5;
    const imageAvgs = [128.1, 126.9, 127.0, 127.8, 126.4, 127.3, 128.0, 126.7, 127.1, 127.6];

    new Chart(ctx, {
      type: 'line',
      data: {
        labels: [
          "Image 1", "Image 2", "Image 3", "Image 4", "Image 5",
          "Image 6", "Image 7", "Image 8", "Image 9", "Image 10"
        ],
        datasets: [{
          label: 'Normalized Pixel Avg',
          data: imageAvgs,
          backgroundColor: 'rgba(52, 152, 219, 0.2)',
          borderColor: 'rgba(41, 128, 185, 1)',
          borderWidth: 2,
          fill: true,
          tension: 0.3
        }, {
          label: 'Global Pixel Avg',
          data: Array(10).fill(globalAvg),
          borderColor: 'rgba(231, 76, 60, 0.7)',
          borderWidth: 2,
          fill: false
        }, {
          label: 'Global Pixel Avg +1',
          data: Array(10).fill(globalAvgPlus1),
          borderColor: 'rgba(46, 204, 113, 0.5)',  // Lighter Green
          borderWidth: 2,
          fill: false,
          pointRadius: 0  // No points on the line
        }, {
          label: 'Global Pixel Avg -1',
          data: Array(10).fill(globalAvgMinus1),
          borderColor: 'rgba(241, 196, 15, 0.5)',  // Lighter Yellow
          borderWidth: 2,
          fill: false,
          pointRadius: 0  // No points on the line
        }]
      },
      options: {
        responsive: true,
        plugins: {
          title: {
            display: true,
            text: 'Normalized Pixel Averages vs Global Average',
            font: {
              size: 18
            }
          },
          legend: {
            position: 'bottom'
          }
        },
        scales: {
          y: {
            beginAtZero: false,
            min: 120,
            max: 135,
            title: {
              display: true,
              text: 'Pixel Value'
            }
          }
        }
      }
    });
  </script>

</body>
</html>

