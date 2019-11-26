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

Now, copy this code in a Python script or a Jupyter notebook and run it:

```python
import pydicom
import os
from glob import glob
import numpy as np
import nibabel as nib

dir_path = "{your_path}"
ordered_slices = list()
for path in glob(dir_path+"*.dcm"):
    dicom_slice = pydicom.dcmread(path)
    ordered_slices.append((dicom_slice, path.split("/")[-1]))

# Sort the slices by ImagePositionPatient
ordered_slices.sort(key=lambda x: x[0].ImagePositionPatient[-1])

# Load the scan converted by dcm2niix
dcm2niix_scan = nib.load(dir_path+"correct__20000101000000_3000519.nii")

def get_slice_from_dcm(dcm_slice):
    """
    Get a slice of the scan from the dcm (apply rescaling and rotation operations)
    """
    # Scale the slice
    slope = np.float32(dcm_slice.get("RescaleSlope", 1))
    intercept = np.float32(dcm_slice.get("RescaleIntercept", 0))
    scaled_slice = dcm_slice.pixel_array.astype(np.float32) * slope + intercept
    
    # Rotate the slice 90°
    rotated = np.rot90(scaled_slice, k=1, axes=[1, 0])
    return rotated

def compare_slices():
    """
    Check that the order of the slices in the converted scan and ordered_slices is the same.
    Raises an exception if this is not True.
    """
    for i in range(len(ordered_slices)):
        dcm_slice = get_slice_from_dcm(ordered_slices[i][0])
        dcm2niix_slice = dcm2niix_scan.get_data()[:,:,i]

        assert np.allclose(dcm_slice, dcm2niix_slice)
        
    print(f"The order of the slices in the converted scan is: {[x[1] for x in ordered_slices]}")

compare_slices()
```

The output is:
```
The order of the slices in the converted scan is: ['000173.dcm', '000241.dcm', '000218.dcm', '000001.dcm']
```

## Conclusion

From my tests on the whole scan (257 slices), it seems that the problem is with the first and last slice, the others are OK.
It adds a slice that is already in the scan in first position (`000241.dcm`) and does not print the last slice (`000001.dcm`).