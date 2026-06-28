# Running Unlimited-OCR on Google Colab

Use this document to quickly set up and run Baidu's **Unlimited-OCR** model in Google Colab using a GPU instance.

---

## 🛠️ Step 1: Change Runtime to GPU
Before running any code, configure the Colab hardware accelerator:
1. In the top menu of Google Colab, go to **Runtime** > **Change runtime type**.
2. Under **Hardware accelerator**, select **T4 GPU**.
3. Click **Save**.
4. Create a cell, paste the following command, and run it to verify the GPU is active:
   ```bash
   !nvidia-smi
   ```

---

## 📦 Step 2: Install Dependencies
Run the following commands in a cell to clear any conflicting packages, install the correct dependencies, and restart the kernel:

```python
# 1. Clean pip cache and install specific dependencies
!pip cache purge
!pip install --force-reinstall transformers==4.46.0 tokenizers==0.20.1 torch torchvision pymupdf einops addict easydict psutil accelerate

# 2. Programmatically restart the Python kernel to load the new libraries
import os
os.kill(os.getpid(), 9)
```

---

## 🚀 Step 3: Run the 10-Page Recursive OCR Pipeline
Create a new cell, paste the following Python script, and run it. 

> [!IMPORTANT]
> Since we are switching the batch size from 20 pages to 10 pages, **please rename or delete your old `output_markdown` folder** on Google Drive before running this script. This avoids folder overlap and ensures a clean 10-page directory structure.

