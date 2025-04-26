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

HTML code:

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


