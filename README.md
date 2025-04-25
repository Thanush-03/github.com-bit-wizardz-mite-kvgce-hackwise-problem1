# github.com-bit-wizardz-mite-kvgce-hackwise-problem1

 Satellite Image Brightness Normalizer 

code: 
import os
import zipfile
import numpy as np
from PIL import Image
import tempfile

def normalize_satellite_images(
    zip_path: str,
    output_dir: str = '.',
    global_avg_override: float = None,
    verbose: bool = True
):
    if not os.path.exists(zip_path):
        print(f"[Error] The file '{zip_path}' does not exist.")
        return

    try:
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            print(f"[✔] Successfully opened '{zip_path}' as a ZIP file.")

            with tempfile.TemporaryDirectory() as temp_dir:
                zip_ref.extractall(temp_dir)
                image_files = sorted([f for f in os.listdir(temp_dir) if f.endswith('.png')])

                if len(image_files) != 10:
                    print(f"[Error] Expected 10 images, found {len(image_files)}.")
                    return

                images = []
                for filename in image_files:
                    try:
                        img_path = os.path.join(temp_dir, filename)
                        img = Image.open(img_path).convert('L')
                        img_array = np.array(img, dtype=np.float32)
                        images.append(img_array)
                    except Exception as e:
                        print(f"[Error] Error loading image '{filename}': {e}")
                        return

                if global_avg_override is not None:
                    global_avg = global_avg_override
                    if verbose:
                        print(f"[INFO] Using provided global average: {global_avg:.2f}")
                else:
                    all_pixels = np.concatenate([img.flatten() for img in images])
                    global_avg = np.mean(all_pixels)
                    if verbose:
                        print(f"[INFO] Computed global average: {global_avg:.2f}")

                os.makedirs(output_dir, exist_ok=True)
                for idx, img_array in enumerate(images):
                    current_avg = np.mean(img_array)
                    scale = global_avg / current_avg if current_avg != 0 else 1.0
                    normalized = np.clip(img_array * scale, 0, 255).astype(np.uint8)
                    output_path = os.path.join(output_dir, f'normalized_image{idx + 1}.png')
                    Image.fromarray(normalized).save(output_path)

                    if verbose:
                        result_avg = np.mean(normalized)
                        status = "✅" if abs(result_avg - global_avg) <= 1 else "⚠"
                        print(f"{status} normalized_image{idx + 1}.png | Avg: {result_avg:.2f}")

                if verbose:
                    print("[✔] All images processed and saved successfully.")

    except zipfile.BadZipFile:
        print(f"[Error] The file '{zip_path}' is not a valid ZIP archive.")

if __name__ == '__main__':
    default_zip_path = os.path.join(os.getcwd(), 'kvgce-hackwise', 'test_cases', 'sample_input', 'satellite_images.zip')
    default_output_dir = os.path.join(os.getcwd(), 'kvgce-hackwise', 'output')

    print(f"Current working directory: {os.getcwd()}")
    print(f"Checking default file: {default_zip_path}")

    if not os.path.exists(default_zip_path):
        print(f"[Error] Default file '{default_zip_path}' does not exist.")
        user_input = input("Please enter the full path to satellite_images.zip (or press Enter to exit): ").strip('"')
        if user_input:
            zip_path = os.path.normpath(user_input)
            output_dir = os.path.dirname(zip_path) if os.path.dirname(zip_path) else default_output_dir
        else:
            print("[Error] Exiting due to invalid path.")
            exit()
    else:
        zip_path = default_zip_path
        output_dir = default_output_dir

    print(f"Using file: {zip_path}")
    print(f"Using output directory: {output_dir}")

    normalize_satellite_images(zip_path, output_dir=output_dir, global_avg_override=127.5)


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


[hi1.zip](https://github.com/user-attachments/files/19914544/hi1.zip)