```python
import os
import sys
import time
import torch
import shutil
import tempfile
import logging
import warnings
import re
import fitz  # PyMuPDF
from transformers import AutoModel, AutoTokenizer
from google.colab import drive

# ── Suppress verbose warnings and logs ──
warnings.filterwarnings("ignore")
logging.getLogger("transformers").setLevel(logging.ERROR)
os.environ["TOKENIZERS_PARALLELISM"] = "false"

# 1. Mount Google Drive
print("📁 Mounting Google Drive...")
drive.mount('/content/drive')

# 2. Configured Paths
INPUT_DIR = "/content/drive/MyDrive/for OCR"
OUTPUT_DIR = "/content/drive/MyDrive/for OCR/output_markdown"
os.makedirs(OUTPUT_DIR, exist_ok=True)

# 3. Initialize Model on GPU with Retry Loop
model_name = 'baidu/Unlimited-OCR'
print("🧠 Loading model and tokenizer...")

tokenizer = None
model = None
for attempt in range(1, 4):
    try:
        print(f"   Attempt {attempt}/3 to load model...")
        tokenizer = AutoTokenizer.from_pretrained(model_name, trust_remote_code=True)
        model = AutoModel.from_pretrained(
            model_name,
            trust_remote_code=True,
            use_safetensors=True,
            torch_dtype=torch.bfloat16,
        )
        model = model.eval().cuda()
        print("✅ Model loaded successfully!\n")
        break
    except Exception as e:
        print(f"   ⚠️ Loading failed: {e}")
        if attempt < 3:
            print("   Waiting 10 seconds before retrying...")
            time.sleep(10)
        else:
            print("   ❌ Fatal: Could not load the model after 3 attempts.")
            sys.exit(1)

def pdf_to_images(pdf_path, dpi=200):
    doc = fitz.open(pdf_path)
    tmp_dir = tempfile.mkdtemp(prefix='pdf_ocr_')
    mat = fitz.Matrix(dpi / 72, dpi / 72)
    paths = []
    for i, page in enumerate(doc):
        out = os.path.join(tmp_dir, f'page_{i+1:04d}.png')
        page.get_pixmap(matrix=mat).save(out)
        paths.append(out)
    return paths

def chunk_pages(lst, n):
    for i in range(0, len(lst), n):
        yield lst[i:i + n]

def format_time(seconds):
    if seconds < 60:
        return f"{seconds:.0f}s"
    elif seconds < 3600:
        return f"{seconds // 60:.0f}m {seconds % 60:.0f}s"
    else:
        return f"{seconds // 3600:.0f}h {(seconds % 3600) // 60:.0f}m"

def run_inference(image_files, output_dir):
    os.makedirs(output_dir, exist_ok=True)
    old_stdout = sys.stdout
    sys.stdout = open(os.devnull, 'w')
    try:
        model.infer_multi(
            tokenizer,
            prompt='<image>Multi page parsing.',
            image_files=image_files,
            output_path=output_dir,
            image_size=1024,
            max_length=32768,
            no_repeat_ngram_size=35, 
            ngram_window=1024,
            save_results=True,
        )
    finally:
        sys.stdout = old_stdout
    torch.cuda.empty_cache()

def merge_subsets(dir1, dir2, dest_dir, offset):
    os.makedirs(dest_dir, exist_ok=True)
    
    md1_path = os.path.join(dir1, "result.md")
    md2_path = os.path.join(dir2, "result.md")
    
    content1 = ""
    if os.path.exists(md1_path):
        with open(md1_path, 'r', errors='ignore') as f:
            content1 = f.read().strip()
            
    content2 = ""
    if os.path.exists(md2_path):
        with open(md2_path, 'r', errors='ignore') as f:
            content2 = f.read().strip()
            
    # Copy images from part 1
    dest_images_dir = os.path.join(dest_dir, "images")
    os.makedirs(dest_images_dir, exist_ok=True)
    
    img1_dir = os.path.join(dir1, "images")
    if os.path.exists(img1_dir):
        for filename in os.listdir(img1_dir):
            shutil.copy2(os.path.join(img1_dir, filename), os.path.join(dest_images_dir, filename))
            
    # Copy and rename images from part 2 to prevent collision
    img2_dir = os.path.join(dir2, "images")
    if os.path.exists(img2_dir):
        for filename in os.listdir(img2_dir):
            match = re.match(r'page_(\d+)_(\d+)\.jpg', filename)
            if match:
                local_page = int(match.group(1))
                img_idx = int(match.group(2))
                new_page = local_page + offset
                new_filename = f"page_{new_page}_{img_idx}.jpg"
                shutil.copy2(os.path.join(img2_dir, filename), os.path.join(dest_images_dir, new_filename))
                
                # Replace reference in content2
                content2 = content2.replace(f"images/page_{local_page}_{img_idx}.jpg", f"images/page_{new_page}_{img_idx}.jpg")
            else:
                shutil.copy2(os.path.join(img2_dir, filename), os.path.join(dest_images_dir, filename))
                
    # Combine markdown
    combined_content = content1 + "\n" + content2
    with open(os.path.join(dest_dir, "result.md"), 'w', encoding='utf-8') as f:
        f.write(combined_content)
        
    # Copy and rename debug result_with_boxes_X.jpg
    for filename in os.listdir(dir1):
        if filename.startswith("result_with_boxes_") and filename.endswith(".jpg"):
            shutil.copy2(os.path.join(dir1, filename), os.path.join(dest_dir, filename))
            
    for filename in os.listdir(dir2):
        if filename.startswith("result_with_boxes_") and filename.endswith(".jpg"):
            match = re.match(r'result_with_boxes_(\d+)\.jpg', filename)
            if match:
                local_idx = int(match.group(1))
                new_idx = local_idx + offset
                new_filename = f"result_with_boxes_{new_idx}.jpg"
                shutil.copy2(os.path.join(dir2, filename), os.path.join(dest_dir, new_filename))

# Global progress state
global_processed_pages = 0
global_total_pages = 0

def process_range_recursive(image_files, output_dir, path_label):
    """Recursively processes a list of images. Splits into halves if truncation occurs."""
    global global_processed_pages, global_total_pages
    expected_pages = len(image_files)
    if expected_pages == 0:
        return
    
    # Run the model on the range
    run_inference(image_files, output_dir)
    
    res_md = os.path.join(output_dir, "result.md")
    page_count = 0
    if os.path.exists(res_md):
        with open(res_md, 'r', errors='ignore') as f:
            content = f.read()
        page_count = content.count("<PAGE>")
        
    if page_count >= expected_pages:
        # Success! Update global count
        global_processed_pages += expected_pages
        return
        
    if expected_pages > 1:
        print(f"   ⚠️ Truncation detected ({page_count}/{expected_pages} pages) in {path_label}. Splitting range...")
        
        # Clear the failed output
        if os.path.exists(output_dir):
            shutil.rmtree(output_dir)
        os.makedirs(output_dir, exist_ok=True)
        
        mid = expected_pages // 2
        part1 = image_files[:mid]
        part2 = image_files[mid:]
        
        with tempfile.TemporaryDirectory() as tmp1, tempfile.TemporaryDirectory() as tmp2:
            process_range_recursive(part1, tmp1, f"{path_label} (Part 1)")
            process_range_recursive(part2, tmp2, f"{path_label} (Part 2)")
            merge_subsets(tmp1, tmp2, output_dir, len(part1))
    else:
        print(f"   ❌ Fatal: 1-page batch failed. Keeping partial result.")
        global_processed_pages += 1

# 4. Process each PDF
pdf_files = [f for f in os.listdir(INPUT_DIR) if f.lower().endswith('.pdf')]
print(f"📄 Found {len(pdf_files)} PDFs to process:")
for i, f in enumerate(pdf_files, 1):
    print(f"   {i}. {f}")
print()

total_start = time.time()

for file_idx, pdf_file in enumerate(pdf_files, 1):
    pdf_path = os.path.join(INPUT_DIR, pdf_file)
    file_output_dir = os.path.join(OUTPUT_DIR, os.path.splitext(pdf_file)[0])
    os.makedirs(file_output_dir, exist_ok=True)
    
    print(f"{'='*60}")
    print(f"📖 [{file_idx}/{len(pdf_files)}] Processing: {pdf_file}")
    print(f"{'='*60}")
    
    try:
        # Convert PDF pages to temporary images
        print("   Converting PDF pages to images...", end=" ")
        img_start = time.time()
        image_files = pdf_to_images(pdf_path)
        total_pages = len(image_files)
        print(f"Done! ({total_pages} pages in {format_time(time.time() - img_start)})")
        
        # Reset global progress variables for this file
        global_processed_pages = 0
        global_total_pages = total_pages
        
        # Batch preparation (20-page batches for boundary alignment)
        batch_size = 20
        batches = list(chunk_pages(image_files, batch_size))
        num_batches = len(batches)
        
        # --- Pre-scan and clean up incomplete batches ---
        print("   Checking existing batch folders for truncation...")
        cleaned_any = False
        for idx in range(num_batches):
            page_start = idx * batch_size + 1
            page_end = min((idx + 1) * batch_size, total_pages)
            expected_pages = page_end - page_start + 1
            
            batch_output_dir = os.path.join(file_output_dir, f"batch_{idx+1}")
            res_md = os.path.join(batch_output_dir, "result.md")
            
            if os.path.exists(res_md):
                with open(res_md, 'r', errors='ignore') as f:
                    content = f.read()
                page_count = content.count("<PAGE>")
                
                # Check for mismatch
                if page_count != expected_pages:
                    print(f"   🗑️ Batch {idx+1} is partial/truncated ({page_count}/{expected_pages} pages). Clearing folder for split re-run.")
                    if os.path.exists(res_md):
                        os.remove(res_md)
                    img_dir = os.path.join(batch_output_dir, "images")
                    if os.path.exists(img_dir):
                        shutil.rmtree(img_dir)
                    cleaned_any = True
                else:
                    # If it is already complete, update global processed pages
                    global_processed_pages += expected_pages
        
        if not cleaned_any:
            print("   ✅ All existing batch folders are complete.")
        
        # --- Process batches ---
        print(f"   Processing in {num_batches} batches of {batch_size} pages...\n")
        file_start_time = time.time()
        
        for idx, batch in enumerate(batches):
            batch_idx = idx + 1
            batch_start = time.time()
            page_start = idx * batch_size + 1
            page_end = min((idx + 1) * batch_size, total_pages)
            expected_pages = page_end - page_start + 1
            
            batch_output_dir = os.path.join(file_output_dir, f"batch_{batch_idx}")
            res_md = os.path.join(batch_output_dir, "result.md")
            
            # Since incomplete batches had their result.md deleted in the pre-scan step,
            # any existing result.md here is guaranteed to be complete.
            if os.path.exists(res_md):
                print(f"   ⏩ Batch {batch_idx}/{num_batches} (pages {page_start}-{page_end}) already completed. Skipping.")
                continue
            
            # Smart check: if the folder exists but result.md is missing, this was a failed run.
            # Bypasses the 10-page run and splits immediately to save time.
            should_split = os.path.exists(batch_output_dir)
            
            os.makedirs(batch_output_dir, exist_ok=True)
            
            if not should_split:
                # Attempt to process the whole batch
                process_range_recursive(batch, batch_output_dir, f"Batch {batch_idx}")
            else:
                # Force instant split
                print(f"   ⚠️ Batch {batch_idx} splitting into 2 sub-batches of {expected_pages//2} pages...")
                mid = expected_pages // 2
                part1 = batch[:mid]
                part2 = batch[mid:]
                with tempfile.TemporaryDirectory() as tmp1, tempfile.TemporaryDirectory() as tmp2:
                    process_range_recursive(part1, tmp1, f"Batch {batch_idx} (Part 1)")
                    process_range_recursive(part2, tmp2, f"Batch {batch_idx} (Part 2)")
                    merge_subsets(tmp1, tmp2, batch_output_dir, len(part1))
                print(f"   ✨ Batch {batch_idx} split-processing complete.")
            
            # Progress reporting
            batch_elapsed = time.time() - batch_start
            elapsed_total = time.time() - file_start_time
            
            # Calculate progress bar
            pct = (global_processed_pages / global_total_pages) * 100
            bar_len = 15
            filled_len = int(bar_len * global_processed_pages // global_total_pages)
            bar = '█' * filled_len + '░' * (bar_len - filled_len)
            
            # Calculate ETA
            if global_processed_pages > 0:
                avg_time_per_page = elapsed_total / global_processed_pages
                remaining_pages = global_total_pages - global_processed_pages
                eta = avg_time_per_page * remaining_pages
                eta_str = format_time(eta)
            else:
                eta_str = "Calculating..."
                
            print(f"   [{bar}] {pct:.1f}% | {global_processed_pages}/{global_total_pages} pages | Batch {batch_idx}/{num_batches} | ETA: {eta_str}")
        
        file_elapsed = time.time() - file_start_time
        print(f"\n   🎉 Finished {pdf_file} in {format_time(file_elapsed)}\n")
        
    except Exception as e:
        print(f"\n   ❌ Error processing {pdf_file}: {e}\n")

total_elapsed = time.time() - total_start
print(f"{'='*60}")
print(f"🏁 All OCR tasks completed in {format_time(total_elapsed)}!")
print(f"📂 Output saved to: {OUTPUT_DIR}")
print(f"{'='*60}")
```
