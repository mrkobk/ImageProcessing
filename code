# @title
!pip install Pillow --upgrade

from PIL import Image, ImageChops, ImageOps, ImageFile
from os import walk
from tqdm import tqdm

ImageFile.LOAD_TRUNCATED_IMAGES = True

import concurrent.futures
import time

CONNECTIONS = 50
TIMEOUT = 5
# loop over Image Input Folder and retrieve filenames
lst_img = next(walk("input/"), (None, None, []))[2]
lst_img = [i for i in lst_img if i != ".DS_Store"]

def load_url(url, timeout):
    
    # open image and save filename to variable
    im = Image.open(f"input/{url}")
    if hasattr(im, 'filename'):
        filename = im.filename
        filename = filename.split("/")[-1]
        filename = filename.split(".")[0]
    
    # get background(bg)color from first top left pixel and chop borders to lay bare prod image    
    bg = Image.new(im.mode, im.size, im.getpixel((0,0)))
    diff = ImageChops.difference(im, bg)
    diff = ImageChops.add(diff, diff, 2.0, -100)
    bbox = diff.getbbox()
    
    if bbox:
        im = im.crop(bbox)
    
    # resize product image to 900x900 by maintaining ratio
    im_resized = ImageOps.contain(im, (900,900))
    if im_resized.mode == "CMYK":
        im_resized = im_resized.convert("RGB")
    # save product images as tmp foreground image
    im_resized.save(f"input/tmp/tmp_{filename}.png", dpi=(1000,1000))
    fg = Image.open(f"input/tmp/tmp_{filename}.png")
    
    # if background is transparent transform to white
    if fg.mode == "RGBA":
        transp_replace = Image.new("RGBA", fg.size, "WHITE") 
        transp_replace.paste(fg, (0, 0), fg)
        transp_replace.convert('RGB').save(f'input/tmp/tmp_{filename}.jpg', "JPEG")
        fg = Image.open(f"input/tmp/tmp_{filename}.jpg")
        
    else:
        fg = Image.open(f"input/tmp/tmp_{filename}.png")
    
    # create 1000x1000 background image    
    img_w, img_h = fg.size
    background = Image.new('RGBA', (1000, 1000), (255, 255, 255, 255))

	# paste 900x900 foreground image (fg) onto 1000x1000x white background image to make all images of same size and scale   
    bg_w, bg_h = background.size
    offset = ((bg_w - img_w) // 2, (bg_h - img_h) // 2)
    background.paste(fg, offset)
    background = background.convert('RGB')
    
    return background.save(f'output/{filename}_cropped.jpg', optimize=True)

# run above function asynchronously with 50 images per loop            
with concurrent.futures.ThreadPoolExecutor(max_workers=CONNECTIONS) as executor:
    future_to_url = (executor.submit(load_url, url, TIMEOUT) for url in lst_img)
    time1 = time.time()
    for future in tqdm(concurrent.futures.as_completed(future_to_url)):

        try:

            data = future.result()
            
        except Exception as exc:

            print(exc)

    time2 = time.time()



