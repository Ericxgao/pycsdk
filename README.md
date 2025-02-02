# pyCSDK

A Python/Cython wrapper for the Omnipage CSDK. It offers a simple python API to get OCR results.

## Install

First, set `LD_LIBRARY_PATH` to point to the CSDK shared libraries.

For example:

```bash
$ export LD_LIBRARY_PATH=/usr/local/lib/nuance-omnipage-csdk-lib64-20.3:$LD_LIBRARY_PATH
```

Next, clone this repo, enter the directory and

```bash
$ python3 setup.py install
```

If the install fails, you may have to edit the paths pointing to the SDK include files and shared libraries. They can be found in `setup.py`.

## Usage

```python
import os
from pycsdk import *
from PIL import Image
    
csdk = CSDK('my_company', 'my_product', license_file='/path/to/license_file', code='license_code')
csdk.set_setting('Kernel.OcrMgr.TradeOff', TO_ACCURATE)
csdk.set_setting('Kernel.OcrMgr.PDF.TradeOff', TO_ACCURATE)
csdk.set_setting('Kernel.OcrMgr.PDF.ProcessingMode', PDF_PM_AUTO)
csdk.set_setting('Kernel.OcrMgr.DefaultRecognitionModule', RM_OMNIFONT_PLUS3W)
csdk.set_language(LANG_FRE)
csdk.add_language(LANG_ENG)

# note each file can be handled by a child process with e.g. os.fork()
# (the CSDK does not support multithreading)
with csdk.open_file('myfile.pdf') as f:
    for page_id in range(f.nb_pages):
        with f.open_page(page_id) as p:
            p.process(despeckle_method=DESPECKLE_MEDIAN, remove_rule_lines=True) 
            print('found {} zones, {} letters'.format(len(p.zones), len(p.letters)))
            
            # do something interesting with zones (and cells in zones) and letters
            print(p.zones)
            print(p.letters)
            
            # save image for page
            p.image.save('myfile_{:04}.png'.format(page_id + 1), 'PNG')
```
