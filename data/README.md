# CWRU dataset

- [CWRU data overview](https://engineering.case.edu/bearingdatacenter/download-data-file)
- [12k drive-end fault recordings](https://engineering.case.edu/bearingdatacenter/12k-drive-end-bearing-fault-data)
- [Normal baseline recordings](https://engineering.case.edu/bearingdatacenter/normal-baseline-data)

The raw dataset is deliberately **outside this project** and is not in the submission ZIP. MATLAB filenames must retain their numeric recording IDs. Subdirectories are supported, but duplicate filenames are rejected.

| Paper class | Condition | 1 HP file | 2 HP file | 3 HP file |
| --- | --- | --- | --- | --- |
| 1 | Healthy | 98.mat | 99.mat | 100.mat |
| 2 | Ball 0.007 inch | 119.mat | 120.mat | 121.mat |
| 3 | Ball 0.014 inch | 186.mat | 187.mat | 188.mat |
| 4 | Ball 0.021 inch | 223.mat | 224.mat | 225.mat |
| 5 | Inner race 0.007 inch | 106.mat | 107.mat | 108.mat |
| 6 | Inner race 0.014 inch | 170.mat | 171.mat | 172.mat |
| 7 | Inner race 0.021 inch | 210.mat | 211.mat | 212.mat |
| 8 | Outer race 0.007 inch, centered @6 | 131.mat | 132.mat | 133.mat |
| 9 | Outer race 0.014 inch, centered @6 | 198.mat | 199.mat | 200.mat |
| 10 | Outer race 0.021 inch, centered @6 | 235.mat | 236.mat | 237.mat |

**Manual setup:** open the official pages, select the named class/load links above, save all 30 numeric `.mat` files in a local directory, and set `DATASET_PATH` in the notebook configuration cell. The fault page labels links with names such as `B007_1`, but downloaded files use the numeric IDs above. Select the **12k drive-end** dataset, not 48k fault data or fan-end fault experiments. Both DE and FE sensor channels are read from each selected file.

Optional automatic download (tested successfully against the official source):

Set `DOWNLOAD_DATA=True` in the first notebook cell and run the dataset section. The download function is included directly in the notebook.

If the university site is unavailable, use the manual procedure; the code does not substitute synthetic data or unverified mirrors. To choose a different directory without editing code:

```python
DATASET_PATH = r"D:/datasets/CWRU_raw"
```

Normal baseline files are treated as 48 kHz and downsampled on both channels to 12 kHz. The baseline source page does not specify each channel's rate; this assumption is documented. The loader selects keys matching the recording ID, which correctly handles the extra X098 data embedded in 99.mat. The source manifest records exact keys, rates, lengths and SHA-256 hashes.


`download_manifest.csv` records the official download URLs, class/load mapping and SHA-256 hashes. Keep raw recordings outside the repository (the default is `../CWRU_raw`). Preprocessing and task sampling code are included directly in the notebook.
