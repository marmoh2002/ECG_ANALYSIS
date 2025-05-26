# ECG Arrhythmia Detection Project

## Description

This project undertakes the critical task of analyzing electrocardiogram (ECG) signals for the detection of cardiac arrhythmias, leveraging the well-established MIT-BIH Arrhythmia Database. Raw ECG signals obtained from clinical or ambulatory settings are often contaminated with various forms of noise and artifacts, such as baseline wander caused by patient movement or respiration, power line interference from electrical equipment, and high-frequency noise from muscle tremors or instrumentation. These disturbances can significantly obscure the underlying physiological information, hindering accurate diagnosis and analysis. This project implements a systematic preprocessing pipeline designed to mitigate these issues. By loading ECG records, applying targeted filtering techniques to remove noise and artifacts, and visualizing the results, the primary objective is to produce a clean, high-quality ECG signal. This cleaned data serves as a reliable foundation for subsequent, more complex analyses, such as the accurate detection of R-peaks, calculation of heart rate variability (HRV), and ultimately, the classification of different types of arrhythmias.

## Dataset

The foundation of this analysis is the [MIT-BIH Arrhythmia Database](https://physionet.org/content/mitdb/1.0.0/), specifically version 1.0.0, a widely recognized benchmark resource in the field of cardiac electrophysiology research. Curated by the Beth Israel Hospital (BIH) Arrhythmia Laboratory between 1975 and 1979, this database comprises 48 half-hour excerpts of two-channel ambulatory ECG recordings. These recordings were captured from 47 different subjects, providing a diverse set of physiological conditions and arrhythmia examples. Each record includes the raw ECG signal data (`.dat` file), header information detailing sampling frequency, signal units, and channel names (`.hea` file), and expert-annotated beat labels (`.atr` file). For convenient access, the dataset is also available on Kaggle under the identifier [mariaamm/mit-bih-arrhythmia-database-1-0-0](https://www.kaggle.com/datasets/mariaamm/mit-bih-arrhythmia-database-1-0-0). Accessing the data via Kaggle requires setting up API credentials.

## Installation

To execute the code associated with this project, a Python environment is required, along with several essential scientific computing and data visualization libraries. These dependencies can be readily installed using the Python package manager, `pip`. Execute the following command in your terminal:

```bash
pip install wfdb matplotlib pandas numpy scipy scikit-learn kaggle
```

This command installs:
*   `wfdb`: A library specifically designed for reading, writing, and processing ECG signals and annotations stored in the PhysioBank format (used by the MIT-BIH database).
*   `matplotlib`: A fundamental library for creating static, animated, and interactive visualizations in Python, used here for plotting ECG waveforms.
*   `pandas`: A powerful data manipulation and analysis library, useful for handling metadata or results.
*   `numpy`: The core library for numerical computing in Python, providing support for arrays and mathematical functions.
*   `scipy`: A library building upon NumPy, offering modules for optimization, signal processing (including filtering), and statistics.
*   `scikit-learn`: A comprehensive machine learning library (though used minimally here, potentially for future classification steps).
*   `kaggle`: The official Kaggle API client, enabling programmatic download of datasets.

Furthermore, if you intend to download the dataset directly using the Kaggle API within the script or notebook, you must configure your Kaggle API credentials. This typically involves placing a `kaggle.json` file (containing your username and API key) in the appropriate location (e.g., `~/.kaggle/` or as specified by the `KAGGLE_CONFIG_DIR` environment variable). For security, ensure this file has restricted permissions:

```bash
chmod 600 /path/to/your/kaggle.json
```

## Usage

Follow these steps to run the ECG analysis pipeline:

1.  **Obtain the Dataset:** First, ensure the dataset archive, `mit-bih-arrhythmia-database-1.0.0.zip`, is available locally. You can download this manually from the PhysioNet or Kaggle links provided above, or you can utilize the Kaggle API integration within the provided script/notebook if your credentials are correctly configured. Place the downloaded zip file in the project's working directory or another accessible location.
2.  **Extract the Dataset:** Unzip the `mit-bih-arrhythmia-database-1.0.0.zip` archive. This will create a directory (e.g., `mit-bih-arrhythmia-database-1.0.0/`) containing the individual record files (`.dat`, `.hea`, `.atr` for each record like '100', '101', etc.).
3.  **Execute the Analysis:** Run the main Python script or Jupyter Notebook file associated with this project. The code is designed to process a predefined list of records (e.g., '100', '101', '102' as demonstrated in the source). It will load the specified records, apply the sequence of preprocessing filters, and generate comparative plots illustrating the effects of each filtering stage.

## Methodology

The core of this project lies in its signal processing pipeline, designed to systematically enhance the quality of the raw ECG data. This involves several distinct stages:

1.  **Data Loading and Initial Exploration:** The process begins by loading the specified ECG records from the extracted dataset directory. The `wfdb` library is employed to read the signal data (`.dat`), header information (`.hea`), and annotation files (`.atr`). An initial exploration phase involves visualizing short segments (e.g., 10 seconds) of the raw ECG signal from a chosen channel (typically 'MLII' or 'V5'). Crucially, the corresponding expert annotations (indicating heartbeat types like 'N' for normal, 'V' for ventricular ectopic, etc.) are overlaid onto these plots. This initial visualization provides valuable context, revealing the baseline signal quality, the types of noise present, and the distribution of different heartbeat types within the segment.

2.  **Signal Preprocessing:** This is the most critical phase, addressing the common contaminants in ECG signals through a sequence of filtering operations:
    *   **Baseline Wander Removal:** Low-frequency oscillations, often below 0.5 Hz, constitute baseline wander. This artifact, caused by factors like patient breathing or electrode movement, shifts the entire signal vertically, making it difficult to analyze ST segments or measure true wave amplitudes. In this project, baseline wander is tackled using a high-pass filtering approach implemented via the Fast Fourier Transform (FFT). The signal is transformed into the frequency domain, components below the 0.5 Hz cutoff are set to zero, and the signal is transformed back to the time domain, effectively removing the slow drift while preserving higher-frequency cardiac components.
    *   **Power Line Interference Removal:** Electrical grids introduce a pervasive sinusoidal interference at their operating frequency (typically 50 Hz or 60 Hz). This noise manifests as a high-frequency ripple superimposed on the ECG signal. A notch filter is specifically designed to eliminate this narrow frequency band. The `scipy.signal.iirnotch` function is used to design an Infinite Impulse Response (IIR) notch filter centered precisely at the power line frequency (50 Hz in the provided code). Applying this filter using `scipy.signal.filtfilt` (which performs zero-phase filtering to avoid phase distortion) effectively removes the power line hum without significantly impacting adjacent frequencies containing valuable ECG information.
    *   **Noise Reduction and Smoothing (Bandpass Filtering):** Beyond baseline wander and powerline noise, ECG signals can contain other high-frequency noise from muscle activity (EMG artifacts) or electronic instrumentation. To address this and generally smooth the signal, a bandpass filter is applied. The chosen passband is typically 0.5 Hz to 40 Hz, encompassing the frequencies where most clinically relevant ECG features (P waves, QRS complexes, T waves) reside. Frequencies below 0.5 Hz (already attenuated by the baseline wander removal) and above 40 Hz are attenuated. This is achieved using a Butterworth filter (designed with `scipy.signal.butter`) due to its flat passband characteristics, again applied using `filtfilt` to maintain phase integrity. This step significantly improves the signal-to-noise ratio and enhances the visual clarity of the ECG waveforms.

## Results & Visualization

A key output of this project is the visualization of the signal transformation process. For each processed record, the script generates a series of plots that allow for direct comparison of the signal at different stages. Typically, a plot will display the original raw ECG segment alongside the signals resulting from baseline wander removal, notch filtering, and the final bandpass filtering. These comparative visualizations clearly demonstrate the impact of each preprocessing step: the stabilization of the baseline, the elimination of the 50 Hz hum, and the overall smoothing effect of the bandpass filter. The final filtered signal exhibits significantly enhanced clarity, making the P, QRS, and T waves distinctly identifiable against a cleaner background.

## Commentary

The sequential application of baseline wander removal, notch filtering, and bandpass filtering demonstrably improves the quality and interpretability of the raw ECG signals from the MIT-BIH database. Removing the low-frequency drift ensures that amplitude and segment measurements are accurate relative to a stable isoelectric line. Eliminating the pervasive power line interference removes a significant source of visual clutter and potential misinterpretation. Finally, attenuating high-frequency noise smooths the signal, making the characteristic waveforms of the cardiac cycle stand out more clearly. The resulting preprocessed signal, largely free from common artifacts while preserving essential physiological information within the 0.5-40 Hz band, is significantly more suitable for reliable automated analysis. Tasks such as accurate R-peak detection (crucial for heart rate calculation), heart rate variability (HRV) analysis, and the development of machine learning models for arrhythmia classification benefit immensely from this enhanced signal quality, leading to more robust and clinically relevant findings.

