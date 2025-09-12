# cnnxin25
We are using MACOS with Sequoia (V15.5) with a M4 chip. First, need to download FSL, since Quantiphyse is a toolbox made for FSL. On MACOS, we use terminal for the command line.

## 1. Downloading FSL 
 <pre>curl -Ls https://fsl.fmrib.ox.ac.uk/fsldownloads/fslconda/releases/getfsl.sh | sh -s</pre> 
After installation is finished, you’ll see a message “FSL successfully installed”. After that, need to open a new terminal

> To access FSL guidelines, type "fsl &".
> To access FSLEYES viewer, type "fsleyes -std &"

## 2. Downloading Quantiphyse
Installing Quantiphyse needs a conda environment, so need to install anaconda. But you might have to bypass the new M4 chip and use the older Intel chip which has the installations for Quantiphyse already configured

<pre>CONDA_SUBDIR=osx-64 conda create -n qp python=3.7 --no-default-packages -c conda-forge
conda activate qp
pip install -U sphinx
pip install quantiphyse 
pip install quantiphyse-asl
pip install pyobjc 
export QT_MAC_WANTS_LAYER=1
"Quantiphyse"</pre>

Quantiphyse will now begin running. 
> NOTE: to check the version for Quantiphyse, Sphinx or any installations, type:
> sphinx-build --version

## 3. Preprocess Structural Images (T1w) using FSL
For my test run, I will be analyzing OSIPI DRO1, 3, 5, 7, and 9

Preprocessing: fsl_anat -i {directory}/${subid}_T1w.nii.gz -o {directory}/${subid}_T1w.anat --clobber --nosubcortseg --nocro
<pre>
BASE_DIR=~/Documents/Challenge_Data/connexin25test

for SUBID in DRO1 DRO3 DRO5 DRO7 DRO9; do
    INPUT="${BASE_DIR}/sub-${SUBID}/sub-${SUBID}_T1w.nii.gz"
    OUTPUT="${BASE_DIR}/sub-${SUBID}/sub-${SUBID}_T1w.anat"

    echo "Running fsl_anat for subject: $SUBID"
    fsl_anat -i "$INPUT" -o "$OUTPUT" --clobber --nosubcortseg --nocrop
done </pre> 

OR 

<pre>for SUBID in DRO2 DRO4 DRO6 DRO8; 
do fsl_anat -i ~/Documents/Challenge_Data/connexin25test/sub-${SUBID}/sub-${SUBID}_T1w.nii.gz -o ~/Documents/Challenge_Data/connexin25test/sub-${SUBID}/sub-${SUBID}_T1w.anat --clobber --nosubcortseg --nocrop; 
done
</pre>

## 4. Load ASL and MO images into Quantiphyse 
1. Open Quantiphyse
3. Go to File → Load Data
4. Load: <pre>ASL image: {directory}/${subid}_asl.nii.gz → sub-DRO1_asl.nii.gz 
   MO image: {directory}/${subid}_mOscan.nii.gz → sub-DRO1_mOscan.nii.gz</pre>

> NOTE: if Quantiphyse doesn't open, or if says "Command not found", run step 2 again and install under Intel chip. 
   
## 5.  Fix NaN Voxels 
Why? NaN voxels can appear when something goes wrong during scanning or preprocessing — like dividing by zero or applying a faulty mask. These NaN values aren't real numbers, so can't be processed properly. If they’re not fixed, they can cause crashes or give empty or incorrect results. To avoid this, we replace NaNs with zeros before running the analysis.

  1. Go to Widgets → Processing → Simple Maths
  2. Set the command to: np.nan_to_num(${subid}_asl) → np.nan_to_num(subDRO1_asl)</pre>
  3. Choose an output name for subject file with no NaNs: ${subid}_asl_nonan. Example → subDRO1_asl_nonan
  4. Uncheck "Output is an ROI" (unless you want ROI)
  5. Click Run

  Now you’ll use ${subid}_asl_nonan for ASL analysis.

## 6.  Run ASL Data Processing

1. Go to Widgets → ASL → ASL Data Processing
2. Remember to load data for No NaN asl.nii files
3. Usually, the software will autodetect the parameters, but double check with the parameter details from the asl.json file

**ASL Data Tab** 
<li>ASL data → subDRO1_asl_nonan.nii</li>
<li>Control-label pairs → Select if applicable, the order is important! For this data, it's CONTROL-LABEL (find this information in the DRO asl_context file) </li>
<li> Labelling → cASL/pcASL </li>
<li> Readout → 3D (GRE)</li>
<li> PLDs → 0.25 </li>
<li> Bolus duration → 1.8 s </li><br></br>

**Corrections Tab**
<li> Select motion correction (M0 file) </li>
<br></br>

**Structural Data Tab**
<li>Load structural output from FSL: Folder → {directory}/${subid}_T1w.anat </li>
<li>Don't override automatic segmentation!</li>
<br></br>
**Calibration Tab**
<li>Calibration method: Voxelwise </li>
<li>Calibration image: m0scan.nii </li>
<li>Sequence TR (s): 10 </li>
<li>Sequence TE (ms): 0 </li>
<li>Calibration gain: 1 </li>
<li>Inversion efficiency: 0.85 </li>
<li>(Use defaults or custom values from Alsop et al., 2015) </li><br></br>

**Analysis Tab**
<li>Select "Spatial regularization", "Fix label duration", "Fix arterial transit time"</li>
<li>Check Partial Volume Correction if needed. (You must run separately for PVC and non-PVC analyses)</li><br></br>

**Output Tab**
<li>Choose output space: native, structural, or MNI (I selected native ASL space)</li>
<li>Select Output mask, Output calibration data, Output structural segmentation</li>
<li>Select Save HTML report, because this contains the numerical values for the ASL data</li>
<br></br>

> NOTE: After selecting setting the options in the "ASL data processing" widget, remember to check the "Save copy of output data" option and create a folder before clicking "Run", otherwise Quantiphyse would not save the output within a folder by default. You can manually save each output data by selecting the data in the "Volume" widget (first widget) and select "File" menu → "Save current data".





