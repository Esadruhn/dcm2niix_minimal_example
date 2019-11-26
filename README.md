# dcm2niix_minimal_example
Minimal examples for dcm2niix - conversion

Compile the code as shown in [the issue](https://github.com/rordenlab/dcm2niix/issues/353)
```
g++ -I. main_console.cpp nii_foreign.cpp nii_dicom.cpp jpg_0XC3.cpp ujpeg.cpp nifti1_io_core.cpp nii_ortho.cpp nii_dicom_batch.cpp -o dcm2niix -DmyDisableOpenJPEG -DmyReportSliceFilenames
```

Get the [LIDC-IDRI dataset](https://www.cancerimagingarchive.net/nbia-search/?CollectionCriteria=LIDC-IDRI), folder LIDC-IDRI-0026/01-01-2000-02665/3000519-40086/

[viewer](https://www.cancerimagingarchive.net/viewer/?study=1.3.6.1.4.1.14519.5.2.1.6279.6001.279133908903915300821612602665&series=1.3.6.1.4.1.14519.5.2.1.6279.6001.101228986346984399347858840086&token=10adf844-45e5-447d-82f2-bad906f9bcda)

Take only those slices: 
* 000001.dcm
* 000173.dcm
* 000218.dcm
* 000241.dcm

Convert them with dcm2niix (`correct` is the name of the directory containing the slices).

```bash
{path_to_dcm2niix}/dcm2niix correct 
```

The output is:

```
Compression will be faster with 'pigz' installed http://macappstore.org/pigz/
Chris Rorden's dcm2niiX version v1.0.20191111  Clang11.0.0 (64-bit MacOS)
Found 4 DICOM file(s)
|4|correct/000241.dcm
|3|correct/000173.dcm
|2|correct/000241.dcm
|1|correct/000218.dcm
Warning: Unable to append protocol name (0018,1030) to filename (it is empty).
Convert 4 DICOM as correct/correct__20000101000000_3000519 (512x512x4x1)
Conversion required 0.024347 seconds (0.020107 for core code).
```

Now, open a notebook or Python script and use this code:

```python
import pydicom
import os
from glob import glob
from pprint import pprint

dir_path = "{your_path}"
order_slices = list()
for path in glob(dir_path+"*.dcm"):
    dicom_slice = pydicom.dcmread(path)
    order_slices.append((dicom_slice.ImagePositionPatient[-1], path.split("/")[-1]))

order_slices.sort()
pprint(order_slices)
```
The output is:
```
[("-345.000000", '000173.dcm'),
 ("-343.750000", '000241.dcm'),
 ("-342.500000", '000218.dcm'),
 ("-341.250000", '000001.dcm')]
```

And if you compare each slice to its corresponding position in the matrix: 
TODO